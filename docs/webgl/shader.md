## shader 基础

#### 变量常用类型

- vec2：浮点数的二维向量
- vec3：浮点数的三维向量
- mat4：包含 4 列和 4 行浮点数的矩阵
- sampler2D：2D 纹理图像
  > 两个着色器中使用的所有变量都必须被赋予一个类型，并且分配给该变量的任何数字都必须与其类型一致。

#### 内置变量

| 变量         | 描述                           |
| ------------ | ------------------------------ |
| gl_Position  | 提供屏幕坐标的位置数据         |
| gl_FragColor | 为屏幕上的分面表示提供颜色数据 |

#### 内置输入

| 属性           | 类型 | 描述                                |
| -------------- | ---- | ----------------------------------- |
| position       | vec3 | 像素：顶点位置 / 片段：面部像素位置 |
| normal（法线） | vec3 | 像素：顶点法线 / 片段：面部像素法线 |
| uv             | vec2 | 纹理坐标                            |

| Uniform             | 类型  | 描述                                      |
| ------------------- | ----- | ----------------------------------------- |
| world               | mat4  | mesh 全局变换状态 (move + rotate + scale) |
| worldView           | mat4  | global view part of mesh                  |
| worldViewProjection | mat4  | 全局摄像机                                |
| view                | mat4  | mesh local view part                      |
| projection          | mat4  | 局部摄像机                                |
| time                | float | 每一帧                                    |

#### 第一个 shader 程序

```c
void main(void) {
  // *code*
}

```

![shader 示意图](/assets/shade3.jpg)

#### precision

是一种在 webgl 着色器中设置浮点数精度的语句，格式: precision mediump float;
其中 precision 是固定的

第二个关键字
|值|含义|
|-|-|
|lowp|低精度|
|mediump|中等精度|
|highp|高精度|

第三个关键字：int float

#### 在 babylon 中使用 shader

1. 代码中直接导入

```typescript
// 设置顶点着色器(注意这里的命名规则)
BABYLON.Effect.ShadersStore['customVertexShader'] = `
    precision highp float;
    // 属性
    attribute vec3 position;
    attribute vec2 uv;
    // 全局变量
    uniform mat4 worldViewProjection;
    // 可变量
    varying vec2 vUV;
    // 主函数
    void main(void) {
        gl_Position = worldViewProjection * vec4(position, 1.0);
        vUV = uv;
    }`;
// 设置像素着色器(注意这里的命名规则)
BABYLON.Effect.ShadersStore['customFragmentShader'] = `
    precision highp float;
    // 可变量
    varying vec2 vUV;
    // 全局变量
    uniform sampler2D textureSampler;
    void main(void) {
        float luminance = dot(texture2D(textureSampler, vUV).rgb, vec3(0.3, 0.59, 0.11));
        gl_FragColor = vec4(luminance, luminance, luminance, 1.0);
    }`;
const shaderMaterial = new BABYLON.ShaderMaterial(
  'shader',
  scene,
  {
    // 这里应用了着色器
    vertex: 'custom',
    fragment: 'custom',
    // 也可以直接使用 vertexSource fragmentSource 把着色器代码放进来
  },
  {
    // 属性
    attributes: ['position', 'normal', 'uv'],
    // 全局变量
    uniforms: [
      'world',
      'worldView',
      'worldViewProjection',
      'view',
      'projection',
    ],
  },
);
// 给材质添加纹理
const mainTexture = new BABYLON.Texture('amiga.jpg', scene); // 创建纹理
shaderMaterial.setTexture('textureSampler', mainTexture);
// 给mesh使用材质
mesh.material = shaderMaterial;
```

2. [script 标签导入](https://doc.babylonjs.com/features/featuresDeepDive/materials/shaders/shaderCodeInBjs#shader-code-in-script-tags)
3. [fx 文件导入](https://doc.babylonjs.com/features/featuresDeepDive/materials/shaders/shaderCodeInBjs#shader-code-in-fx-files)

> 着色器 Includes
>
> 添加到 `IncludeShadersStore` 中
>
> ```typescript
> BABYLON.Effect.IncludesShadersStore['includeName'] = '...';
> ```
>
> 导入
>
> ```c
> #include<includeName>
> ```

#### 着色器生成器 `没看懂`

- babylon.js 的扩展，

```html
<script src="Babylonx.ShaderBuilder.js"></script>
```

```typescript
// 着色器生成器引擎需要初始化
BABYLONX.ShaderBuilder.InitializeEngine();
mesh.material = new BABYLONX.ShaderBuilder()
  .Map({ path: 'textures/amiga.jpg' })
  .BuildMaterial(scene);
```

#### new BABYLON.ShaderMaterial 第四个参数：`option`

- 顶点着色器代码中的任何属性（attribute）都必须出现在 attributes 数组中
- 全局变量（uniform）必须在顶点着色器中声明为类型，并且必须位于 uniforms 数组中 `worldViewProjectionmat4uniforms`
- 数组中命名但未在着色器代码中使用的属性和全局变量将被忽略。
- 如果着色器代码包含#define 值，则可以指定要在数组中激活的值。 `defines`
- 在着色器代码中分配给纹理的全局变量必须存在于采样器数组（samplers）中，其他所有全局变量必须存在于 uniforms 数组中。

```typescript
const myShaderMaterial = new BABYLON.ShaderMaterial(
  'shader',
  scene,
  './COMMON_NAME',
  {
    // 属性
    attributes: ['position', 'normal', 'uv'],
    // 全局变量
    uniforms: [
      'world',
      'worldView',
      'worldViewProjection',
      'view',
      'projection',
      'time',
      'direction',
    ],
    // 纹理变量
    samplers: ['textureSampler'],
    // 指定要激活的变量
    defines: ['MyDefine'],
    needAlphaBlending: true,
    needAlphaTesting: true,
  },
);
```
