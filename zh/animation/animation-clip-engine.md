
# 动画片段

动画片段是一组动画曲线，包含了所有动画数据。

## 动画曲线

动画曲线描述了某一对象上某一属性值随着时间的变化。
在内部，动画曲线存储了一系列时间点，每个时间点都对应着一个（曲线）值，并称为一帧，或关键帧。
当动画系统运作时，动画组件根据当前动画状态计算出指定时间点应有的（结果）值并赋值给对象，完成属性变化；这一计算过程称为采样。

### 目标对象

动画曲线的目标可以是任意 Cocos3D 组件或结点。
运行时，由动画组件根据路径动态确定目标对象；因此，动画片段可以复用到多个对象上。

### 采样

若采样时间点恰好就等于某一关键帧的时间点，则使用该关键帧上的动画数据。
否则——当采样时间点居于两帧之间时，结果值应同时受两帧数据的影响，
采样时间点在两处关键帧的时刻区间上的比例（`[0,1]`）反应了影响的程度。
Cocos3D 允许将该比例映射为另一个比例，以实现不同的“渐变”效果。
这些映射方式，在 Cocos3D中称为**渐变方式**。
在比例确定之后，根据指定的**插值方式**计算出最终的结果值。
渐变方式和插值方式都影响着动画的平滑度。

#### 渐变方式

可以为每一帧指定渐变方式，也可以为所有帧指定统一的渐变方式。
渐变方式可以是内置渐变方式的名称或贝塞尔控制点。

以下列出了几种常用的渐变方式。
- `linear` 保持原有比例，即线性渐变；当未指定渐变方式时默认使用这种方式；
- `constant` 始终使用比例 0，即不进行渐变；与插值方式 `Step` 类似；
- `quadIn` 渐变由慢到快。
- `quadOut` 渐变由快到慢。
- `quadInOut` 渐变由慢到快再到慢。
- `quadOutIn` 渐变由快到慢再到快。
- `IBezierControlPoints`

<script src="./easing-method-example.js"> </script>
<button onclick="onEasingMethodExampleButtonClicked()">展开对比</button>
<div id="easing-method-example-panel"> </div>

#### 曲线值与插值方式

有些插值算法需要每一帧的曲线值中存储额外的数据，因此，
曲线值与目标属性的值类型不一定相同。
对于数值类型或值类型，Cocos3D 提供了几种通用的插值方式；
同时，也可以定义自己的插值方式。

当曲线数据的 `interpolate` 属性为 `true` 时，曲线将尝试使用插值函数：
- 若曲线值的类型为 `number`、`Number`，将应用线性插值；
- 否则，若曲线值继承自 `ValueType`，将调用 `ValueType` 的 `lerp` 函数完成插值，
Cocos3D 内置的大多数值类型都将其 `lerp` 实现为线性插值；
- 否则，若曲线值是[可插值的]()，将调用曲线值的 `lerp` 函数完成插值<sup id="a1">[1](#f1)</sup>。

若曲线值不满足上述任何条件，或当曲线数据的 `interpolate` 属性为 `false`时，
将不会进行插值操作 --- 永远使用前一帧的曲线值作为结果。

渐变方式和插值方式都影响着动画的平滑度。

<b id="f1">1</b> 对于数值、四元数以及各种向量，Cocos 提供了相应的可插值类以实现[三次样条插值](https://en.wikipedia.org/wiki/Spline_interpolation)。 [↩](#a1