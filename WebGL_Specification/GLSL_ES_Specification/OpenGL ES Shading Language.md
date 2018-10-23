# OpenGL ES Shading Language

1. 简介

2. OpenGL ES Shading概述

3. 基础概念

4. 变量和类型

5. 操作符和表达式

6. 语句和结构体

7. 内置变量

8. 内置函数

9. Shading语法

10. 错误

11. 输入和输出计数

12. 问题

## 简介

本文档描述OpenGL ES Shading Language 3.00版本.

由本语言编写的独立编译单元称为着色器.程序是由一整套着色器编译和链接在一起组成的.

### 兼容性

OpenGL ES 3.0 API被设计成可以和GLSL ES v1.00和GLSL ES 3.00一起工作,通常情况下为OpenGL ES 2.0编写的着色器不需要修改应该也可以在OpenGL ES 3.0上工作.

当从OpenGL ES 2.0移植应用程序到OpenGL ES 3.0的时候,应该注意以下几点:

- 不是所有在v1.00中出现的语言结构都可以在v3.00中使用.例如attribute和varying限定符.但是,GLSL ES 3.00的功能是GLSL ES 1.00的超集.
- 有些OpenGL ES 3.0 API要求的的语言特性只有在GLSL ES 3.00中存在,而GLSL ES 1.00中不存在.
- OpenGL ES 2.0 API不支持GLSL ES 3.00编写的着色器.
- 在OpenGL ES 3.0中使用GLSL ES 1.00着色器可以使用扩展的资源.这样的着色器不需要能够在OpenGL ES 2.0实现中运行.

## OpenGL ES Shading概述

OpenGL ES处理管线中包含两个可编程的处理器,分别是顶点处理器和片元处理器.

### 顶点处理器

顶点处理器是一个用于操作输入的顶点以及关联数据的可编程单元,运行在这个处理器上的OpenGL ES Shading语言编写的编译单元称为顶点着色器.

顶点着色器一次操作一个 顶点.它不能替代那些一次需要多个形状点的图形操作.

### 片元处理器

顶点处理器是一个用于操作片元值以及关联数据的可编程单元,运行在这个处理器上的OpenGL ES Shading语言编写的编译单元称为片元着色器.

片元着色器不能改变片元的位置.也不允许访问邻近的片元值.片元着色器计算出来的值最终用来更新帧缓冲内存或者文理内存,取决于当前OpenGLES的状态和OpenGL ES命令.

### 可执行体

一个顶点着色器和一个片元着色器被编译和链接到一起形成一个可执行体.OpenGL ES 3.0在每个阶段不支持多个编译单元.

## 基本概念

### 字符集

OpenGL ES Shading 语言使用的字符集是UTF-8编码架构中的Unicode.无效的UTF-8字符会被忽略.

- 值为零的字节总是被解释为字符串的结束
- 反斜杠用于表示行连续
- 空格可以由以下字符组成: 空格,水平tab,垂直tab,换页,回车,换行
- \#符号被用于预处理指令
- 宏名字限制:
  - a-z,A-Z和下划线_
  - 数组0-9,但不能是宏名字第一个字符

行结一般是以回车或换行结束,也可以同时使用,是一样的效果.

字符集是大小写敏感的.

没有字符或者字符串类型,所以没有引号字符.

没有文件结束字符.

### 版本声明

着色器必须声明使用的语言版本.版本在着色器的第一行被指定:

```glsl
#version number es
```

其中number必须是合法的语言版本,必须跟\_\_VERSION\_\_宏保持一致.指令"#version 300 es"要求着色器使用3.00版本语言.任何编译器不支持的版本号将会导致错误产生.1.00版本的语言不需要着色器包含这个指令,因此如果着色器不包含#version指令将会被认为是1.00版本.

声明为3.00版本编写的着色器不能和1.00版本的着色器链接到一起,因此顶点着色器和片元着色器的版本声明应该保持一致.

\#version指令必须出现在着色器的第一行并且后面必须跟一个换行符.

version-declearation:

​	whitespace(opt) POUND whitespace(opt) VERSION whitespace number whitespace ES whitespace(opt)

Tokens:

- PUNND      #
- VERSION    version
- ES                es

### 预处理器

完整的预处理指令如下:

```glsl
#
#define
#undef

#if
#ifdef
#ifndef
#else
#elif
#endif

#error
#pragma

#extension

#line
```

另外操作符defined也可用.

注意版本指定不认为是预处理指令所以不再列表里面.

如果#在一行中单独出现则被忽略.列表中不存在的指令会导致错误.

\#define和\#undef是C++标准预处理器,用于带参数和不带参数的宏定义.

下面的预定义宏可用

```glsl
__LINE__
__file__
__VERSION__
GL_ES
```

\_\_LINE\_\_将被替换成一个比前面换行数加1的十进制整数.

\_\_FILE\_\_将被替换成一个代表当前被处理的源代码字符串数量的十进制整数.

\_\_VERSION\_\_将被替换成一个代表OpenGL ES Shading语言版本号的十进制整数.其值是300.

GL_ES将被定义成1.在非OpenGL ES Shading语言环境中不是true,所以可以用在做编译时的测试,决定shader是都是运行在ES系统上.

有这样的约定,所有包含连续两个下划线_的宏名字都保留给底层软件使用.在着色器中定义这样名字的宏本身并不会导致错误,但是可能会导致无意识的行为,因为出现了相同名字的多个定义.所有以"GL_\_"前缀开头的宏名字也是保留的,定义这样的宏名字将会导致编译时错误.

undfine或redefine内置(预定义)宏名字是错误的.

宏名字的最大长度是1024.声明超出这个长度的名字是错误的.

\#if,\#ifdef,\#ifndef,\#else,#elif和\#endif有如下规定:

- 在\#if和\#elif之后的表达式必须是pp-constant-expression.
- 未定义的标识符默认值不是'0',因此不能被defined操作符使用,如果使用这样的标识符将会导致错误.
- 字符常量不支持

pp-constant-expression是一个在编译时的预处理阶段计算的表达式,它由字符,整数,常量和下面的操作符组成:

| 优先级     | 操作类型 | 操作符               | 结合律   |
| ---------- | -------- | -------------------- | -------- |
| 1(最高级)  | 括号分组 | ()                   | NA       |
| 2          | 一元     | defined<br />＋－~ ! | 从右到左 |
| 3          | 乘法     | * / %                | 从左到右 |
| 4          | 加法     | ＋－                 | 从左到右 |
| 5          | 位移     | <<  >>               | 从左到右 |
| 6          | 关系     | <  >  <=  >=         | 从左到右 |
| 7          | 相等     | ===  !=              | 从左到右 |
| 8          | 位与     | &                    | 从左到右 |
| 9          | 位异或   | ^                    | 从左到右 |
| 10         | 位或     | \|                   | 从左到右 |
| 11         | 逻辑与   | &&                   | 从左到右 |
| 12(最低级) | 逻辑或   | \|\|                 | 从左到右 |

defined操作可以被这样使用:

```glsl
defined identifier
defined (identifier)
```

前面不需要\#或@或\#\#操作符.

与处理器的语义与c++中预处理器语义一样,以下例外:

- 当且仅当第一个操作数是非0时才计算&&的第二个操作数值
- 当且仅当第一个操作数是0时才计算||的第二个操作数值
- 没有布尔类型和布尔字面量.true和false分别用整数1和0表达.

如果操作数没有被计算,则操作数中的未定义标识符不会导致错误.

\#error会导致实现把诊断信息放到着色器对象的信息日志里.这个信息将会是从\#error指令到第一个换行符之间的内容.实现必须把\#error指令当做编译时错误来对待.

如果实现不能识别\#pragma后面的符号,它将忽略这个pragma.下面的pragmas作为语言的一部分被定义.

```glsl
#pragma STDGL
```

STDGL是保留的pragma,供本语言现在和将来的版本使用.没有实现可以使用名为STDGL的pragma.

```glsl
#pragma optimize(on)
#pragma optimize(off)
```

optimize可以用来在开发和调试着色器的时候关闭优化.只能在函数定义外使用,所有着色器的优化是默认打开的.

```glsl
#pragma debug(on)
#pragam debug(off)
```

debug可以用来对带有调试信息的着色器启用编译和注释,所以可以被调试器使用.只能在函数定义外使用.默认情况下debug是关闭的.

