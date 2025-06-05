# 几何体与材质入门

在 Three.js 中，几何体和材质是创建 3D 对象的基础。几何体定义了对象的形状，而材质则决定了对象的外观。让我们深入了解这两个核心概念。

## 基础几何体

Three.js 提供了多种基础几何体，可以满足大多数基本需求。

### 1. 立方体（BoxGeometry）

![立方体示例](https://threejs.org/examples/screenshots/webgl_geometry_cube.jpg)

*图 3.1: 立方体几何体示例*

```javascript
// 创建立方体几何体
const geometry = new THREE.BoxGeometry(
    1,    // 宽度
    1,    // 高度
    1,    // 深度
    1,    // 宽度分段
    1,    // 高度分段
    1     // 深度分段
);

// 创建材质
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });

// 创建网格
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);
```

### 2. 球体（SphereGeometry）

![球体示例](https://threejs.org/examples/screenshots/webgl_geometry_sphere.jpg)

*图 3.2: 球体几何体示例*

```javascript
// 创建球体几何体
const geometry = new THREE.SphereGeometry(
    1,     // 半径
    32,    // 水平分段
    16     // 垂直分段
);

// 创建材质
const material = new THREE.MeshPhongMaterial({
    color: 0x00ff00,
    shininess: 100
});

// 创建网格
const sphere = new THREE.Mesh(geometry, material);
scene.add(sphere);
```

### 3. 圆柱体（CylinderGeometry）

![圆柱体示例](https://threejs.org/examples/screenshots/webgl_geometry_cylinder.jpg)

*图 3.3: 圆柱体几何体示例*

```javascript
// 创建圆柱体几何体
const geometry = new THREE.CylinderGeometry(
    1,     // 顶部半径
    1,     // 底部半径
    2,     // 高度
    32     // 分段
);

// 创建材质
const material = new THREE.MeshStandardMaterial({
    color: 0x00ff00,
    metalness: 0.3,
    roughness: 0.4
});

// 创建网格
const cylinder = new THREE.Mesh(geometry, material);
scene.add(cylinder);
```

### 4. 其他基础几何体

```javascript
// 平面几何体
const planeGeometry = new THREE.PlaneGeometry(2, 2);

// 圆环几何体
const torusGeometry = new THREE.TorusGeometry(1, 0.4, 16, 100);

// 圆锥几何体
const coneGeometry = new THREE.ConeGeometry(1, 2, 32);

// 八面体几何体
const octahedronGeometry = new THREE.OctahedronGeometry(1);
```

## 材质系统

材质决定了对象如何与光线交互，从而影响其外观。

### 1. 基础材质（BasicMaterial）

![基础材质示例](https://threejs.org/examples/screenshots/webgl_materials_basic.jpg)

*图 3.4: 基础材质效果*

```javascript
// 创建基础材质
const material = new THREE.MeshBasicMaterial({
    color: 0x00ff00,        // 颜色
    wireframe: false,       // 线框模式
    side: THREE.FrontSide,  // 渲染面
    transparent: true,      // 透明度
    opacity: 0.5           // 不透明度
});
```

### 2. 标准材质（StandardMaterial）

![标准材质示例](https://threejs.org/examples/screenshots/webgl_materials_standard.jpg)

*图 3.5: 标准材质效果*

```javascript
// 创建标准材质
const material = new THREE.MeshStandardMaterial({
    color: 0x00ff00,        // 颜色
    metalness: 0.5,         // 金属度
    roughness: 0.5,         // 粗糙度
    envMap: envTexture,     // 环境贴图
    normalMap: normalMap,   // 法线贴图
    roughnessMap: roughnessMap, // 粗糙度贴图
    metalnessMap: metalnessMap  // 金属度贴图
});
```

### 3. 物理材质（PhysicalMaterial）

![物理材质示例](https://threejs.org/examples/screenshots/webgl_materials_physical.jpg)

*图 3.6: 物理材质效果*

```javascript
// 创建物理材质
const material = new THREE.MeshPhysicalMaterial({
    color: 0x00ff00,
    metalness: 0.5,
    roughness: 0.5,
    clearcoat: 1.0,         // 清漆层
    clearcoatRoughness: 0.1, // 清漆层粗糙度
    reflectivity: 1.0,      // 反射率
    ior: 1.5,              // 折射率
    transmission: 0.5       // 透射率
});
```

## 纹理基础

纹理可以为材质添加更多细节和真实感。

### 1. 基础纹理

```javascript
// 创建纹理加载器
const textureLoader = new THREE.TextureLoader();

// 加载纹理
const texture = textureLoader.load('texture.jpg');

// 创建材质并应用纹理
const material = new THREE.MeshStandardMaterial({
    map: texture,           // 颜色贴图
    normalMap: normalTexture, // 法线贴图
    roughnessMap: roughnessTexture, // 粗糙度贴图
    metalnessMap: metalnessTexture  // 金属度贴图
});
```

### 2. 纹理设置

```javascript
// 设置纹理参数
texture.wrapS = THREE.RepeatWrapping;  // 水平重复
texture.wrapT = THREE.RepeatWrapping;  // 垂直重复
texture.repeat.set(2, 2);              // 重复次数
texture.offset.set(0.5, 0.5);          // 偏移
texture.rotation = Math.PI / 4;         // 旋转
texture.center.set(0.5, 0.5);          // 旋转中心
```

## 组合与变换

### 1. 对象组合

```javascript
// 创建组
const group = new THREE.Group();

// 添加对象到组
group.add(mesh1);
group.add(mesh2);

// 将组添加到场景
scene.add(group);

// 操作整个组
group.rotation.y = Math.PI / 4;
group.position.set(1, 2, 3);
group.scale.set(2, 2, 2);
```

### 2. 对象变换

```javascript
// 位置
mesh.position.set(x, y, z);
mesh.position.x = 1;
mesh.position.y = 2;
mesh.position.z = 3;

// 旋转
mesh.rotation.set(x, y, z);
mesh.rotation.x = Math.PI / 2;
mesh.rotation.y = Math.PI / 4;
mesh.rotation.z = Math.PI / 6;

// 缩放
mesh.scale.set(x, y, z);
mesh.scale.x = 2;
mesh.scale.y = 2;
mesh.scale.z = 2;
```

## 实战：创建一个组合对象

让我们创建一个由多个几何体组成的复杂对象：

```javascript
// 创建组
const group = new THREE.Group();

// 创建主体（球体）
const bodyGeometry = new THREE.SphereGeometry(1, 32, 32);
const bodyMaterial = new THREE.MeshStandardMaterial({
    color: 0x00ff00,
    metalness: 0.3,
    roughness: 0.4
});
const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
group.add(body);

// 创建眼睛（小球体）
const eyeGeometry = new THREE.SphereGeometry(0.2, 16, 16);
const eyeMaterial = new THREE.MeshStandardMaterial({
    color: 0x000000,
    metalness: 0.5,
    roughness: 0.2
});

// 左眼
const leftEye = new THREE.Mesh(eyeGeometry, eyeMaterial);
leftEye.position.set(-0.3, 0.3, 0.8);
group.add(leftEye);

// 右眼
const rightEye = new THREE.Mesh(eyeGeometry, eyeMaterial);
rightEye.position.set(0.3, 0.3, 0.8);
group.add(rightEye);

// 创建嘴巴（圆环）
const mouthGeometry = new THREE.TorusGeometry(0.3, 0.1, 16, 32);
const mouthMaterial = new THREE.MeshStandardMaterial({
    color: 0xff0000,
    metalness: 0.3,
    roughness: 0.4
});
const mouth = new THREE.Mesh(mouthGeometry, mouthMaterial);
mouth.rotation.x = Math.PI / 2;
mouth.position.set(0, -0.2, 0.8);
group.add(mouth);

// 将组添加到场景
scene.add(group);

// 动画
function animate() {
    requestAnimationFrame(animate);

    // 旋转整个组
    group.rotation.y += 0.01;

    // 渲染场景
    renderer.render(scene, camera);
}

animate();
```

## 性能优化建议

1. **几何体优化**
   - 使用适当的几何体复杂度
   - 合并相似的几何体
   - 使用 LOD（细节层次）

2. **材质优化**
   - 重用材质实例
   - 使用适当的材质类型
   - 优化纹理大小

3. **纹理优化**
   - 使用适当的纹理格式
   - 压缩纹理
   - 使用纹理图集

## 练习

1. 创建一个由多个几何体组成的角色
2. 为对象添加不同的材质和纹理
3. 实现对象的组合和变换
4. 添加简单的动画效果

## 下一步学习

在下一章中，我们将学习：
- 光源类型和设置
- 阴影效果
- 环境光和反射
- 光照优化技巧