---
title: "Grid 布局算法！自己动手实现一个 Grid"
date: 2018-05-20 15:11:54 +0800
tags: wpf uwp algorithm
coverImage: /static/posts/2018-05-20-15-50-53.png
permalink: /post/grid-layout-algorithm.html
---

[Avalonia](https://github.com/AvaloniaUI/Avalonia) 是一款尚在开发中的基于 .NET Core 的跨平台 UI 框架。目前用在个人项目中还是不错的，不过还需要大家在开源社区中多多支持。

我为它写了一个全新的 `Grid` 布局算法，此算法是 WPF 在通常情况下的性能的两倍。本文将分享我在此项目中实现的算法的原理。

---

<div id="toc"></div>

## Grid 的布局行为到底是怎样的？

Grid 算是 WPF/UWP 入门中非常重要的一个布局容器了。面对它那强大而熟悉的布局方式，大家应该没有什么疑问吧！

比如：
- 可以定义行和列
- 可以分别为每一行和列指定宽高
- 宽高的值可选 `Auto`, `*` 和数值
    - `Auto` 表示 `Grid` 将按照元素的实际所需尺寸进行布局
    - `*` 表示行列在布局中的比例，`*` 前面的数值表示比例值
    - 数值使用的是 WPF/UWP 布局单位
- 元素在 Grid 中可跨行或跨列

基本上大家所熟知的 `Grid` 布局差不多就这样么多了。

*如果想了解 WPF/UWP 的布局单位，可以阅读我之前的一篇文字[将 UWP 的有效像素（Effective Pixels）引入 WPF - 吕毅](/post/introduce-uwp-effective-pixels-into-wpf)*。

然而，事实上 `Grid` 的布局行为才没有那么简单呢！它诡异的地方在于没有定义好多种复杂布局情况下的交叉行为。我写了中英两篇文章来说明了这些不太符合预期的行为：

- [WPF/UWP 的 Grid 布局竟然有 Bug，还不止一个！了解 Grid 中那些未定义的布局规则 - 吕毅](/post/the-bugs-of-grid)
- [The undefined behaviors of WPF Grid (the so-called bugs) - 吕毅](/post/the-bugs-of-grid-en)

作为一个非常有潜力的 .NET Core 跨平台 UI 框架 Avalonia，应该认真定义好这些行为，而不是像 WPF/UWP 现有的 `Grid` 那样在某些情况下比较含糊，出现难以解释的布局行为。

## 为这样的 Grid 布局行为设计一套算法

如果你熟知 WPF/UWP 的布局系统，那么 `MeasureOverride` 和 `ArrangeOverride` 一定不陌生，虽然它们只是布局的一部分（为什么是一部分？详见 [Visual->UIElement->FrameworkElement，带来更多功能的同时也带来了更多的限制 - 吕毅](/post/features-and-limits-on-visual-uielement-frameworkelement)）。

不过，写一个 `Grid` 确实只需要关心这两个函数就够了。`MeasureOverride` 传入父级测量的可用尺寸，返回此 `Grid` 测量发现所需的最小尺寸；`ArrangeOverride` 传入父级实际可提供的可用尺寸，返回此 `Grid` 实际布局所用的尺寸。

### 分析 Grid 的布局思路

如果行或列设置为 `Auto`，那么 `Grid` 的行或者列将为这个元素的尺寸进行适配，并且元素的所需尺寸也会影响到 `Grid` 的最小所需尺寸；如果行或列设置为 `*`，那么 `Grid` 的行列不会为此元素适配，但是元素的所需尺寸依然会影响到 `Grid` 的最小所需尺寸。

由于我们必须要计算 `Grid` 的最小所需尺寸，所以整个布局过程中，必须得到每个行列的最小所需尺寸。这意味着，即便我们不能确定此行或此列的尺寸，或者甚至在父级尺寸确定的情况下能够确定此行或此列时，也应该计算最小尺寸。而 `Auto`、元素的 `DesiredSize`、`*` 或者行列的最小值都会影响到此最小尺寸，所以这些都应该先考虑。而行或列的最大值应该在最后再考虑。

于是，我们将整个布局过程分成以下几步：

1. 测量行列范围中包含 `Auto` 或 `*` 的元素（前者影响行列和最小尺寸，后者仅影响最小尺寸）
1. 将所有的已确定尺寸确定
1. 将所有的有最小尺寸，且 `*` 展开后超过此最小尺寸的行列按最小值确定
1. 将所有 `Auto` 行列确定
1. 按照父级尺寸估算 `*` 的尺寸
1. 计算 `Grid` 所需的最小尺寸
1. 将估算缩得的尺寸作为实际尺寸进行测量

### 布局算法设计

`Grid` 的布局算法似乎难以用语言描述，不过，我可以尝试用更具体的文字用接近代码的方式来描述：

* 测量过程
    1. 寻找所有行列范围中包含 Auto 和 * 的元素，使用全部可用尺寸提前测量
    1. 排除所有固定尺寸的行列，然后从总长中将其减掉
    1. 进行循环（以排除全部 min 要求的，*总长为负也要继续*）
        - 计算单位星长（单位星长 = 剩余总长 / 星数，最小为 0）
        - 找出第一个不满足 min 要求的 *，置其长度为 min，排除此行列，然后从总长中将其减掉
        - 所有的 * 检查完毕后，退出循环
    1. 进行循环（以排除全部 Auto，*总长为负也要继续*）
        - 计算单位星长（单位星长 = 剩余总长 / 星数，最小为 0）
        - 找到第一个 Auto，找到对此 Auto 的约束（跨行列或不跨行列都需要）
        - 对每个约束，检查目前尺寸是否满足约束（跨行列尺寸 >= Max(DesiredSize, min.Sum()）
        - 满足约束的忽略，不满足的约束需要计算约束大出行列的尺寸值，将此值设定为此 Auto 的待选长度
        - 当所有的约束检查完毕，在所有的待选长度中取最大值，设定为 Auto 的尺寸，排除此行列，然后从总长中将其减掉
        - 所有的 Auto 检查完毕后，退出循环
    1. 按照父级尺寸估算 `*` 的尺寸
        - 如果还有剩余长度，则将 * 展开，但需要考虑 `*` 行列的最小尺寸
    1. 确定 Grid 的 DesiredSize
        - 如果剩余总长 >= 0，则 Grid.DesiredSize = 可用长度 - 剩余总长
        - 如果剩余总长 < 0，则返回的 Grid.DesiredSize = 可用长度，实际需求的 Grid.DesiredSize = 可用长度 - 剩余总长
    1. 如果总长 >= 0，则进行循环（以确定剩余全部子元素的测量所用尺寸）；否则直接将剩余 * 全部设置为 0
        + 进行循环
            - 计算单位星长（单位星长 = 剩余总长 / 星数）
            - 找出第一个不满足 max 要求的 *，置其长度为 max，排除此行列，然后从总长中将其减掉
            - 所有的 * 检查完毕后，退出循环
        + 至此，剩余的所有 * 都将不再有约束（即便元素 DesiredSize 不满足也无需处理，直接将元素裁剪）
            - 计算单位星长（单位星长 = 剩余总长 / 星数）
            - 计算所有剩余 * 的长度
        - 至此，所有子元素测量所用尺寸已经全部确定，使用此尺寸对子元素进行测量
    1. 分开保存 1~5、6 计算后的所得结果，布局过程即结束
* 排列过程
    1. 如果排列可用尺寸大于测量可用尺寸，则测量过程中的全部计算部分需要重新进行（因为可能此前的 min 现在不满足条件）
    1. 如果排列可用尺寸等于测量可用尺寸，则直接使用测量第 6 步的结果，依次进行排列，不足部分的空间全部使用 0
    1. 如果排列可用尺寸小于测量可用尺寸，则重新执行测量第 6 步的计算，以确定新的行列尺寸，依次进行排列，不足部分的空间全部使用 0

我在 Avalonia 的代码注释中，画出了每一个步骤的变化图。

```csharp
// 1. 测量行列范围中包含 `Auto` 或 `*` 的元素（前者影响行列和最小尺寸，后者仅影响最小尺寸）
//
// 2. 将所有的已确定尺寸确定
// +-----------------------------------------------------------+
// |  *  |  A  |  *  |  P  |  A  |  *  |  P  |     *     |  *  |
// +-----------------------------------------------------------+
//                   |#fix#|           |#fix#|
//
// 3. 将所有的有最小尺寸，且 `*` 展开后超过此最小尺寸的行列按最小值确定
// +-----------------------------------------------------------+
// |  *  |  A  |  *  |  P  |  A  |  *  |  P  |     *     |  *  |
// +-----------------------------------------------------------+
// | min | max |     |           | min |     |  min max  | max |
//                   | fix |     |#fix#| fix |
//
// 4. 将所有 `Auto` 行列确定
// +-----------------------------------------------------------+
// |  *  |  A  |  *  |  P  |  A  |  *  |  P  |     *     |  *  |
// +-----------------------------------------------------------+
// | min | max |     |           | min |     |  min max  | max |
//       |#fix#|     | fix |#fix#| fix | fix |
//
// 5. 按照父级尺寸估算 `*` 的尺寸
// +-----------------------------------------------------------+
// |  *  |  A  |  *  |  P  |  A  |  *  |  P  |     *     |  *  |
// +-----------------------------------------------------------+
// | min | max |     |           | min |     |  min max  | max |
// |#des#| fix |#des#| fix | fix | fix | fix |   #des#   |#des#|
//
// 6. 计算 `Grid` 所需的最小尺寸
//
// 7. 将估算缩得的尺寸作为实际尺寸进行测量
// +-----------------------------------------------------------+
// |  *  |  A  |  *  |  P  |  A  |  *  |  P  |     *     |  *  |
// +-----------------------------------------------------------+
// | min | max |     |           | min |     |  min max  | max |
// |#fix#| fix |#fix#| fix | fix | fix | fix |   #fix#   |#fix#|
```

### 布局算法的代码

为了让代码更容易调试，我专门写了一个 `GridLayout` 类来完成布局过程，而且 `GridLayout` 的计算设计为与 `Grid` 布局过程无关。做法是，将 `GridLayout` 的大部分方法设计为“纯方法”（纯方法只随便调用，调用此方法不会改变任何系统状态，只有拿到其返回值才会真正发挥作用）。

具体的代码非常长，含单元测试供 1200+ 行，建议去 Avalonia 仓库查看：

- [Avalonia/Grid.cs at master · AvaloniaUI/Avalonia](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Grid.cs)
- [Avalonia/GridLayout.cs at master · AvaloniaUI/Avalonia](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Utils/GridLayout.cs)

## 效果和性能

在性能测试中，此算法还是表现不错的，以下是 Pull Request 中的性能测试截图（已经合并）。

![](/static/posts/2018-05-20-15-50-53.png)