优化和调试的范围和效果是依赖于实现的,但是使用它们都不会产生错误.

默认情况下,编译器必须报出着色器中所有不符合规范的句法,语法,语义错误.\#extension指令可以控制编译器扩展方面的行为.

```glsl
#extension extension_name : behavior
#extension all : behavior
```

extension_name是扩展的名字,扩展名字这里没有列出来.all意味着行为会被应用到编译器支持的所有扩展上面.behavior可以是以下之一:

- require
- enable
- warn
- disable

扩展指令不允许组合,必须单独设置.指令的顺序没有关系,后面出现的指令覆盖前面的.

编译器的初始状态等于

```glsl
#extension all : diable
```

这告诉编译器报告所有的错误和警告都必须报告,但是忽略掉所有扩展的.

每个扩展都有一个关联的宏.在实现中已经定义好所支持的扩展,可以如下使用:

```glsl
#ifdef OES_extension_name
	#extension OES_extension_name : enable
	// 使用扩展
#else
	// 替代方案
#endif
```

### 注释

有两种注释形式/*……/*和//.注释内部不能嵌套注释.

### 符号

语言由一系列符号组成.符号包括

- 关键字
- 标识符
- 整数常量
- 浮点常量
- 操作符
- ; { }

### 关键字

以下关键字不能被使用

const uniform
layout
centroid flat smooth
break continue do for while switch case default
if else
in out inout
float int void bool true false
invariant
discard return
mat2 mat3 mat4
mat2x2 mat2x3 mat2x4
mat3x2 mat3x3 mat3x4
mat4x2 mat4x3 mat4x4
vec2 vec3 vec4 ivec2 ivec3 ivec4 bvec2 bvec3 bvec4
uint uvec2 uvec3 uvec4
lowp mediump highp precision
sampler2D sampler3D samplerCube
sampler2DShadow samplerCubeShadow
sampler2DArray
sampler2DArrayShadow
isampler2D isampler3D isamplerCube
isampler2DArray
usampler2D usampler3D usamplerCube
usampler2DArray
struct

以下关键字为预留的,也不能使用

attribute varying
coherent volatile restrict readonly writeonly
resource atomic_uint
noperspective
patch sample
subroutine
common partition active
asm
class union enum typedef template this
goto
inline noinline volatile public static extern external interface
long short double half fixed unsigned superp
input output
hvec2 hvec3 hvec4 dvec2 dvec3 dvec4 fvec2 fvec3 fvec4
sampler3DRect
filter
image1D image2D image3D imageCube
iimage1D iimage2D iimage3D iimageCube
uimage1D uimage2D uimage3D uimageCube
image1DArray image2DArray
iimage1DArray iimage2DArray uimage1DArray uimage2DArray
imageBuffer iimageBuffer uimageBuffer
sampler1D sampler1DShadow sampler1DArray sampler1DArrayShadow
isampler1D isampler1DArray usampler1D usampler1DArray
sampler2DRect sampler2DRectShadow isampler2DRect usampler2DRect
samplerBuffer isamplerBuffer usamplerBuffer
sampler2DMS isampler2DMS usampler2DMS
sampler2DMSArray isampler2DMSArray usampler2DMSArray
sizeof cast
namespace using

另外加上所有包含两个连续下换线\_\_开头的.

### 标识符

标识符被用作变量名,函数名,结构名和字段选择器(字段选择器从向量,矩阵,结构体中选择组件).标识符形式

```glsl
identifier
	nonfigit
	identifier nondigit
	identifier digit
```

nondigit:

​	\_ a b c d e f g h i j k l m n o p q r s t u v w x y z
	A B C D E F G H I J K L M N O P Q R S T U V W X Y Z

digit:

​	0 1 2 3 4 5 6 7 8 9

以"gl\_"开头的标识符被OpenGL ES使用,如果重新声明会报错.

## 变量和类型

变量和函数先声明后使用,变量和函数名字都是标识符.

所有变量和函数声明都必须指定类型和可选的限定符.一次可以声明多个变量,中间用逗号分隔.变量也可以在声明的时候同时使用赋值操作符(=)进行初始化.

### 基本类型

基本数据类型可以分为以下几类:

简单类型

- void 
  没有返回值的函数使用

- bool

  true或false

- int 
  有符号整数

- uint 
  无符号整数

- float 
  单个浮点标量

- vec2 
  两个浮点标量组成的向量

- vec3 
  3个浮点标量组成的向量

- vec4 
  4个浮点标量组成的向量

- bvec2
  两个布尔值组成的向量 

- bvec3
  三个布尔值组成的向量 

- bvec4 
  四个布尔值组成的向量

- ivec2 
  两个有符号整数组成的向量

- ivec3 
  三个有符号整数组成的向量

- ivec4 
  四个有符号整数组成的向量

- uvec2 
  两个无符号整数组成的向量

- uvec3 
  三个无符号整数组成的向量

- uvec4 
  四个无符号整数组成的向量

- mat2 
  2×2浮点数矩阵

- mat3 
  3×3浮点数矩阵

- mat4 
  4×4浮点数矩阵

- mat2x2 
  和mat2相同

- mat2x3 
  3行2列的浮点矩阵

- mat2x4 
  4行3列的浮点矩阵

- mat3x2 
  2行3列的浮点矩阵

- mat3x3 
  和mat3相同

- mat3x4 
  4行3列的浮点矩阵

- mat4x2 
  2行4列的浮点矩阵

- mat4x3 
  3行4列的浮点矩阵

- mat4x4 
  和mat4相同

浮点采样类型

- sampler2D 
  访问2D纹理的句柄
- sampler3D 
  访问3D纹理的句柄
- samplerCube 
  访问立方体纹理映射的句柄
- samplerCubeShadow 
  a handle for accessing a cube map depth texture with comparison
- sampler2DShadow 
  a handle for accessing a 2D depth texture with comparison
- sampler2DArray 
  访问2D纹理数组的句柄
- sampler2DArrayShadow 
  a handle for accessing a 2D array depth texture with comparison

有符号整数采样类型

- isampler2D 
  访问整数2D纹理的句柄
- isampler3D 
  访问整数3D纹理的句柄
- isamplerCube 
  访问整数立方体纹理映射的句柄
- isampler2DArray 
  访问整数2D纹理数组的句柄

无法好整数采样类型

- usampler2D 
  访问无符号整数2D纹理的句柄
- usampler3D 
  访问无符号整数3D纹理的句柄
- usamplerCube 
  访问无符号整数立方体纹理映射的句柄
- usampler2DArray 
  访问无符号整数2D纹理数组的句柄

还可以通过这些类型通过数组和结构体构建出更复杂的类型.

没有指针类型.

#### Void

没有返回值的函数必须声明为void.

#### 布尔

支持bool类型,有两个字面值常量true和false.可以如下声明和初始化

```glsl
bool success; // 声明success为布尔类型
bool done = false; // 声明和初始化done
```

赋值操作符右边的表达式必须是bool类型

if,for,?:,while,do-while中的条件表达式必须是bool类型.

#### 整数

有符号和无符号整数都支持.高精度无符号整数有32位精度.高精度有符号整数有32位精度,其中包括符号位,采用二进制补码形式表示.中精度和低精度整数由实现来定义精度位数.

整数声明和初始化如下:

```glsl
int i, j = 42; // 整数字面量默认类型是int
unint k = 3u; // u后缀表示类型是uint
```

字面整数常量可以表达为十进制,八进制或者16进制,如下:

**整数常量:**

- 十进制常量       后缀(可选)
- 八进制常量       后缀(可选)
- 十六进制常量   后缀(可选)

**后缀:** u U其中之一

**十进制常量:**0 1 2 3 4 5 6 7 8 9

**八进制常量:**0 1 2 3 4 5 6 7

**十六进制常量:**0x或0X 0 1 2 3 4 5 6 7 8 9 a b c d e f A B C D E F

例子:

- 1 // 正确. 有符号整数, 值为1
- 1u // 正确. 无符号整数, 值为1
- -1 // 正确. 值为-1.
- -1u // 正确. 值为0xffffffff
- 0xA0000000 // 正确. 32位16进制有符号整数
- 0xABcdEF00u // 正确. 32位16进制无符号整数
- 0xffffffff // 正确. 有符号整数, 值为-1
- 0x80000000 // 正确. 等于-2147483648
- 0xffffffffu // 正确. 无符号整数, 值为0xffffffff
- 0xffffffff0 // 错误: 超出32位
- 0xfffffffff // 错误: 超出32位
- 3000000000 // 正确. 有符号十进制整数.等于-1294967296
- 2147483648 // 正确. 等于-2147483648
- 5000000000 // 错误: 超出32位
- int x = 1u; // 错误: 类型不匹配.
- uint y = 1; // 错误: 类型不匹配.

**溢出:**n位二进制能表达的有符号整数范围是[-2^(n-1),2^(n-1)-1],当数值范围超出这个范围就会发生溢出,当正溢出时,可以用公式N-2^(n)计算溢出后的值,计算出的值大于0,则数值超出n位二进制能表达的范围.当负溢出时,用公式N+2^(n)计算溢出后的值,计算出的值小于0,则数值超出n位二进制能表达的范围.

**补码:**正整数的补码是其二进制表示.负整数的补码由其绝对值的二进制表示所有位取反后加1得到.

下表展示符号位+原码,反码,补码之间的不同

| 十进制数 | 符号位+ 二进制绝对值 的表示方式        | 反码                                         | 补码                                                         |
| -------- | :------------------------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| +7       | **0111**                               | 表示方式不变                                 | 表示方式不变                                                 |
| +6       | **0110**                               | 表示方式不变                                 | 表示方式不变                                                 |
| +5       | **0101**                               | 表示方式不变                                 | 表示方式不变                                                 |
| +4       | **0100**                               | 表示方式不变                                 | 表示方式不变                                                 |
| +3       | **0011**                               | 表示方式不变                                 | 表示方式不变                                                 |
| +2       | **0010**                               | 表示方式不变                                 | 表示方式不变                                                 |
| +1       | **0001**                               | 表示方式不变                                 | 表示方式不变                                                 |
| +0       | **0000**                               | 表示方式不变                                 | 表示方式不变                                                 |
| -0       | 1000                                   | *1111*                                       | **0000**(与+0相同)                                           |
| -1       | 1001                                   | 1110                                         | 1111                                                         |
| -2       | 1010                                   | 1101                                         | 1110                                                         |
| -3       | 1011                                   | 1100                                         | 1101                                                         |
| -4       | 1100                                   | 1011                                         | 1100                                                         |
| -5       | 1101                                   | 1010                                         | 1011                                                         |
| -6       | 1110                                   | 1001                                         | 1010                                                         |
| -7       | 1111                                   | 1000                                         | 1001                                                         |
| -8       | 超出4个bit所能表达范围                 | 超出4个表达范围                              | 1000                                                         |
| 注：     | 要设计硬件区分符号位，比较绝对值大小。 | 无需设计硬件比较大小，但零存在两种表示方法。 | 较好的解决上述问题。由于零只有一种表达方式，所以，可以比别的方式多表达一个-8. |

####浮点数

浮点数遵守IEEE754单精度浮点数的定义和动态范围.可以像这样定义:

```glsl
float a, b = 1.5;
```

**浮点常量**定义如下:

- 小数常量 指数部分(可选) 后缀
- 数字 指数部分 后缀

**小数常量:**数字.数字

**指数部分:**

- e 符号(可选) 数字
- E 符号(可选) 数字

**符号:**+ -

**后缀:**f F

浮点数值超出单精度范围最大值和最小值时将会被转换成正无穷和负无穷.

#### 向量

向量有2,3,4维向量,其中的元素可以是浮点数,整数,布尔值.浮点数向量通常被用来存储颜色,法向量,位置,纹理坐标,纹理查找结果等.向量声明的例子:

```glsl
vec2 texcoord1, texcoord2;
vec3 position;
vec4 myRGBA;
ivec2 textureLookup;
bvec3 less;
```

#### 矩阵

内置有2×2,2×3,2×4,3×2,3×3,3×4,4×2,4×3和4×4的浮点数矩阵,第一个数组代表列数,第二个数代表行数.声明矩阵例子:

```glsl
mat2 mat2D;
mat3 optMatrix;
mat4 view, projection;
mat4x4 view; // 跟mat4一样
mat3x2 m; // 3列两行的矩阵
```

**mat2**是**mat2x2**的别名.**mat3**和**mat4**也是相似的,下面的定义是合法的:

```glsl
mat2 a;
mat2x2 b = a;
```

#### 不透明类型

不透明类型声明了对其他对象有效的不透明句柄的变量。 这些对象只能通过内置功能访问,不能直接通过声明的变量进行读写.他们可以作为函数的参数或者in uniform修饰的变量.不透明变量不允许出现在表达式中,否则会引起编译时错误.

不透明变量不能作为函数的out或inout参数使用,也不能被赋值.但是可以作为in参数被传递.只能通过OpenGL API初始化.

#### 采样器

采样器类型(例如**sampler2D**)是不透明类型.它们是纹理和过滤器的句柄与内置的纹理函数一起使用(在第8.7节“纹理查找函数”中描述)来指定要访问哪个纹理以及如何过滤纹理.采样器聚合成数组时只能用常量表达式来索引.

#### 结构体

用户定义的类型可以通过将其他已定义的类型聚合到一个结构中来创建,使用**struct**关键字。例如:

```glsl
struct light {
	float intensity;
	vec3 position;
} lightVar;
```

在这个例子中,*light*成为新类型的名字,*lightVar*称为*light*类型的变量.然后可以使用它的名字声明新类型的变量.

```glsl
light lightVar2;
```

结构体声明语法如下:

**结构体定义:**

​	限定符(可选) **struct** 名字(可选) {成员列表} 声明符(可选);

**成员列表:**

​	成员声明

**成员声明:**

​	基本类型声明符

这里的名字成为用户定义的类型,可以用来声明这种新类型的变量.该名称与其他变量、类型和函数共享相同的名称空间。 之前所有可见的具有该名称的变量、类型、构造函数或函数都会被隐藏。可选的限定符只适用声明符，并不是被定义类型名称的一部分。

结构体必须至少要有一个成员.成员成名可能包含精度限定符,也可能不包含.bit字段不支持.成员类型必须是已经定义的.成员声明不能包含初始化.成员成名可以包含数组.数组必须制定长度,长度必须是大于0的常量表达式.每个级别的结构体有自己的命名空间,只要保证在命名空间内名字唯一即可.

匿名结构体不支持,嵌入式结构体定义不支持.

```glsl
struct S { float f; } // 允许:S被定义为一个结构体.
struct T {
    S; // 错误:匿名结构体不允许
    struct { ... }; // 错误:嵌入式结构体定义不允许
    S s; // 语序:有名字的嵌套结构体
}
```

#### 数组

数组的长度必须是大于0的常量表达式.数组长度可以是有符号和无符号的整数.使用大于或等于数组长度的索引来访问数组是非法的.使用负的常量表达式作为索引也是非法的.只能声明一维数组.所有基本类型和结构体都可以组成数组.例如:

```glsl
float frequencies[3];
uniform vec4 lightPosition[4u];

const int numLights = 2;
light lights[numLights];
```

数组类型可以通过类型后面加方括号和长度形成:

```glsl
float[5]
```

这个类型可以在任何地方使用

```glsl
float[5] foo() {} // 作为函数返回值
float[5](3.4, 4.2, 5.0, 5.2, 1.1) // 作为构造函数
void foo(float[5]) // 作为未命名的参数
float[5] a // 声明变量或函数参数
```

数组类型也可以不指定长度,但定义的时候必须包含初始化器:

```glsl
float x[] = float[2](1.0, 2.0); // 声明长度为2的数组
float y[] = float[](1.0, 2.0, 3.0) // 声明长度为3的数组
float a[5];
float b[] = a;
```

注意,初始化器不必是一个常量表达式,但是它的长度必须是常量表达式.

错误的数组声明:

```glsl
float a[5][3]; // 非法:只允许一维数组
float[5] a[3]; // 非法
```

数组可以使用数组构造函数进行初始化:

```glsl
float a[5] = float[5](3.4, 4.2, 5.0, 5.2, 1.1);
float a[5] = float[](3.4, 4.2, 5.0, 5.2, 1.1); // 一样的效果
```

数组声明不指定长度是错误的.

数组有固定的元素数量,可以通过长度方法获取:

```glsl
a.length(); // 对于上面的声明会返回5
```

#### 术语定义

*整型*

​	*整型*是指任何有符号和无符号的整型标量或向量.它包含数组和结构体.

​	int uint

​	ivec2 ivec3 ivec4

​	uvec2 uvec3 uvec4

*标量整型*

​	*标量整型*标量有符号和无符号整型:

​	int uint

*向量整型*

​	*向量整型*是有符号或无符号整型组成的向量:

​	ivec2 ivec3 ivec4

​	uvec2 uvec3 uvec4

*浮点类型*

​	*浮点类型*指任何浮点标量,向量或矩阵.它包括数组和结构体,包括以下:

​	float

​	vec2 vec3 vec4

​	mat2 mat3 mat4

​	mat2x2 mat2x3 mat2x4

​	mat3x2 mat3x3 mat3x4

​	mat4x2 mat4x3 mat4x4

*布尔类型*

​	*布尔类型*指任何布尔标量或向量.保罗数组和结构体.

​	bool

​	bvec2 bvec3 bvec4

*不透明类型*

​	*不透明类型*是隐藏在语言中的内部结构类型.

​	sampler2D sampler3D samplerCube

​	sampler2DShadow samplerCubeShadow

​	sampler2DArray

​	sampler2DArrayShadow

​	isampler2D isampler3D isamplerCube

​	isampler2DArray

​	usampler2D usampler3D usamplerCube

​	usampler2DArray

### 作用域

声明的作用域决定声明在何处可见 .GLSL ES使用静态嵌套作用域系统.这允许名字被重定义.

#### 术语定义

声明的作用域是指在程序中声明可见的区域.

作用域可以嵌套.

命名空间定义了可以定义名称的位置.在单个命名空间中,一个名字最多只能有一个入口,可能是结构体,变量或函数.

一般情况下,每个作用域会关联一个命名空间.然后,在某些情况下不同,例如uniform,多个作用域共享同一个命名空间.在这种情况下,声明冲突是错误的,虽然名称只在声明名称的作用域中可见 .

#### 作用域类型

在所有函数外面声明的变量是全局作用域.在while或for语句中声明的变量作用域就是它的子语句.如果在符合语句中定义,那作用域就到复合语句结束.如果作为函数参数声明,作用域就到函数结束.函数参数和函数体是一样的作用域.

```glsl
int f( /* 嵌套作用域从这里开始 */ int k)
{
	int k = k + 3; // 名称k重声明错误
...
}

