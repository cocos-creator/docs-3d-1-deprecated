# Label 组件参考

Label 组件用来显示一段文字，文字可以是系统字体，TrueType 字体或者 BMFont 字体。另外，Label 还具有排版功能。

![label-property](./label/label-property.png)

点击 **属性检查器** 下面的 **添加组件** 按钮，然后从 **UI** 中选择 **Label**，即可添加 Label 组件到节点上。

文字的脚本接口请参考 [Label API](https://docs.cocos.com/creator/2.1/api/zh/classes/Label.html)。

## Label 属性

| 属性 |   功能说明
| -------------- | ----------- |
| Color | 图片颜色
| SharedMaterial | 用户自定义材质，没有设置则用内置材质
| Priority | 渲染排序优先级。渲染排序采用广度排序方式，即在同一级子节点下，带有渲染组件的节点，优先级高的后渲染。
| String | 文本内容字符串。
| Horizontal Align | 文本的水平对齐方式。可选值有 LEFT，CENTER 和 RIGHT。
| Vertical Align | 文本的垂直对齐方式。可选值有 TOP，CENTER 和 BOTTOM。
| Font Size | 文本字体大小。
| Line Height | 文本的行高。
| Overflow | 文本的排版方式，目前支持 CLAMP，SHRINK 和 RESIZE_HEIGHT。详情见下方的 [Label 排版](#label-%E6%8E%92%E7%89%88)。
| Enable Wrap Text | 是否开启文本换行。（在排版方式设为 CLAMP、SHRINK 时生效）
| SpacingX | 文本字符之间的间距。（使用 BMFont 位图字体时生效）
| Font | 指定文本渲染需要的字体文件，如果使用系统字体，则此属性可以为空。
| Font Family | 文字字体名字。在使用系统字体时生效。
| Cache Mode | 文本缓存类型（v2.0.9 中新增），仅对 **系统字体** 或 **ttf** 字体有效，BMFont 字体无需进行这个优化。包括 **NONE**、**BITMAP**、**CHAR** 三种模式。详情见下方的 [文本缓存类型](#%E6%96%87%E6%9C%AC%E7%BC%93%E5%AD%98%E7%B1%BB%E5%9E%8B%EF%BC%88cache-mode%EF%BC%89)。
| Use System Font | 布尔值，是否使用系统字体。
| Src Blend Factor | 当前渲染混合模式
| Dst Blend Factor | 背景混合模式，和上面的属性共同作用，可以将前景和背景渲染的文本用不同的方式混合，效果预览可以参考 [glBlendFunc Tool](http://www.andersriggelsen.dk/glblendfunc.php)

## Label 排版

| 属性 |   功能说明
| -------------- | ----------- |
|CLAMP| 文字尺寸不会根据 Bounding Box 的大小进行缩放，Wrap Text 关闭的情况下，按照正常文字排列，超出 Bounding Box 的部分将不会显示。Wrap Text 开启的情况下，会试图将本行超出范围的文字换行到下一行。如果纵向空间也不够时，也会隐藏无法完整显示的文字。
|SHRINK| 文字尺寸会根据 Bounding Box 大小进行自动缩放（不会自动放大，最大显示 Font Size 规定的尺寸），Wrap Text 开启时，当宽度不足时会优先将文字换到下一行，如果换行后还无法完整显示，则会将文字进行自动适配 Bounding Box 的大小。如果 Wrap Text 关闭时，则直接按照当前文字进行排版，如果超出边界则会进行自动缩放。
|RESIZE_HEIGHT| 文本的 Bounding Box 会根据文字排版进行适配，这个状态下用户无法手动修改文本的高度，文本的高度由内部算法自动计算出来。

## 详细说明

Label 组件可以通过往 **属性检查器** 里的 **Font** 属性拖拽 TTF 字体文件和 BMFont 字体文件来修改渲染的字体类型。如果不想继续使用字体文件，可以通过勾选 **Use System Font** 来重新启用系统字体。

使用艺术数字字体需要创建 [艺术数字资源](../asset-workflow/label-atlas.md)，参考链接中的文档设置好艺术数字资源的属性之后，就可以像使用 BMFont 资源一样来使用艺术数字了。

### BMFont 与 UI 合图自动批处理

 从 Creator 1.4 版本开始， BMFont 支持与 UI 一起合图进行批量渲染。
 理论上，如果你的游戏 UI 没有使用系统字体或者 TTF 字体，并且所有的 UI 图片资源都可以合在一张图上，那么 UI 是可以只用一个 Draw Call 来完成的。
 更多关于 BMFont 与 UI 合图自动批处理的内容，请参考 [BMFont 与 UI 合图自动批处理](https://docs.cocos.com/creator/2.1/manual/zh/advanced-topics/ui-auto-batch.html)
