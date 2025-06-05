# 高级材质与纹理

在 Three.js 中，材质和纹理是创建真实感 3D 场景的关键要素。通过合理使用高级材质和纹理，可以让场景更加生动逼真。

## 高级材质类型

### 1. MeshStandardMaterial

标准材质是最常用的 PBR（基于物理的渲染）材质，支持多种贴图类型。

![标准材质效果](https://threejs.org/examples/screenshots/webgl_materials_standard.jpg)

*图 6.1: 标准材质效果示例*

```javascript
// 创建标准材质
const material = new THREE.MeshStandardMaterial({
    color: 0xffffff,          // 基础颜色
    metalness: 0.5,           // 金属度
    roughness: 0.5,           // 粗糙度
    envMap: envMap,           // 环境贴图
    envMapIntensity: 1.0,     // 环境贴图强度
    aoMap: aoMap,             // 环境光遮蔽贴图
    aoMapIntensity: 1.0,      // 环境光遮蔽强度
    displacementMap: dispMap, // 置换贴图
    displacementScale: 0.1,   // 置换强度
    normalMap: normalMap,     // 法线贴图
    normalScale: new THREE.Vector2(1, 1), // 法线贴图强度
    roughnessMap: roughMap,   // 粗糙度贴图
    metalnessMap: metalMap,   // 金属度贴图
    emissive: 0x000000,       // 自发光颜色
    emissiveMap: emissiveMap, // 自发光贴图
    emissiveIntensity: 1.0    // 自发光强度
});
```

### 2. MeshPhysicalMaterial

物理材质是标准材质的扩展，增加了更多物理特性。

```javascript
// 创建物理材质
const material = new THREE.MeshPhysicalMaterial({
    // 继承自 MeshStandardMaterial 的所有属性
    clearcoat: 1.0,           // 清漆层强度
    clearcoatRoughness: 0.1,  // 清漆层粗糙度
    clearcoatMap: clearcoatMap, // 清漆层贴图
    clearcoatRoughnessMap: clearcoatRoughnessMap, // 清漆层粗糙度贴图
    clearcoatNormalMap: clearcoatNormalMap, // 清漆层法线贴图
    clearcoatNormalScale: new THREE.Vector2(1, 1), // 清漆层法线贴图强度
    ior: 1.5,                 // 折射率
    reflectivity: 0.5,        // 反射率
    sheen: 1.0,               // 光泽度
    sheenRoughness: 0.5,      // 光泽度粗糙度
    sheenColor: 0xffffff,     // 光泽度颜色
    sheenColorMap: sheenColorMap, // 光泽度颜色贴图
    sheenRoughnessMap: sheenRoughnessMap, // 光泽度粗糙度贴图
    transmission: 0.0,        // 透射率
    transmissionMap: transmissionMap, // 透射率贴图
    thickness: 0.5,           // 厚度
    thicknessMap: thicknessMap, // 厚度贴图
    attenuationDistance: 0.5, // 衰减距离
    attenuationColor: 0xffffff // 衰减颜色
});
```

### 3. MeshToonMaterial

卡通材质用于创建卡通风格的渲染效果。

```javascript
// 创建卡通材质
const material = new THREE.MeshToonMaterial({
    color: 0xffffff,          // 基础颜色
    gradientMap: gradientMap, // 渐变贴图
    emissive: 0x000000,       // 自发光颜色
    emissiveMap: emissiveMap, // 自发光贴图
    emissiveIntensity: 1.0    // 自发光强度
});
```

## 纹理映射

### 1. 基础纹理

```javascript
// 创建纹理加载器
const textureLoader = new THREE.TextureLoader();

// 加载纹理
const texture = textureLoader.load('texture.jpg',
    // 加载完成回调
    (texture) => {
        console.log('Texture loaded');
    },
    // 加载进度回调
    (xhr) => {
        console.log((xhr.loaded / xhr.total * 100) + '% loaded');
    },
    // 加载错误回调
    (error) => {
        console.error('Error loading texture:', error);
    }
);

// 设置纹理参数
texture.wrapS = THREE.RepeatWrapping;  // 水平重复
texture.wrapT = THREE.RepeatWrapping;  // 垂直重复
texture.repeat.set(2, 2);              // 重复次数
texture.offset.set(0.5, 0.5);          // 偏移
texture.center.set(0.5, 0.5);          // 中心点
texture.rotation = Math.PI / 4;         // 旋转
```

### 2. 环境贴图

```javascript
// 创建环境贴图加载器
const cubeTextureLoader = new THREE.CubeTextureLoader();

// 加载环境贴图
const envMap = cubeTextureLoader.load([
    'px.jpg', 'nx.jpg',
    'py.jpg', 'ny.jpg',
    'pz.jpg', 'nz.jpg'
]);

// 设置场景环境
scene.environment = envMap;
scene.background = envMap;
```

### 3. 多纹理组合

```javascript
// 加载多个纹理
const colorMap = textureLoader.load('color.jpg');
const normalMap = textureLoader.load('normal.jpg');
const roughnessMap = textureLoader.load('roughness.jpg');
const aoMap = textureLoader.load('ao.jpg');

// 创建材质
const material = new THREE.MeshStandardMaterial({
    map: colorMap,
    normalMap: normalMap,
    roughnessMap: roughnessMap,
    aoMap: aoMap,
    aoMapIntensity: 1.0
});
```

## 自定义着色器

### 1. 基础着色器材质

```javascript
// 创建着色器材质
const material = new THREE.ShaderMaterial({
    uniforms: {
        time: { value: 0 },
        color: { value: new THREE.Color(0xffffff) }
    },
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    fragmentShader: `
        uniform float time;
        uniform vec3 color;
        varying vec2 vUv;
        void main() {
            vec3 finalColor = color * (0.5 + 0.5 * sin(time + vUv.x * 10.0));
            gl_FragColor = vec4(finalColor, 1.0);
        }
    `
});

// 在动画循环中更新
function animate() {
    requestAnimationFrame(animate);
    material.uniforms.time.value += 0.01;
    renderer.render(scene, camera);
}
```

### 2. 后处理效果

```javascript
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';

// 创建后处理器
const composer = new EffectComposer(renderer);

// 添加渲染通道
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// 添加泛光效果
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    1.5,  // 强度
    0.4,  // 半径
    0.85  // 阈值
);
composer.addPass(bloomPass);

// 在动画循环中使用后处理器
function animate() {
    requestAnimationFrame(animate);
    composer.render();
}
```

## 材质优化

### 1. 纹理优化

```javascript
// 设置纹理参数
texture.minFilter = THREE.LinearFilter;  // 最小过滤
texture.magFilter = THREE.LinearFilter;  // 最大过滤
texture.anisotropy = renderer.capabilities.getMaxAnisotropy();  // 各向异性过滤

// 压缩纹理
texture.encoding = THREE.sRGBEncoding;  // sRGB 编码
texture.flipY = false;  // 翻转 Y 轴
```

### 2. 材质优化

```javascript
// 设置材质参数
material.precision = 'highp';  // 精度
material.flatShading = false;  // 平面着色
material.vertexColors = false; // 顶点颜色
material.side = THREE.FrontSide;  // 渲染面
```

### 3. 性能优化

```javascript
// 使用纹理压缩
const compressedTexture = textureLoader.load('texture.jpg', (texture) => {
    texture.encoding = THREE.sRGBEncoding;
    texture.generateMipmaps = false;
});

// 使用实例化渲染
const instancedMaterial = new THREE.MeshStandardMaterial({
    map: texture,
    instanced: true
});
```

## 实战：创建一个高级材质场景

让我们创建一个展示各种高级材质的场景：

```javascript
// 创建场景
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a1a);

// 创建相机
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.set(5, 5, 5);
camera.lookAt(0, 0, 0);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.0;
document.body.appendChild(renderer.domElement);

// 加载纹理
const textureLoader = new THREE.TextureLoader();
const colorMap = textureLoader.load('color.jpg');
const normalMap = textureLoader.load('normal.jpg');
const roughnessMap = textureLoader.load('roughness.jpg');
const aoMap = textureLoader.load('ao.jpg');

// 创建环境贴图
const cubeTextureLoader = new THREE.CubeTextureLoader();
const envMap = cubeTextureLoader.load([
    'px.jpg', 'nx.jpg',
    'py.jpg', 'ny.jpg',
    'pz.jpg', 'nz.jpg'
]);
scene.environment = envMap;

// 创建标准材质
const standardMaterial = new THREE.MeshStandardMaterial({
    map: colorMap,
    normalMap: normalMap,
    roughnessMap: roughnessMap,
    aoMap: aoMap,
    envMap: envMap,
    metalness: 0.5,
    roughness: 0.5
});

// 创建物理材质
const physicalMaterial = new THREE.MeshPhysicalMaterial({
    map: colorMap,
    normalMap: normalMap,
    roughnessMap: roughnessMap,
    aoMap: aoMap,
    envMap: envMap,
    clearcoat: 1.0,
    clearcoatRoughness: 0.1,
    metalness: 0.5,
    roughness: 0.5
});

// 创建物体
const geometry = new THREE.SphereGeometry(1, 32, 32);
const standardMesh = new THREE.Mesh(geometry, standardMaterial);
standardMesh.position.x = -2;
scene.add(standardMesh);

const physicalMesh = new THREE.Mesh(geometry, physicalMaterial);
physicalMesh.position.x = 2;
scene.add(physicalMesh);

// 添加光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1.0);
directionalLight.position.set(5, 5, 5);
scene.add(directionalLight);

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    standardMesh.rotation.y += 0.01;
    physicalMesh.rotation.y += 0.01;

    renderer.render(scene, camera);
}

animate();
```

## 练习

1. 创建一个包含多种材质的场景
2. 实现材质的动态变化
3. 添加自定义着色器效果
4. 实现后处理效果

## 下一步学习

在下一章中，我们将学习：
- 粒子系统与特效
- 粒子动画
- 粒子材质
- 性能优化