int f(int k)
{
    {
        int k = k + 3; // 第二个k是参数,用来初始化嵌套中的第一个k
        int m = k // 使用新的k,这个k会隐藏参数k
    }
}
```

对于for和while循环,子语句不会引进新的做用户,因此下面的代码会报错:

```glsl
for ( /* 嵌套作用域从这里开始 */ int i = 0; i < 10; i++)
{
	int i; // 重声明错误
}
```

do-while的循环体会引入新的作用域,但while却仍然使用外部作用域

```glsl
int i = 17;
do
	int i = 4;  // 正确,在嵌套作用域中
while (i == 0); // i的值是17,使用外部作用域中的i
```

if结构:**if** 表达式 **then** 语句1 **else** 语句2

语句1和语句2中声明的变量作用域是独立的.

#### 重声明名称

在同一作用域中函数可以被重复声明,参数一样不一样都可以.其他已经被声明过的名字则不能被再次声明.如果在嵌套作用域中重声明在外部已经使用过的名字,那将会隐藏掉外部定义.

内置函数不能被重新声明,因此不能重载和钉钉一内置函数.

声明只是在符号表中添加了一个名字,定义是一个名字的完整定义:

```glsl
int f(); // 声明;
int f() {return 0;} // 声明和定义
int x; // 声明和定义
int a[4]; // 声明和定义
struct S {int x;}; // 声明和定义
```

对于函数来说,只有整个签名一样才会被认为是同一个声明.对于变量,数组和结构体,只要名字一样就被认为是同一个声明.

下面是一些正确的例子:

```glsl
void f(int) {...}
void f(float) {...} // 允许函数重载

