# Effect 语法

如果希望在引擎中实现自定义的着色效果, 需要书写自定义 Effect.<br>
一个 Effect 文件大体可以分为两部分的内容, 一份 YAML 格式的流程控制清单, 和相关的 (类)GLSL 语法的 shader 片段.<br>
这两部分内容上相互补充, 共同构成了一个完整的渲染流程描述.

## 语法框架
以 skybox.effect 为例, 这个 Effect 文件的内容大致是这样:

![effect](effect.png)

## 关于 YAML
YAML 是一门面向数据序列化的，对人类书写十分友好的语言，但它引入了一些独特的语法来表示不同类型的数据，<br>
对于不熟悉这门语言的开发者可能会有一点门槛，我们在 [这里](yaml-101.md) 快速总结了一下最常用的一些语法和语言特性，有需要可以参考。

## Pass 中可配置的参数
vert 和 frag 声明了当前 pass 使用的 shader, 格式为 `片段名:入口函数名`<br>
这个名字可以是本文件中声明的 shader 片段名, 也可以是引擎提供的标准头文件.
片段中不应出现 main 函数入口, 预处理阶段会将指定入口函数的返回值赋值给当前 shader 的输出 (gl_Position 或最终的输出颜色).<br>
其他可配置 GL 参数及默认值见 [完整列表](pass-parameter-list.md).

## Shader 片段
Shader 片段在语法上是标准 GLSL 300 es 语法的一个超集, 在资源加载时有相应的预处理流程.

这一节会介绍所有的扩展语法, 更多实际使用示例, 可参考编辑器内提供的 builtin effect.

在标准 GLSL 语法上, 我们引入了一些非常自然的 C 风格语法扩展:

### include 机制
类似 c 的头文件 include 机制, 可供引用的 builtin 头文件都在 chunks 目录下, 主要包括一些常用的工具函数, 和标准光照模型等. 另外所有在当前 effect 文件中声明的 shader 片段都可相互引用.

### 预处理宏定义
Effect 系统的设计倾向于在游戏项目运行时可以方便地利用 shader 中的各类预处理宏, 而减少 runtime branching.<br>
编辑器会在加载资源时收集所有在 shader 中出现的 defines, 然后引擎在运行时动态地将需要的声明加入 shader 内容.<br>
所以要使用这些预处理宏, 只需要如上面的例子中一样, 在 shader 中直接进行逻辑判断即可.<br>
所有的 define 都会被序列化到 inspector 上, 供用户调整.<br>
注意相关写法的一些限制:
1. 如果在 shader 中声明了 extension, 这个 extension 必须有且只有一个 define 来控制启用与否.
2. 运行时会显式定义所有 shader 中出现的宏，所以请不要使用 #ifdef 或 #if defined() 这样的形式做判断.

### macro tags
虽然我们会自动识别所有出现在预处理分支逻辑中 (#ifdef) 的宏定义，但有时同样利用宏定义可以方便地做一些其他事情，如：
```
float metallic = texture(pbrMap, uv).METALLIC_SOURCE;
```
针对这类没有在预处理分支逻辑中出现的宏定义，需要选择一个合适的 tag 显式声明：<br>

| Tag     | Effect |
|:-------:|:------:|
| range   | 针对连续数字类型的宏定义，显式指定它的取值范围，范围应当控制到最小，有利于运行时的 shader 管理 |
| options | 针对有清晰选项的宏定义，显示指定它的可用选项 |
| default | 针对运行时可能取任意值的宏定义，显式指定它的默认值 |

```glsl
#pragma define NUM_SPOT_LIGHTS range([0, 4])
#pragma define METALLIC_SOURCE options([r, g, b, a])
#pragma define JOINTS default(30)
```
每个 tag 只有一个参数，支持完整 YAML 语法.

### functional macros
由于 WebGL1 不原生支持，我们将函数式预处理宏提供为资源导入期的功能：
```c
#define DECL_CURVE_STRUCT(name) \
  uniform int u_##name##_curveMode;
#define DECL_CURVE_STRUCT_INT(name) \
  DECL_CURVE_STRUCT(name) \
  uniform float u_##name##_minIntegral[MAX_KEY_NUM - 1];

DECL_CURVE_STRUCT_INT(velocity_pos_x)
```
对这样的声明，最后一行会在资源导入期就被展开变成:
```glsl
  uniform int u_velocity_pos_x_curveMode;
  uniform float u_velocity_pos_x_minIntegral[MAX_KEY_NUM - 1];
```

### WebGL 1 fallback 支持
由于 WebGL 1 仅支持 GLSL 100 标准语法, 在预处理阶段会提供 300 es 转 100 的 fallback shader, 用户基本不需关心这层变化.<br>
但注意目前的 fallback 只支持一些基本的格式转换，如果使用了 300 es 独有的 shader 函数（texelFetch、textureGrad 等）或 extension，我们推荐根据 \_\_VERSION__ 宏定义判断 shader 版本，自行实现更稳定的 fallback。

### 关于 Uniform Block
我们规定在 shader 中所有非 sampler 的 uniform 都应以 block 形式声明, 但因 WebGL2 只支持 std140 的布局，<br>
当 block member 不是 16 字节对齐时（长度小于 vec4）GL 会自动做 padding 补齐，这与渲染器底层的 assumption 相背，<br>
因此我们会在资源导入期做 UBO 的布局检测，对会引起自动 padding 的 UBO 声明生成警告。
