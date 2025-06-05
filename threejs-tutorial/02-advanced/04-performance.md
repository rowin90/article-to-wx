# 性能优化技巧

在 Three.js 开发中，性能优化是一个重要的课题。本章将介绍各种性能优化技巧，帮助你创建流畅的 3D 应用。

## 渲染优化

### 1. 渲染器设置

![渲染性能对比](https://threejs.org/examples/screenshots/webgl_performance.jpg)

*图 9.1: 渲染性能对比*

```javascript
// 优化渲染器设置
const renderer = new THREE.WebGLRenderer({
    antialias: false,  // 关闭抗锯齿
    powerPreference: 'high-performance',  // 优先使用高性能模式
    precision: 'mediump'  // 使用中等精度
});

// 设置像素比
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

// 启用阴影
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;  // 使用 PCF 软阴影
```

### 2. 相机优化

```javascript
// 优化相机设置
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);

// 设置相机视锥体
camera.far = 1000;  // 减小远平面距离
camera.near = 0.1;  // 增大近平面距离

// 使用正交相机（适用于 2D 场景）
const orthoCamera = new THREE.OrthographicCamera(
    -width / 2,
    width / 2,
    height / 2,
    -height / 2,
    0.1,
    1000
);
```

## 几何体优化

### 1. 几何体合并

```javascript
// 合并多个几何体
function mergeGeometries(geometries) {
    const mergedGeometry = new THREE.BufferGeometry();
    const positions = [];
    const normals = [];
    const uvs = [];

    // 合并顶点数据
    geometries.forEach(geometry => {
        const positionAttribute = geometry.getAttribute('position');
        const normalAttribute = geometry.getAttribute('normal');
        const uvAttribute = geometry.getAttribute('uv');

        for (let i = 0; i < positionAttribute.count; i++) {
            positions.push(
                positionAttribute.getX(i),
                positionAttribute.getY(i),
                positionAttribute.getZ(i)
            );
            normals.push(
                normalAttribute.getX(i),
                normalAttribute.getY(i),
                normalAttribute.getZ(i)
            );
            uvs.push(
                uvAttribute.getX(i),
                uvAttribute.getY(i)
            );
        }
    });

    // 设置合并后的属性
    mergedGeometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
    mergedGeometry.setAttribute('normal', new THREE.Float32BufferAttribute(normals, 3));
    mergedGeometry.setAttribute('uv', new THREE.Float32BufferAttribute(uvs, 2));

    return mergedGeometry;
}
```

### 2. 实例化渲染

```javascript
// 使用实例化渲染
class InstancedMesh {
    constructor(geometry, material, count) {
        this.geometry = geometry;
        this.material = material;
        this.count = count;
        this.mesh = null;
        this.init();
    }

    init() {
        // 创建实例化网格
        this.mesh = new THREE.InstancedMesh(
            this.geometry,
            this.material,
            this.count
        );

        // 设置实例化矩阵
        const matrix = new THREE.Matrix4();
        for (let i = 0; i < this.count; i++) {
            matrix.setPosition(
                Math.random() * 10 - 5,
                Math.random() * 10 - 5,
                Math.random() * 10 - 5
            );
            this.mesh.setMatrixAt(i, matrix);
        }
    }

    update() {
        // 更新实例化矩阵
        const matrix = new THREE.Matrix4();
        for (let i = 0; i < this.count; i++) {
            this.mesh.getMatrixAt(i, matrix);
            matrix.multiply(
                new THREE.Matrix4().makeRotationY(0.01)
            );
            this.mesh.setMatrixAt(i, matrix);
        }
        this.mesh.instanceMatrix.needsUpdate = true;
    }
}
```

## 材质优化

### 1. 材质共享

```javascript
// 共享材质
class MaterialManager {
    constructor() {
        this.materials = new Map();
    }

    getMaterial(type, params) {
        const key = this.getMaterialKey(type, params);
        if (!this.materials.has(key)) {
            this.materials.set(key, this.createMaterial(type, params));
        }
        return this.materials.get(key);
    }

    getMaterialKey(type, params) {
        return `${type}-${JSON.stringify(params)}`;
    }

    createMaterial(type, params) {
        switch (type) {
            case 'standard':
                return new THREE.MeshStandardMaterial(params);
            case 'basic':
                return new THREE.MeshBasicMaterial(params);
            default:
                return new THREE.MeshStandardMaterial(params);
        }
    }
}
```

### 2. 纹理优化

```javascript
// 纹理优化
class TextureManager {
    constructor() {
        this.textures = new Map();
        this.loader = new THREE.TextureLoader();
    }

    loadTexture(url, options = {}) {
        if (this.textures.has(url)) {
            return this.textures.get(url);
        }

        const texture = this.loader.load(url, undefined, undefined, undefined);

        // 设置纹理参数
        texture.minFilter = THREE.LinearFilter;
        texture.magFilter = THREE.LinearFilter;
        texture.generateMipmaps = false;

        // 应用自定义选项
        Object.assign(texture, options);

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

### 1. 资源释放

```javascript
// 资源管理器
class ResourceManager {
    constructor() {
        this.geometries = new Set();
        this.materials = new Set();
        this.textures = new Set();
    }

    addGeometry(geometry) {
        this.geometries.add(geometry);
    }

    addMaterial(material) {
        this.materials.add(material);
        if (material.map) this.addTexture(material.map);
        if (material.normalMap) this.addTexture(material.normalMap);
        if (material.roughnessMap) this.addTexture(material.roughnessMap);
        if (material.metalnessMap) this.addTexture(material.metalnessMap);
    }

    addTexture(texture) {
        this.textures.add(texture);
    }

    dispose() {
        // 释放几何体
        this.geometries.forEach(geometry => {
            geometry.dispose();
        });

        // 释放材质
        this.materials.forEach(material => {
            material.dispose();
        });

        // 释放纹理
        this.textures.forEach(texture => {
            texture.dispose();
        });

        // 清空集合
        this.geometries.clear();
        this.materials.clear();
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
// 性能监控
class PerformanceMonitor {
    constructor() {
        this.stats = new Stats();
        this.stats.showPanel(0);
        document.body.appendChild(this.stats.dom);

        this.fps = 0;
        this.frameCount = 0;
        this.lastTime = performance.now();
    }

    update() {
        this.stats.begin();

        // 计算 FPS
        const now = performance.now();
        this.frameCount++;

        if (now - this.lastTime >= 1000) {
            this.fps = this.frameCount;
            this.frameCount = 0;
            this.lastTime = now;
        }

        this.stats.end();
    }

    getFPS() {
        return this.fps;
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
            drawCalls: [],
            triangles: [],
            geometries: [],
            textures: []
        };
    }

    collectMetrics(renderer) {
        const info = renderer.info;

        this.metrics.fps.push(performance.now());
        this.metrics.drawCalls.push(info.render.calls);
        this.metrics.triangles.push(info.render.triangles);
        this.metrics.geometries.push(info.memory.geometries);
        this.metrics.textures.push(info.memory.textures);
    }

    analyze() {
        return {
            averageFPS: this.calculateAverageFPS(),
            averageDrawCalls: this.calculateAverage(this.metrics.drawCalls),
            averageTriangles: this.calculateAverage(this.metrics.triangles),
            peakMemory: this.calculatePeakMemory()
        };
    }

    calculateAverageFPS() {
        const times = this.metrics.fps;
        if (times.length < 2) return 0;

        const intervals = [];
        for (let i = 1; i < times.length; i++) {
            intervals.push(times[i] - times[i - 1]);
        }

        return 1000 / (intervals.reduce((a, b) => a + b) / intervals.length);
    }

    calculateAverage(array) {
        return array.reduce((a, b) => a + b) / array.length;
    }

    calculatePeakMemory() {
        return {
            geometries: Math.max(...this.metrics.geometries),
            textures: Math.max(...this.metrics.textures)
        };
    }
}
```

## 实战：性能优化示例

让我们创建一个展示各种性能优化技巧的场景：

```javascript
// 创建场景
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x000000);

// 创建相机
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.set(0, 0, 10);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({
    antialias: false,
    powerPreference: 'high-performance',
    precision: 'mediump'
});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
document.body.appendChild(renderer.domElement);

// 创建资源管理器
const resourceManager = new ResourceManager();
const materialManager = new MaterialManager();
const textureManager = new TextureManager();

// 创建实例化网格
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = materialManager.getMaterial('standard', {
    color: 0x00ff00,
    metalness: 0.5,
    roughness: 0.5
});
const instancedMesh = new InstancedMesh(geometry, material, 1000);
scene.add(instancedMesh.mesh);

// 创建性能监控
const performanceMonitor = new PerformanceMonitor();
const performanceAnalyzer = new PerformanceAnalyzer();

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新实例化网格
    instancedMesh.update();

    // 更新性能监控
    performanceMonitor.update();
    performanceAnalyzer.collectMetrics(renderer);

    // 渲染场景
    renderer.render(scene, camera);
}

// 窗口大小改变时更新
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});

animate();
```

## 练习

1. 实现几何体合并
2. 使用实例化渲染
3. 优化材质和纹理
4. 实现性能监控

## 下一步学习

在下一章中，我们将学习：
- 模型加载与动画
- 骨骼动画
- 动画混合
- 动画控制