void f(int); // 第一次声明
void f(int); // 重声明
void f(int) {...} // 只定义了一次
```

下面是错误的例子:

```glsl
void f(int) {...}
void f(int) {...} // 错误: 重定义

void f(int);
struct f {int x;}; // 错误:f和函数f冲突

struct f {int x;};
int f; // 错误:和结构体类型f冲突

int a[3];
int a[3]; // 错误:数组重定义

int x;
int x; // 错误:变量重定义
```

#### 全局作用域

全局作用域中包含内置函数,用户定义的全局变量和用于定义的函数.函数定义不能出现在函数内部,因此函数的作用域必须是全局的.

#### 共享全局作用域

共享全局作用域是指其中的变量可以被多个编译单元访问.在GLSL ES中只有uniform是共享全局作用域.顶点着色器的输出不认为是共享全局作用域,因为它们必须被传递到片元着色器作为输入.

### 存储限定符

变量声明可以在类型前面指定一个存储限定符.

| 限定符     | 含义                                                        |
| ---------- | ----------------------------------------------------------- |
| 默认值none | 本地读写内存,或作为函数的输入参数                           |
| const      | 编译时常量,或只读的函数参数                                 |
| in         | 从前一个阶段输入                                            |
| out        | 输入到下一个阶段                                            |
| uniform    | 值在处理的时候不会被改变,用于链接着色器,OpenGL ES和应用程序 |

**out**和**in**可以被下面的插值限定符进一步修饰

| 限定符 | 意义                              |
| ------ | --------------------------------- |
| smooth | perspective correct interpolation |
| flat   | 不插值                            |

这些插值限定符只能放在**in**,**out**前面.他们无法应用在顶点着色器的输入和片元着色器的输出上.

本地变量只能使用**const**限定符.

函数参数可以使用**const**, **int**和**out**限定符.

函数返回值和结构体字段不能使用存储限定符.

#### 默认存储限定符

如果全局变量没有限定符,那么这个变量不会链接到应用程序或者在其他管道阶段运行的着色器.对于全局或本地没有限定符的变量来说,声明的时候会分配内存,这个变量将会提供访问这个内存的方式.

#### 常量限定符

命名的编译时常量可以使用**const**限定符来声明.任何被修饰为常量的变量都是只读的.**const**限定符可以和任何非void透明基本类型使用,包括结构体和数组.在声明意外修改**const**变量是错误的,因此必须在声明的时候初始化.

```glsl
const vec3 zAxis = vec3(0.0, 0.0, 1.0);
```

结构体字段不能被修饰为**const**.结构体变量可以被声明为**const**,并且使用结构体构造函数进行初始化.

常量声明的初始化器必须是常量表达式.

#### 常量表达式

常量表达式是以下的一种:

- 字面值(例如5或true)
- 被**const**修饰的全局和本地变量(不包括函数参数)
- 数组的length()方法
- 参数是常量表达式的构造函数
- 参数是常量的内置函数调用,纹理查找函数除外

用户自定义函数的函数调用不能作为常量表达式.

带const限定符的标量,向量,矩阵,数组和结构体都是常量表达式.采样器类型不能作为常量表达式.

常量整型表达式也是常量表达式,其值为有符号或无符号的标量整数.

#### 输入变量

着色器输入变量声明时会带上**in**存储限定符.它们形成了OpenGL管道前一个阶段和当前着色之间的输入接口.输入变量必须是全局作用域的.在着色器开始执行的时候值从前一个管道阶段拷贝到输入变量.用**in**关键字声明的变量不能在着色器执行期间被改写.只有实际上被用到的输入变量才会被前一个阶段写入;多余的输入变量声明时允许的.

顶点着色器输入变量是以下类型:

- 布尔类型
- 不透明类型
- 数组
- 结构体

顶点着色器输入声明例子:

```glsl
in vec4 position;
in vec3 normal;
```

图形硬件有少量的固定向量位置用于传递顶点输入.因此,OpenGL ES Shading 语言这样定义,每个非矩阵的输入变量都占用一个这样的向量位置.可用的位置数量依赖于实现,如果超出这个限制将会导致链接错误(声明的输入变量不是静态的,不会被计入这个限制).标量输入会被当做一个vec4进行计数,因此应用程序可以考虑将4个不相关的浮点输入组合到一个向量中以更好 的利用底层硬件的功能.矩阵输入会占用多个位置,使用的未知数等于矩阵的列数.

片元着色器输入会得到每个片元值,通常来自前一个阶段的输入会被插值.他们声明的时候会使用**in**限定符.

片元着色器的输入不能使用以下类型:

- 布尔类型
- 不透明类型
- 数组构成的数组
- 结构体构成的数组
- 包含数组的结构体

当片元着色器输入是有符号或无符号整数,整数向量时,必须使用插值限定符f**lat**.

片元着色器输入声明例子:

```glsl
in vec3 normal;
centroid in vec2 TexCoord;
flat in vec3 myColor;
```

顶点着色器的输出和片元着色器的输入形成一对接口.因此,顶点着色器的输出变量和片元着色器的输入变量,如果名字相同,则类型和限定符必须匹配(精度除外,**out**匹配**in**).

#### Uniform变量

**uniform**限定符被用来声明全局变量,它的值在整个处理过程中是不变的.所有**unifrom**变量都是只读的.他们在链接的时候被初始化为0,可用通过API进行更新.

声明例子:

```glsl
uniform vec4 lightPosition;
```

可用的**unifrom**变量的数量跟实现相关,如果超出则会导致编译时或连接时错误.Uniform变量声明后但未被使用不会被计数.用于定义的unifrom变量和内置的unifrom变量是一起计数的.

在顶点着色器和片元着色器中的uniform变量共享一个命名空间.因此在被链接到同一个程序中的不同着色器中定义的同名uniform变量必须有相同的类型和精度.unifrom变量的作用域是所有阶段,例如在顶点着色器中定义的unifrom变量名也可以在片元着色器中使用.

#### 输出变量

着色器输出变量使用**out**限定符声明.它们构成了当前着色器和管线中下个阶段的输出接口.输出变量必须是全局作用域.在着色器执行过程之它们就像普通变量一下,它们的值在着色器退出的时候会被拷贝到管线的下一个阶段.只有在管线下个阶段实际被用到的输出变量才会被赋值;冗余的输出变量声明时允许的.

不能通过**inout**限定符将一个变量同时声明为输入和输出变量.输出变量和输入变量必须名字不一样.

顶点着色器输出变量不能是以下类型:

- 布尔类型
- 不透明类型
- 数组组成的数组
- 结构体组成的数组
- 包含数组的结构体
- 包含结构体的结构体

形状点着色器输出是整型或整型向量时,必须使用插值限定符**flat**

顶点着色器输出声明例子:

```glsl
out vec3 normal;
centroid out vec2 TexCoord;
invariant centroid out vec4 Color;
flat out vec3 myColor;
```

片元着色器的输出变量不能是以下类型:

- 布尔类型
- 不透明类型
- 矩阵
- 结构体

片元着色器输出被声明为数组时只能通过常量表达式被索引.

片元着色器输出变量声明例子:

```glsl
out vec4 FragmentColor;
out unit Luminosity;
```

#### 布局限定符

##### 输入布局限定符

顶点着色器允许在输入变量声明上加布局限定符,形式如下:

layout(location=整数)

只接受一个参数.例如:

```glsl
layout(location = 3) in vec4 normal;
```

意味着顶点着色器输入变量normal将会从向量第3个位置拷贝.

如果在着色器中没有给位置赋值,而是通过OpenGL ES API赋值,那么API赋值的位置将会被使用.否则,会使用编译器赋值的位置.

片元着色器没有输入布局限定符.

##### 输出布局限定符

顶点着色器没有输出布局限定符.

在片段着色器中,在输出变量和编号的绘制缓冲区之间建立了绑定,通过输出声明中的位置布局限定符.每个输出的位置对应于绘图缓冲区要被写入的位置.位置合法值范围是[0, MAX_DRAW_BUFFERS-1].

输出布局限定符形式如下:

layout(location=整数)

例子:

```glsl
layout(location = 3) out vec4 color;
```

意味着片元着色器输出变量color会被拷贝到绘图缓冲区的位置3处.

如果片元着色器输出是数组,将会从指定位置处连续赋值.例如:

```glsl
layout(location = 2) out vec4 colors[3];
```



意味着colors会被拷贝到绘图缓冲区的2,3,4位置.

如果只有单个输出,则不需要指定位置,这种情况下位置默认为0,例如:

```glsl
out vec4 my_FragColor; // 必须只有一个输出声明
```

如果有多个输出,则必须为每个输出指定位置.

#### 插值

插值类型是由centroid in和centroid out还有smooth和flat决定的.当没有插值限定符时,平滑插值被使用.超过一个插值限定符会引发编译错误.

限定为flat的变量不会被插值.对每个三角形中片元都有相同的值.这个值来自于单个顶点.变量可以被限定为flat centroid,这和只限定为flat是一样的.

限定为smooth的变量将会以透视正确的方式对被渲染的基元插值.透视正确的插值方式在OpenGL ES 3.0规范的3.5节"线段"中描述.

如果是单次采样,值会被插入到像素的中心,centroid限定符会被忽略.如果是多重采样,并且变量没有被限定为centroid,那么值必须被插入到像素中心,或者像素的任何位置,或者到某个像素样本位置.如果是多重采样,并且变量限定为centroid,那么值必须被插值到这样的一个点,这个点即在像素上又在被渲染的基元上,或者其中某个落在基元内像素样品.由于质心的位置缺少规律,它产生的结果跟非质心插值变量比起来更不精确.

#### 链接顶点输出和片元输入

顶点输出的类型和片元输入的同名变量的类型必须匹配,否则链接将会失败.精度不许要匹配.

### 精度和精度限定符

#### 范围和精度

highp浮点变量的精度是32位浮点数.也支持非数字和无穷.

可以使用精度限定符声明存储要求.

highp浮点值以IEEE 754单精度浮点格式存储.Mediump和lowp浮点值具有以下要求的最小范围和精度以及IEEE 754定义的最大范围和精度.

假设所有的整数类型都被实现为整数,因此可能不会被浮点数模拟值.Highp有符号整数表示为32位有符号整数的补码 .Highp无符号整数表示位无符号32位整数.Mediump整数(有符号和无符号)必须被表示为16到32位之间的整数.Lowp整数(有符号和无符号)必须表示为9到32位之间的整数.

精度限定符要求的精度如下表:

| 限定符  | 浮点范围        | 浮点精度  | 有符号整数      | 无符号整数 |
| ------- | --------------- | --------- | --------------- | ---------- |
| highp   | (−2^126, 2^127) | 2^-24     | [-2^31, 2^31-1] | [0,2^32-1] |
| mediump | (−2^14, 2^14)   | 2^-10     | [-2^15, 2^15-1] | [0,2^16-1] |
| lowp    | (-2, 2)         | 2^-8/2^-9 | [-2^8, 2^8-1]   | [0,2^9-1]  |

实际的范围和精度可以通过API查询,请查看OpenGL ES 3.0规范.

#### 精度转换

从低精度到高精度的转换必须是精确的.从高精度到低精度转换时,如果值可以被目标精度表达,那转换必须是精确的,如果值不能被表达,行为取决于类型:

- 对于有符号和无符号整数,值会被截断.
- 对于浮点数,值应该是+INF或-INF,或者实现支持的最大值和最小值.

#### 精度限定符

浮点,整数或采样器类型声明前面都可以加精度限定符:

| 限定符  | 意义                                                 |
| ------- | ---------------------------------------------------- |
| highp   | Highp变量拥有最大的范围和精度,但是可能会导致操作变慢 |
| mediump | Mediump变量典型应用是存储动态范围颜色和低精度几何    |
| lowp    | Lowp变量典型应用是存储8位颜色值                      |

例如:

```glsl
lowp float color;
out mediump vec2 P;
lowp ivec2 foo(lowp mat3);
highp mat4 m;
```

字面常量不能使用精度限定符.布尔变量和构造函数也不能使用.

```glsl
uniform highp float h1;
highp float h2 = 2.3 * 4.7; // operation and result are highp precision
mediump float m;
m = 3.7 * h1 * h2; // all operations are highp precision
h2 = m * h1; // operation is highp precision
m = h2 – h1; // operation is highp precision
h2 = m + m; // addition and result at mediump precision
void f(highp float p);
f(3.3); // 3.3 will be passed in at highp precision
```

不同着色器中声明的同名uniform变量的精度必须相同.

### 默认精度限定符

精度语句可以用来给类型指定默认的精度限定符,格式如下

​	precision精度限定符 类型;

类型可以是int或float或采样器类型,精度限定符可以是lowp,mediump,highp.

顶点着色器有下面几个预先声明好的默认精度语句:

```glsl
precision highp float;
precision highp int;

