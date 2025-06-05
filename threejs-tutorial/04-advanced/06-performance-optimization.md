# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 性能优化

在 Three.js 中，性能优化是开发高质量 3D 应用的关键。本章将介绍各种性能优化技术和最佳实践。

## 渲染优化

### 1. 渲染器设置

![性能优化示例](https://threejs.org/examples/screenshots/webgl_geometry_instancing.jpg)

*图 21.1: 性能优化示例*

```javascript
// 渲染器优化器
class RendererOptimizer {
    constructor(renderer) {
        this.renderer = renderer;
        this.optimize();
    }

    optimize() {
        // 禁用抗锯齿
        this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

        // 设置电源偏好
        this.renderer.setPowerPreference('high-performance');

        // 使用中等精度
        this.renderer.setPrecision('mediump');

        // 优化阴影
        this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        this.renderer.shadowMap.enabled = true;

        // 设置自动清除
        this.renderer.autoClear = true;
        this.renderer.autoClearColor = true;
        this.renderer.autoClearDepth = true;
        this.renderer.autoClearStencil = true;
    }

    setSize(width, height) {
        this.renderer.setSize(width, height, false);
    }

    dispose() {
        this.renderer.dispose();
    }
}
```

### 2. 相机优化

```javascript
// 相机优化器
class CameraOptimizer {
    constructor(camera) {
        this.camera = camera;
        this.optimize();
    }

    optimize() {
        // 设置合适的近平面和远平面
        this.camera.near = 0.1;
        this.camera.far = 1000;

        // 更新投影矩阵
        this.camera.updateProjectionMatrix();

        // 设置视锥体剔除
        this.camera.frustumCulled = true;
    }

    setOrthographic(width, height) {
        const aspect = width / height;
        const frustumSize = 10;

        this.camera = new THREE.OrthographicCamera(
            frustumSize * aspect / -2,
            frustumSize * aspect / 2,
            frustumSize / 2,
            frustumSize / -2,
            0.1,
            1000
        );
    }
}
```

## 几何体优化

### 1. 几何体合并

```javascript
// 几何体合并器
class GeometryMerger {
    constructor() {
        this.geometries = [];
    }

    add(geometry) {
        this.geometries.push(geometry);
    }

    merge() {
        if (this.geometries.length === 0) return null;

        // 合并几何体
        const mergedGeometry = BufferGeometryUtils.mergeBufferGeometries(
            this.geometries
        );

        // 清理原始几何体
        this.geometries.forEach(geometry => geometry.dispose());
        this.geometries = [];

        return mergedGeometry;
    }
}
```

### 2. 实例化渲染

```javascript
// 实例化渲染器
class InstancedRenderer {
    constructor(geometry, material, count) {
        this.geometry = geometry;
        this.material = material;
        this.count = count;
        this.mesh = null;
        this.matrix = new THREE.Matrix4();
        this.init();
    }

    init() {
        this.mesh = new THREE.InstancedMesh(
            this.geometry,
            this.material,
            this.count
        );
    }

    setMatrix(index, position, rotation, scale) {
        this.matrix.compose(
            position,
            rotation,
            scale
        );
        this.mesh.setMatrixAt(index, this.matrix);
    }

    update() {
        this.mesh.instanceMatrix.needsUpdate = true;
    }
}
```

## 材质优化

### 1. 材质共享

```javascript
// 材质管理器
class MaterialManager {
    constructor() {
        this.materials = new Map();
    }

    createMaterial(options) {
        const key = JSON.stringify(options);
        if (this.materials.has(key)) {
            return this.materials.get(key);
        }

        const material = new THREE.MeshStandardMaterial(options);
        this.materials.set(key, material);
        return material;
    }

    dispose() {
        this.materials.forEach(material => material.dispose());
        this.materials.clear();
    }
}
```

### 2. 纹理优化

```javascript
// 纹理优化器
class TextureOptimizer {
    constructor() {
        this.textures = new Map();
    }

    loadTexture(url, options = {}) {
        if (this.textures.has(url)) {
            return this.textures.get(url);
        }

        const texture = new THREE.TextureLoader().load(url, texture => {
            // 设置纹理参数
            texture.minFilter = THREE.LinearFilter;
            texture.magFilter = THREE.LinearFilter;
            texture.generateMipmaps = false;

            // 应用自定义选项
            Object.assign(texture, options);
        });

        this.textures.set(url, texture);
        return texture;
    }

    dispose() {
        this.textures.forEach(texture => texture.dispose());
        this.textures.clear();
    }
}
```

## 内存管理

### 1. 资源管理

```javascript
// 资源管理器
class ResourceManager {
    constructor() {
        this.geometries = new Map();
        this.materials = new Map();
        this.textures = new Map();
    }

    addGeometry(name, geometry) {
        this.geometries.set(name, geometry);
    }

    addMaterial(name, material) {
        this.materials.set(name, material);
    }

    addTexture(name, texture) {
        this.textures.set(name, texture);
    }

    dispose() {
        // 释放几何体
        this.geometries.forEach(geometry => {
            geometry.dispose();
        });
        this.geometries.clear();

        // 释放材质
        this.materials.forEach(material => {
            material.dispose();
        });
        this.materials.clear();

        // 释放纹理
        this.textures.forEach(texture => {
            texture.dispose();
        });
        this.textures.clear();
    }
}
```

### 2. 对象池

```javascript
// 对象池
class ObjectPool {
    constructor(createFn, resetFn, initialSize = 10) {
        this.createFn = createFn;
        this.resetFn = resetFn;
        this.pool = [];
        this.active = new Set();

        // 初始化对象池
        for (let i = 0; i < initialSize; i++) {
            this.pool.push(this.createFn());
        }
    }

    get() {
        let obj = this.pool.pop();
        if (!obj) {
            obj = this.createFn();
        }
        this.active.add(obj);
        return obj;
    }

    release(obj) {
        if (this.active.has(obj)) {
            this.resetFn(obj);
            this.pool.push(obj);
            this.active.delete(obj);
        }
    }

    releaseAll() {
        this.active.forEach(obj => this.release(obj));
    }
}
```

## 性能监控

### 1. 性能统计

```javascript
// 性能统计器
class PerformanceStats {
    constructor() {
        this.stats = new Stats();
        this.stats.showPanel(0);
        document.body.appendChild(this.stats.dom);
    }

    begin() {
        this.stats.begin();
    }

    end() {
        this.stats.end();
    }
}
```

### 2. 性能分析

```javascript
// 性能分析器
class PerformanceAnalyzer {
    constructor() {
        this.metrics = {
            fps: [],
            frameTime: [],
            drawCalls: [],
            triangles: []
        };
        this.maxSamples = 100;
    }

    addMetric(name, value) {
        if (!this.metrics[name]) {
            this.metrics[name] = [];
        }

        this.metrics[name].push(value);
        if (this.metrics[name].length > this.maxSamples) {
            this.metrics[name].shift();
        }
    }

    getAverage(name) {
        const values = this.metrics[name];
        if (!values || values.length === 0) return 0;

        const sum = values.reduce((a, b) => a + b, 0);
        return sum / values.length;
    }

    getMetrics() {
        return {
            fps: this.getAverage('fps'),
            frameTime: this.getAverage('frameTime'),
            drawCalls: this.getAverage('drawCalls'),
            triangles: this.getAverage('triangles')
        };
    }
}
```

## 实战：性能优化应用

让我们创建一个完整的性能优化示例：

```javascript
// 创建场景
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.set(0, 5, 10);
camera.lookAt(0, 0, 0);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 优化渲染器
const rendererOptimizer = new RendererOptimizer(renderer);

// 优化相机
const cameraOptimizer = new CameraOptimizer(camera);

// 创建资源管理器
const resourceManager = new ResourceManager();

// 创建对象池
const cubePool = new ObjectPool(
    () => new THREE.Mesh(
        new THREE.BoxGeometry(1, 1, 1),
        new THREE.MeshStandardMaterial({ color: 0x00ff00 })
    ),
    (cube) => {
        cube.position.set(0, 0, 0);
        cube.rotation.set(0, 0, 0);
        cube.scale.set(1, 1, 1);
    }
);

// 创建性能监控
const performanceStats = new PerformanceStats();
const performanceAnalyzer = new PerformanceAnalyzer();

// 添加光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
scene.add(directionalLight);

// 创建地面
const groundGeometry = new THREE.PlaneGeometry(10, 10);
const groundMaterial = resourceManager.createMaterial({ color: 0x808080 });
const ground = new THREE.Mesh(groundGeometry, groundMaterial);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 性能统计开始
    performanceStats.begin();

    // 更新对象池中的物体
    cubePool.active.forEach(cube => {
        cube.rotation.x += 0.01;
        cube.rotation.y += 0.01;
    });

    // 渲染场景
    renderer.render(scene, camera);

    // 更新性能指标
    performanceAnalyzer.addMetric('fps', 1000 / (performance.now() - lastTime));
    performanceAnalyzer.addMetric('drawCalls', renderer.info.render.calls);
    performanceAnalyzer.addMetric('triangles', renderer.info.render.triangles);

    // 性能统计结束
    performanceStats.end();

    lastTime = performance.now();
}

let lastTime = performance.now();
animate();

// 窗口大小改变时更新
window.addEventListener('resize', () => {
    const width = window.innerWidth;
    const height = window.innerHeight;

    camera.aspect = width / height;
    camera.updateProjectionMatrix();

    rendererOptimizer.setSize(width, height);
});

// 页面卸载时清理资源
window.addEventListener('unload', () => {
    resourceManager.dispose();
    rendererOptimizer.dispose();
    cubePool.releaseAll();
});
```

## 练习

1. 实现渲染优化
2. 优化几何体
3. 管理内存资源
4. 监控性能指标

## 下一步学习

在下一章中，我们将学习：
- 最佳实践
- 项目部署
- 发布上线
- 维护更新

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！