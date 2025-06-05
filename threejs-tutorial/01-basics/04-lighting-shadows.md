# 光源与阴影基础

在 Three.js 中，光源和阴影是创建真实感 3D 场景的关键要素。合理的光照设置可以让场景更加生动，而阴影则能增强场景的深度感和真实感。

## 光源类型

Three.js 提供了多种类型的光源，每种光源都有其特定的用途和效果。

### 1. 环境光（AmbientLight）

环境光提供均匀的照明，没有方向性，通常用于模拟环境中的漫反射光。

![环境光效果](https://threejs.org/examples/screenshots/webgl_lights_ambient.jpg)

*图 4.1: 环境光效果示例*

```javascript
// 创建环境光
const ambientLight = new THREE.AmbientLight(
    0xffffff,  // 颜色
    0.5        // 强度
);
scene.add(ambientLight);
```

### 2. 平行光（DirectionalLight）

平行光模拟太阳光，光线平行且强度相同，可以产生清晰的阴影。

![平行光效果](https://threejs.org/examples/screenshots/webgl_lights_directional.jpg)

*图 4.2: 平行光效果示例*

```javascript
// 创建平行光
const directionalLight = new THREE.DirectionalLight(
    0xffffff,  // 颜色
    1.0        // 强度
);

// 设置光源位置
directionalLight.position.set(5, 5, 5);

// 设置光源目标
directionalLight.target.position.set(0, 0, 0);

// 启用阴影
directionalLight.castShadow = true;

// 设置阴影参数
directionalLight.shadow.mapSize.width = 1024;
directionalLight.shadow.mapSize.height = 1024;
directionalLight.shadow.camera.near = 0.5;
directionalLight.shadow.camera.far = 500;

scene.add(directionalLight);
scene.add(directionalLight.target);
```

### 3. 点光源（PointLight）

点光源从一个点向所有方向发射光线，适合模拟灯泡等光源。

![点光源效果](https://threejs.org/examples/screenshots/webgl_lights_pointlights.jpg)

*图 4.3: 点光源效果示例*

```javascript
// 创建点光源
const pointLight = new THREE.PointLight(
    0xffffff,  // 颜色
    1.0,       // 强度
    100,       // 距离
    2          // 衰减
);

// 设置光源位置
pointLight.position.set(0, 10, 0);

// 启用阴影
pointLight.castShadow = true;

scene.add(pointLight);
```

### 4. 聚光灯（SpotLight）

聚光灯从一个点发射光线，形成一个锥形光束，适合模拟手电筒等光源。

![聚光灯效果](https://threejs.org/examples/screenshots/webgl_lights_spotlight.jpg)

*图 4.4: 聚光灯效果示例*

```javascript
// 创建聚光灯
const spotLight = new THREE.SpotLight(
    0xffffff,  // 颜色
    1.0,       // 强度
    100,       // 距离
    Math.PI / 4, // 角度
    0.5,       // 半影
    2          // 衰减
);

// 设置光源位置
spotLight.position.set(0, 10, 0);

// 设置光源目标
spotLight.target.position.set(0, 0, 0);

// 启用阴影
spotLight.castShadow = true;

scene.add(spotLight);
scene.add(spotLight.target);
```

## 阴影设置

要启用阴影效果，需要同时配置渲染器和对象。

### 1. 渲染器设置

```javascript
// 启用阴影
renderer.shadowMap.enabled = true;

// 设置阴影类型
renderer.shadowMap.type = THREE.PCFSoftShadowMap; // 柔和阴影
```

### 2. 对象设置

```javascript
// 设置对象投射阴影
mesh.castShadow = true;

// 设置对象接收阴影
mesh.receiveShadow = true;
```

### 3. 光源阴影设置

```javascript
// 设置阴影相机参数
light.shadow.camera.near = 0.5;
light.shadow.camera.far = 500;
light.shadow.camera.fov = 45;

// 设置阴影贴图大小
light.shadow.mapSize.width = 1024;
light.shadow.mapSize.height = 1024;

// 设置阴影偏移
light.shadow.bias = -0.0001;
```

## 环境光与反射

### 1. 环境贴图

环境贴图可以模拟周围环境的反射，增强材质的真实感。

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
```

### 2. 反射材质

```javascript
// 创建反射材质
const material = new THREE.MeshStandardMaterial({
    color: 0xffffff,
    metalness: 1.0,
    roughness: 0.0,
    envMap: envMap
});
```

## 光照优化技巧

### 1. 光源优化

```javascript
// 使用适当的光源类型
// 对于大场景，使用平行光
// 对于局部照明，使用点光源或聚光灯

// 控制光源数量
// 避免使用过多光源
// 合并相似的光源

// 调整光源参数
light.intensity = 0.5;  // 降低强度
light.distance = 50;    // 限制范围
```

### 2. 阴影优化

```javascript
// 优化阴影贴图大小
light.shadow.mapSize.width = 512;  // 降低分辨率
light.shadow.mapSize.height = 512;

// 调整阴影相机范围
light.shadow.camera.near = 1;
light.shadow.camera.far = 100;

// 使用阴影偏移
light.shadow.bias = -0.0001;
```

### 3. 性能优化

```javascript
// 禁用不必要的阴影
mesh.castShadow = false;
mesh.receiveShadow = false;

// 使用阴影类型
renderer.shadowMap.type = THREE.BasicShadowMap;  // 基础阴影
// 或
renderer.shadowMap.type = THREE.PCFShadowMap;    // 柔和阴影
```

## 实战：创建一个光照场景

让我们创建一个包含多种光源和阴影的完整场景：

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
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

// 创建环境光
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

// 创建平行光（模拟太阳光）
const directionalLight = new THREE.DirectionalLight(0xffffff, 1.0);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
directionalLight.shadow.mapSize.width = 1024;
directionalLight.shadow.mapSize.height = 1024;
scene.add(directionalLight);

// 创建点光源（模拟灯泡）
const pointLight = new THREE.PointLight(0xff0000, 1.0, 100);
pointLight.position.set(-5, 5, 0);
pointLight.castShadow = true;
scene.add(pointLight);

// 创建地面
const groundGeometry = new THREE.PlaneGeometry(10, 10);
const groundMaterial = new THREE.MeshStandardMaterial({
    color: 0x808080,
    roughness: 0.8,
    metalness: 0.2
});
const ground = new THREE.Mesh(groundGeometry, groundMaterial);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

// 创建物体
const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
const boxMaterial = new THREE.MeshStandardMaterial({
    color: 0x00ff00,
    roughness: 0.7,
    metalness: 0.3
});
const box = new THREE.Mesh(boxGeometry, boxMaterial);
box.position.y = 0.5;
box.castShadow = true;
box.receiveShadow = true;
scene.add(box);

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 旋转物体
    box.rotation.y += 0.01;

    // 移动点光源
    pointLight.position.x = Math.sin(Date.now() * 0.001) * 5;
    pointLight.position.z = Math.cos(Date.now() * 0.001) * 5;

    renderer.render(scene, camera);
}

animate();
```

## 练习

1. 创建一个包含多种光源的场景
2. 实现物体的阴影效果
3. 添加环境贴图
4. 实现光源的动画效果

## 下一步学习

在下一章中，我们将学习：
- 基础动画原理
- 用户交互处理
- 事件系统
- 性能优化