precision lowp samler2D;
precision lowp samlerCube;
```

片元着色器有下面几个预先声明好的默认精度语句:

```glsl
precision mediump int;
precision lowp sampler2D;
precision lowp samplerCube;
```

片元着色器中没有为浮点类型指定默认精度限定符,浮点向量和矩阵也都没有.这些类型声明的时候必须加上精度限定符或者声明之前先为这个类型声明默认的精度限定符.

顶点着色和片元着色器都没有声明默认精度限定符的类型:

```glsl
sampler3D;
samplerCubeShadow;
sampler2DShadow;
sampler2DArray;
sampler2DArrayShadow;
isampler2D;
isampler3D;
isamplerCube;
isampler2DArray;
usampler2D;
usampler3D;
usamplerCube;
usampler2DArray;
```

### 变异和不变限定符

变异指的是相同表达式在不同程序中有可能值不一样.例如,在不同程序中的两个顶点着色器,各自使用相同的表达式给gl_Position赋值,表达式的输入值也是一样的,但是有可能被赋给gl_Position的值不严格相等,因为两个着色器是独立编译的.这可能导致几何体对齐方面的问题.

#### 不变限定符

为了确保输出变量不可变,需要使用**invariant**限定符.可以先声明后在用invariant限定:

```glsl
invariant gl_Position; // 确保gl_Position内置变量不变化
out vec3 Color;
invariant Color; // 确保color变量不变化
invariant Color_2; // 错误:Color_2没有声明
```

也可以直接在变量声明的时候限定

```glsl
invariant centroid out vec3 Color;
```

不变限定符必须出现在插值限定符或存储限定符之前.只有着色器的输出变量才能设置为不可变的.包括用于定义的输出变量和内置输出变量.因为只有输出才能被声明为不可变的,上一个着色器阶段的不变输出会匹配下一个阶段的输入,即使输入没有声明为不可变的.

默认情况下,所有的输出变量都是可变的.为了强制使所有输出变量为不可变的,使用了如下pragma

```glsl
#pragma STDGL invariant(all)
```

片元着色器中不可使用这个pragma.

保证不变性的代价是优化灵活性,所以会导致性能降低.建议只是在调试的时候使用这个prama,实际情况下避免把所有输出变量都声明为不可变的.

### 限定符顺序

当有多个限定符时,他们的顺序如下

​	不变限定符 插值限定符 存储限定符 精度限定符

​	存储限定符 参数限定符 精度限定符

## 操作符和表达式

### 操作符

OpenGL ES Shading语言有这些操作符

| 优先级     | 操作类型           | 操作符                             | 结合律   |
| ---------- | ------------------ | ---------------------------------- | -------- |
| 1(最高级)  | 括号分组           | ()                                 | NA       |
| 2          | 数组下标           | []                                 | 从左到右 |
| 2          | 函数调用和构造函数 | ()                                 | 从左到右 |
| 2          | 字段或方法选择器   | .                                  | 从左到右 |
| 2          | 后置自加和自减     | ++ --                              | 从左到右 |
| 3          | 前置自加和自减     | ++ --                              | 从右到左 |
| 4          | 乘法               | * / %                              | 从右到左 |
| 5          | 加法               | ＋－                               | 从左到右 |
| 6          | 位移               | <<  >>                             | 从左到右 |
| 7          | 关系               | <  >  <=  >=                       | 从左到右 |
| 8          | 相等               | ===  !=                            | 从左到右 |
| 9          | 位与               | &                                  | 从左到右 |
| 10         | 位异或             | ^                                  | 从左到右 |
| 11         | 位或               | \|                                 | 从左到右 |
| 12         | 逻辑与             | &&                                 | 从左到右 |
| 13         | 逻辑异或           | ^^                                 | 从左到右 |
| 14         | 逻辑或             | \|\|                               | 从左到右 |
| 15         | 选择               | ? :                                | 从右到左 |
| 16         | 赋值               | = += -= *= /= %= <<= >>= &= ^= \|= | 从右到左 |
| 17(最低级) | 序列               | ,                                  | 从左到右 |

没有地址操作符和引用操作符.没有类型转换符,使用构造函数替代类型装换操作符.

### 构造函数

构造函数使用函数调用语法,函数名就是类型名,调用会创建一个该类型的对象.构造函数在初始化器和表达式中使用方式相同.参数被用来初始化结构值.构造函数可以用来从一种标量类型转换成另一种标量类型.

对于数组和结构体,其中的每个元素或字段的构造函数只能有一个参数.对于其他类型,参数必须提供初始化所需要的足够数量的组件,过多的参数会导致错误.

#### 转换和标量构造函数

标量类型之间的转换如下:

```glsl
int(bool) // bool转int
int(float) // float转int
float(bool) // bool转float
float(int) // int转float
bool(float) // float转bool
bool(int) // int转bool
uint(bool) // bool转uint
uint(float) // float转uint
uint(int) // int转uint
int(uint) // uint转int
bool(uint) // uint转bool
float(uint) // uint转float
```

当从float转为int或uint时,float的小数部分会被丢弃.从负浮点数转换为uint是未定义的.

当从int,uint或float转bool时,0或0.0被转换为false,非0值转换为true.当从bool转int,uint或float时,false转换为0或0.0,true转换为1或1.0.

标量构造函数的参数如果是一个非标量,那么会使用非标量里的第一个元素,例如float(vec3)将会使用vec3参数的第一个元素.

#### 向量和矩阵构造函数

如果向量构造函数的参数是单个标量,向量所有组件都会被初始化为这个标量值.如果矩阵构造函数的参数是单个标量值,矩阵的对角线上的元素会被初始化为这个标量值,其它组件被初始化为0.0.

如果向量构造函数的参数是多个标量,一个或多个向量,一个或多个矩阵,或者这些的混合体,向量的组件将会由这些参数的组件按照顺序构成.参数按照从左到右的顺序,依次把组件填充到向量中.

如果矩阵构造函数是多个标量或向量,或者这些的混合体,矩阵将按照列优先的顺序被填充.在这种情况下,必须提供足够的参数来初始化矩阵的每个组件,如果超出了需要的数量会报错.

如果矩阵构造函数参数是矩阵,那么使用参数中的[i,j]来初始化矩阵中对应位置的组件,其他组件将按照单位矩阵来初始化.如果参数是矩阵,那么就不能有其他任何参数了.

当构造函数参数的基本类型(bool,int或float)不能跟被构造对象的基本类型匹配时,上一节中描述的转换规则将会被应用到参数上.

向量构造函数用法:

```glsl
vec3(float) // 使用float初始化vec3的每个组件
vec4(ivec4) // 创建vec4,并且每个组件都从int转float
vec4(mat2) // vec4按照第0列和第1列的顺序构成
vec2(float, float) // 使用两个float初始化vec2
ivec3(int, int, int) // 使用3个int初始化ivec3
bvec4(int, int, float, float) // 创建bvec4,每个组件都有类型转换
vec2(vec3) // 丢掉vec3的第三个组件
vec3(vec4) // 丢掉vec4的第四个组件
vec3(vec2, float) // vec3.x = vec2.x, vec3.y = vec2.y, vec3.z = float
vec3(float, vec2) // vec3.x = float, vec3.y = vec2.x, vec3.z = vec2.y
vec4(vec3, float)
vec4(float, vec3)
vec4(vec2, vec2)
```

向量构造函数例子:

```glsl
vec4 color = vec4(0.0, 1.0, 0.0, 1.0);
vec4 rgba = vec4(1.0); // 每个组件都设置为1.0
vec3 rgb = vec3(color); // 丢掉第四个元素
```

矩阵构造函数用法:

```glsl
mat2(vec2, vec2); // 每个参数一列
mat3(vec3, vec3, vec3); // 每个参数一列
mat4(vec4, vec4, vec4, vec4); // 每个参数一列
mat3x2(vec2, vec2, vec2); // 每个参数一列
mat2(float, float, // 第一列
float, float); // 第二列
mat3(float, float, float, // 第一列
float, float, float, // 第二列
float, float, float); // 第三列
mat4(float, float, float, float, // 第一列
float, float, float, float, // 第二列
float, float, float, float, // 第三列
float, float, float, float); // 第四列
mat2x3(vec2, float, // 第一列
vec2, float); // 第二列
```

#### 结构体构造函数

结构体构造函数和结构体类型名一样.例如:

```glsl
struct light {
    float intensity;
    vec3 position;
};
light lightVar = light(3.0, vec3(1.0, 2.0, 3.0));
```

构造函数的参数将会被设置到结构体字段上,按顺序,每个字段使用一个参数.每个参数的类型要和对应字段的类型一致.

构造函数可以作为初始化器或者在表达式中使用.

#### 数组构造函数

数组构造函数名字和数组类型一样,可以作为初始化器或者在表达式中使用.例如:

```glsl
const float c[3] = float[3](5.0, 7.2, 1.1);
const float d[3] = float[](5.0, 7.2, 1.1);
float g;
...
float a[5] = float[5](g, 1, g, 2.3, g);
float b[3];
b = float[3](g, g + 1.0, g + 2.0);
```

参数的数量必须和数组的长度完全相同.参数会按顺序从0开始赋值给数组的每个元素.每个参数的类型必须和数组元素类型相同.

### 向量组件

向量组件的名字可以用一个简单字母表示.如下表:

| {x,y,z,w} | 当向量代表点或者法向量时使用 |
| --------- | ---------------------------- |
| {r,g,b,a} | 当向量代表颜色时使用         |
| {s,t,p,q} | 当向量代表纹理坐标时使用     |

可以通过

​	向量名.组件名

的方式访问向量中的组件.

组件名x,r,s是同义词,都代表向量的第一个组件,其他几个名字同理.

访问的组件名如果超出了向量范围是错误的,例如:

```glsl
vec2 pos;
pos.x // 合法
pos.z // 非法
```

可以同时通过多个组件名来选中向量中的多个组件,例如:

```glsl
vec4 v4;
v4.rgba; // 结果是vec4,和v4相同
v4.rgb; // 结果是vec3
v4.b; // 结果是float
v4.xy; // 结果是vec2
v4.xgba; // 合法,虽然组件名来自不同的集合
```

选择的组件不能超过4个

```glsl
vec4 v4;
v4.xyzw; // 结果是vec4
v4.xyzwxy; // 非法,因为选择了6个组件
(v4.xyzwxy).xy; // 非法,因为中间值选择了6个组件
vec2 v2;
v2.xyxy; // 合法,结果是vec4
```

组件的顺序可以是随意的,甚至可以是重复的:

```glsl
vec4 pos = vec4(1.0, 2.0, 3.0, 4.0);
vec4 swiz= pos.wzyx; // swiz = (4.0, 3.0, 2.0, 1.0)
vec4 dup = pos.xxyy; // dup = (1.0, 1.0, 2.0, 2.0)
```

这种组件名的组合还可以出现在表达式的左边:

```glsl
vec4 pos = vec4(1.0, 2.0, 3.0, 4.0);
pos.xw = vec2(5.0, 6.0); // pos = (5.0, 2.0, 3.0, 6.0)
pos.wx = vec2(7.0, 8.0); // pos = (8.0, 2.0, 3.0, 7.0)
pos.xx = vec2(3.0, 4.0); // 非法 - 'x'出现了两次
pos.xy = vec3(1.0, 2.0, 3.0); // 非法 - vec3和vec2不匹配
```

向量也可以使用下标访问,下标从0开始.下标中的索引必须是常量,并且不能超出向量的长度.

#### 矩阵组件

矩阵可以使用下标语法访问.矩阵每一列可以看成是vector,矩阵是列vector构成的数组.单个下标用于选中列vector,两个下标则可以同时选中列和行.下标从0开始.

```glsl
mat4 m;
m[1] = vec4(2.0); // 设置第二列的值为2.0
m[0][0] = 1.0; // 设置左上角值为1.0
m[2][3] = 2.0; // 设置第3列第4行值为2.o
```

### 结构体和数组操作

结构体和数组上可以应用这些操作符:

| 字段或方法选择器   | .     |
| ------------------ | ----- |
| 判等               | == != |
| 赋值               | =     |
| 三元操作符         | ?:    |
| 序列操作符         | ,     |
| 索引(只有数组可用) | []    |

判等操作和赋值操作必须针对同一种类型.对于结构体来说,只有当所有字段都相等才算相等.对于数组来说,只有当所有元素相等时才算相等.

数组中的元素可以通过下标操作符[]访问.例如:

```glsl
diffuseColor += lightIntensity[3] * NdotL;
```

数组索引从0开始,索引类型只是能int或uint.

还可以使用点符号访问数组的length方法,查询数组的长度.

```glsl
lightIntensity.length() // 返回数组的长度
```

### 赋值

赋值操作符左边的称为左值,右边的称为右值.赋值操作不会有任何默认的类型转换,必须通过构造函数进行显式转换.

### 向量和矩阵操作

当对向量和矩阵进行操作时,操作会被应用到其中的每个组件上.例如

```glsl
vec3 v, u;
float f;
v = u + f;
```

等于

```glsl
v.x = u.x + f;
v.y = u.y + f;
v.z = u.z + f;
```

例如

```glsl
vec3 v, u, w;
w = v + u;
```

等于

```glsl
w.x = v.x + u.x;
w.y = v.y + u.y;
w.z = v.z + u.z;
```

向量和矩阵之间的乘法,矩阵和矩阵之间的乘法符合线性代数的乘法规则.

```glsl
vec3 v, u;
mat3 m;
u = v * m;
```

等于

```glsl
u.x = dot(v, m[0]); // m[0]是m的最左列
u.y = dot(v, m[1]); // dot(a,b)表示a和b的内积(点积)
u.z = dot(v, m[2]);
```

例如

```glsl
u = m * v;
```

等于

```glsl
u.x = m[0].x * v.x + m[1].x * v.y + m[2].x * v.z;
u.y = m[0].y * v.x + m[1].y * v.y + m[2].y * v.z;
u.z = m[0].z * v.x + m[1].z * v.y + m[2].z * v.z;
```

例如

```glsl
mat3 m, n, r;
r = m * n;
```

等于

```glsl
r[0].x = m[0].x * n[0].x + m[1].x * n[0].y + m[2].x * n[0].z;
r[1].x = m[0].x * n[1].x + m[1].x * n[1].y + m[2].x * n[1].z;
r[2].x = m[0].x * n[2].x + m[1].x * n[2].y + m[2].x * n[2].z;
r[0].y = m[0].y * n[0].x + m[1].y * n[0].y + m[2].y * n[0].z;
r[1].y = m[0].y * n[1].x + m[1].y * n[1].y + m[2].y * n[1].z;
r[2].y = m[0].y * n[2].x + m[1].y * n[2].y + m[2].y * n[2].z;
r[0].z = m[0].z * n[0].x + m[1].z * n[0].y + m[2].z * n[0].z;
r[1].z = m[0].z * n[1].x + m[1].z * n[1].y + m[2].z * n[1].z;
r[2].z = m[0].z * n[2].x + m[1].z * n[2].y + m[2].z * n[2].z;
```

## 语句和结构

OpenGL ES Shading语言的基本模块:

- 语句和声明
- 函数定义
- 选择(if-else和switch-case-default)
- 迭代(for,while和do-while)
- 跳转(discard,return,break和continue)

着色器是一系列的声明和函数体组成:

​	函数定义:

​		函数原型 { 语句列表 }

​	语句列表:

​		语句

​	语句:

​		组合语句

​		简单语句

​	组合语句:

​		{ 语句列表 }

​	简单语句:

​		声明语句

​		表达式语句

​		选择语句

​		迭代语句

​		跳转语句

简单语句,表达式和跳转语句以分号结尾;

### 函数定义

函数声明:

```glsl
// prototype
returnType functionName (type0 arg0, type1 arg1, ..., typen argn);
```

函数定义:

```glsl
// definition
returnType functionName (type0 arg0, type1 arg1, ..., typen argn)
{
    // do some computation
    return returnValue;
}
```

或者

```glsl
void functionName (type0 arg0, type1 arg1, ..., typen argn)
{
    // do some computation
    return; // optional
}
```

函数支持重载,即函数名相同,参数不同.当调用重载函数时,会对所有参数进行精确的类型匹配.例如:

```glsl
vec4 f(in vec4 x, out vec4 y);
vec4 f(in vec4 x, out ivec4 y); // 允许, 参数类型不同
int f(in vec4 x, out ivec4 y); // 错误, 只有返回值类型不同
vec4 f(in vec4 x, in ivec4 y); // 错误, 只有限定符不同
int f(const in vec4 x, out ivec4 y); // 错误, 只有限定符不同
```

调用情况:

```glsl
f(vec4, vec4) // 精确匹配到vec4 f(in vec4 x, out vec4 y)
f(vec4, ivec4) // 精确匹配到vec4 f(in vec4 x, out ivec4 y)
f(ivec4, vec4) // 错误, 没有匹配的函数.
f(ivec4, ivec4) // 错误, 没有匹配的函数.
```

用户定义的函数可被声明多次,但是能定义一次.

不能重定义或重载内置函数.

main函数作为着色器执行的入口.所有顶点着色器和片元着色器都必须定义main函数.这个函数没有参数,返回值必须为void:

```glsl
void main()
{
	...
}
```

### 选择

条件控制流通过if,if-else或switch语句完成:

​	选择语句:
		if ( 布尔表达式 ) 语句
		if ( 布尔表达式 ) 语句 else 语句
		switch ( 初始表达式 ) { switch语句列表 }

​	switch语句:
		case 常量表达式:
		default: 语句

初始表达式类型必须是整数.

### 迭代

For,while和do循环个格式:
	for (初始表达式; 条件表达式; 循环表达式)
		子语句

​	while (条件表达式)
		子语句
	
​	do
		语句
	while (条件表达式)

### 跳转

跳转语句:
	continue;
	break;
	return;
	return 表达式;
	discard; // 只能在片元着色器中使用

不允许goto语句.

discard关键字会导致片元被丢弃并且不会更新任何缓冲区.它通常会和条件语句一起使用,例如:

```glsl
if (intensity < 0.0)
	discard;
```

片段着色器可以测试片段的alpha值,并基于该测试丢弃片段 .

然而,应该注意的是,覆盖测试发生在片段着色器运行之后,覆盖测试可能改变alpha值.

## 内置变量

### 顶点着色器特殊变量

有些OpenGL ES操作发生在顶点处理器和片元处理器之间的固定功能中.着色器通过内置变量与OpenGL ES固定功能通信.

顶点着色器中有这几个内置变量:

```glsl
in highp int gl_VertexID;
in highp int gl_InstanceID;
out highp vec4 gl_Position;
out highp float gl_PointSize;
```

变量gl_Position用于写入齐次顶点坐标.它可以在着色器执行的任何时候被写入.这个值将会被基元装配,裁剪,剔除等固定功能操作使用.

变量gl_PointSize用于写入栅格化点的尺寸,单位是像素.

变量gl_VertexID是顶点着色器的输入变量,用于保存顶点的整数索引.

变量gl_InstanceID是顶点着色器输入变量,用于保存在一次绘制调用中的当前基元的实例号.

### 片元着色器特殊变量

片元着色器的内置特殊变量如下:

```glsl
in highp vec4 gl_FragCoord;
in bool gl_FrontFacing;
out highp float gl_FragDepth;
in mediump vec2 gl_PointCoord;
```

片元着色器可执行文件的输出由OpenGL ES管道后端的固定函数操作处理.

固定功能通过读取gl_FragCoord.z来计算片元的深度.

向gl_FragDepth写入值会建立片元的深度值.如果深度缓冲区启用,gl_FragDepth又没被写入值,深度值将会是一个固定值.如果对gl_FragDepth赋值的代码分支没有被执行,片元深度值可能是undefined.

如果着色器执行了discard关键字,片元将会被丢弃,所有用户定义的片元输出都会变得无关紧要.

变量gl_FragCoord是片元着色器的输入变量,用于保存窗口相关坐标(x,y,z,1/w).如果是多重采样,值可能在像素的任何位置,或者片元样品的其中一个.使用centroid in并不能进一步限制这个值在当前基元的内部.这个值是顶点处理之后经过插值传递给片元.z组件是深度值,将会被用来做片元深度.

如果变量gl_FrontFacing为true,片元属于前面基元.它的一个用途是通过选择来模拟双面光照.

### 内置常量

内置常量如下:

```glsl
// 实现相关的值. 下面展示的是允许的最小值

const mediump int gl_MaxVertexAttribs = 16;
const mediump int gl_MaxVertexUniformVectors = 256;
const mediump int gl_MaxVertexOutputVectors = 16;
const mediump int gl_MaxFragmentInputVectors = 15;
const mediump int gl_MaxVertexTextureImageUnits = 16;
const mediump int gl_MaxCombinedTextureImageUnits = 32;
const mediump int gl_MaxTextureImageUnits = 16;
const mediump int gl_MaxFragmentUniformVectors = 224;
const mediump int gl_MaxDrawBuffers = 4;
const mediump int gl_MinProgramTexelOffset = -8;
const mediump int gl_MaxProgramTexelOffset = 7;
```

### 内置Uniform状态

```glsl
//
// 窗口坐标深度,
//
struct gl_DepthRangeParameters {
highp float near; // n
highp float far; // f
highp float diff; // f - n
};
uniform gl_DepthRangeParameters gl_DepthRange;
```

## 内置函数

OpenGL ES Shading语言为标量和向量操作提供了各种各样的内置方便函数.

内置函数可以分为三类:

- 它们以一种方便的方式公开一些必要的硬件功能,比如访问纹理地图.语言中没有办法让这些函数被着色器模拟.
- 它们表示一个简单的操作(裁剪、混合等),用户可以很容易地编写这些操作,但是它们是非常常见的,可能有直接的硬件支持.对于编译器来说,将表达式映射到复杂的汇编指令是一个非常困难的问题.
- 它们代表了一个操作图形硬件可能会在某个时候加速.三角函数属于这一类

应用程序应该尽可能的使用这些内置函数,而不是自己编写相同功能的函数.

genType表示float,vec2,vec3,vec4.genIType表示int,ivec2,ivec3,ivec3.genUType表示uint,uvec2,uvec3,uvec4.genBType表示bool,bvec2,bvec3,bvec4.

### 角度和三角函数

函数参数angle以弧度为单位.所有的操作都是针对每个组件的.

| 语法                                | 描述                                        |
| ----------------------------------- | ------------------------------------------- |
| genType radians (genType degrees)   | 度转换为弧度                                |
| genType degrees (genType radians)   | 弧度转度                                    |
| genType sin (genType angle)         | 三角函数sin                                 |
| genType cos (genType angle)         | 三角函数cos                                 |
| genType tan (genType angle)         | 三角函数tan                                 |
| genType asin (genType x)            | 反三角函数asin                              |
| genType acos (genType x)            | 反三角函数acos                              |
| genType atan (genType y, genType x) | 反三角函数atan,[-π,π],x和y为零返回undefined |
| genType atan (genType y_over_x)     | 反三角函数atan,[-π/2,π/2]                   |
| genType sinh (genType x)            | 双曲线正弦                                  |
| genType cosh (genType x)            | 双曲线余弦                                  |
| genType tanh (genType x)            | 双曲线正切                                  |
| genType asinh (genType x)           | 反双曲线正弦                                |
| genType acosh (genType x)           | 反双曲线余弦,x<1返回undefined               |
| genType atanh (genType x)           | 反双曲线正切\|x\|>=1返回undefined           |

###指数函数

| 语法                               | 描述             |
| ---------------------------------- | ---------------- |
| genType pow (genType x, genType y) | 求x的y次幂       |
| genType exp (genType x)            | 求e的x次幂       |
| genType log (genType x)            | 求以e为底x的对数 |
| genType exp2 (genType x)           | 求2的x次幂       |
| genType log2 (genType x)           | 求以2为底x的对数 |
| genType sqrt (genType x)           | x开根号          |
| genType inversesqrt (genType x)    | x开根号的倒数    |

### 公共函数

请查阅原文

### 浮点打包和解包

请查阅原文

### 几何函数

请查阅原文

###矩阵函数

请查阅原文

###向量关系函数

请查阅原文

###纹理查询函数

请查阅原文

###片元处理函数

请查阅原文