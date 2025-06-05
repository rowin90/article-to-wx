# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 性能监控与调试

在 Three.js 应用中，性能监控和调试是确保应用流畅运行的关键。本章将介绍如何使用各种工具和技术来监控和优化 Three.js 应用的性能。

## 性能监控工具

### 1. Stats.js

![性能监控示例](https://threejs.org/examples/screenshots/webgl_geometry_teapot.jpg)

*图 15.1: 性能监控示例*

```javascript
// 性能监控管理器
class PerformanceMonitor {
    constructor() {
        this.stats = new Stats();
        this.stats.showPanel(0); // 0: fps, 1: ms, 2: mb, 3+: custom
        document.body.appendChild(this.stats.dom);

        this.metrics = {
            fps: 0,
            frameTime: 0,
            drawCalls: 0,
            triangles: 0,
            geometries: 0,
            textures: 0
        };
    }

    begin() {
        this.stats.begin();
    }

    end() {
        this.stats.end();
    }

    update(renderer) {
        // 更新性能指标
        this.metrics.fps = 1000 / this.stats.domElement.children[0].children[0].textContent;
        this.metrics.frameTime = this.stats.domElement.children[1].children[0].textContent;

        // 更新渲染器统计信息
        const info = renderer.info;
        this.metrics.drawCalls = info.render.calls;
        this.metrics.triangles = info.render.triangles;
        this.metrics.geometries = info.memory.geometries;
        this.metrics.textures = info.memory.textures;

        return this.metrics;
    }
}
```

### 2. 性能分析器

```javascript
// 性能分析器
class PerformanceAnalyzer {
    constructor() {
        this.metrics = new Map();
        this.samples = new Map();
        this.maxSamples = 60; // 保存最近60帧的数据
    }

    startMeasure(name) {
        if (!this.metrics.has(name)) {
            this.metrics.set(name, {
                total: 0,
                count: 0,
                min: Infinity,
                max: -Infinity
            });
        }

        const startTime = performance.now();
        return () => this.endMeasure(name, startTime);
    }

    endMeasure(name, startTime) {
        const duration = performance.now() - startTime;
        const metric = this.metrics.get(name);

        metric.total += duration;
        metric.count++;
        metric.min = Math.min(metric.min, duration);
        metric.max = Math.max(metric.max, duration);

        // 更新样本
        if (!this.samples.has(name)) {
            this.samples.set(name, []);
        }

        const samples = this.samples.get(name);
        samples.push(duration);

        if (samples.length > this.maxSamples) {
            samples.shift();
        }
    }

    getMetrics() {
        const result = {};

        this.metrics.forEach((metric, name) => {
            result[name] = {
                average: metric.total / metric.count,
                min: metric.min,
                max: metric.max,
                samples: this.samples.get(name)
            };
        });

        return result;
    }

    reset() {
        this.metrics.clear();
        this.samples.clear();
    }
}
```

## 内存管理

### 1. 资源管理器

```javascript
// 资源管理器
class ResourceManager {
    constructor() {
        this.geometries = new Map();
        this.materials = new Map();
        this.textures = new Map();
        this.meshes = new Map();
    }

    createGeometry(key, geometry) {
        this.geometries.set(key, geometry);
        return geometry;
    }

    createMaterial(key, material) {
        this.materials.set(key, material);
        return material;
    }

    createTexture(key, texture) {
        this.textures.set(key, texture);
        return texture;
    }

    createMesh(key, mesh) {
        this.meshes.set(key, mesh);
        return mesh;
    }

    getGeometry(key) {
        return this.geometries.get(key);
    }

    getMaterial(key) {
        return this.materials.get(key);
    }

    getTexture(key) {
        return this.textures.get(key);
    }

    getMesh(key) {
        return this.meshes.get(key);
    }

    disposeGeometry(key) {
        const geometry = this.geometries.get(key);
        if (geometry) {
            geometry.dispose();
            this.geometries.delete(key);
        }
    }

    disposeMaterial(key) {
        const material = this.materials.get(key);
        if (material) {
            material.dispose();
            this.materials.delete(key);
        }
    }

    disposeTexture(key) {
        const texture = this.textures.get(key);
        if (texture) {
            texture.dispose();
            this.textures.delete(key);
        }
    }

    disposeMesh(key) {
        const mesh = this.meshes.get(key);
        if (mesh) {
            mesh.geometry.dispose();
            mesh.material.dispose();
            this.meshes.delete(key);
        }
    }

    disposeAll() {
        this.geometries.forEach(geometry => geometry.dispose());
        this.materials.forEach(material => material.dispose());
        this.textures.forEach(texture => texture.dispose());
        this.meshes.forEach(mesh => {
            mesh.geometry.dispose();
            mesh.material.dispose();
        });

        this.geometries.clear();
        this.materials.clear();
        this.textures.clear();
        this.meshes.clear();
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
        let obj;

        if (this.pool.length > 0) {
            obj = this.pool.pop();
        } else {
            obj = this.createFn();
        }

        this.active.add(obj);
        return obj;
    }

    release(obj) {
        if (this.active.has(obj)) {
            this.resetFn(obj);
            this.active.delete(obj);
            this.pool.push(obj);
        }
    }

    releaseAll() {
        this.active.forEach(obj => {
            this.resetFn(obj);
            this.pool.push(obj);
        });
        this.active.clear();
    }

    getActiveCount() {
        return this.active.size;
    }

    getPoolSize() {
        return this.pool.length;
    }
}
```

## 渲染优化

### 1. 渲染器优化

```javascript
// 渲染器优化管理器
class RendererOptimizer {
    constructor(renderer) {
        this.renderer = renderer;
        this.setupOptimizations();
    }

    setupOptimizations() {
        // 设置渲染器优化选项
        this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        this.renderer.powerPreference = 'high-performance';
        this.renderer.shadowMap.enabled = true;
        this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;

        // 设置自动清理
        this.renderer.autoClear = true;
        this.renderer.autoClearColor = true;
        this.renderer.autoClearDepth = true;
        this.renderer.autoClearStencil = true;
    }

    optimizeForPerformance() {
        // 降低阴影质量
        this.renderer.shadowMap.type = THREE.BasicShadowMap;

        // 禁用抗锯齿
        this.renderer.setPixelRatio(1);
    }

    optimizeForQuality() {
        // 提高阴影质量
        this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;

        // 启用抗锯齿
        this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    }

    updateSize(width, height) {
        this.renderer.setSize(width, height);
    }
}
```

### 2. 场景优化

```javascript
// 场景优化管理器
class SceneOptimizer {
    constructor(scene) {
        this.scene = scene;
        this.optimizations = new Map();
    }

    optimizeGeometry(geometry) {
        // 合并顶点
        geometry.mergeVertices();

        // 计算法线
        geometry.computeVertexNormals();

        // 计算切线
        geometry.computeTangents();

        return geometry;
    }

    optimizeMaterial(material) {
        // 设置材质优化选项
        material.precision = 'mediump';
        material.flatShading = true;
        material.vertexColors = false;

        return material;
    }

    optimizeMesh(mesh) {
        // 设置网格优化选项
        mesh.frustumCulled = true;
        mesh.castShadow = true;
        mesh.receiveShadow = true;

        return mesh;
    }

    optimizeScene() {
        this.scene.traverse(object => {
            if (object instanceof THREE.Mesh) {
                this.optimizeMesh(object);

                if (object.geometry) {
                    this.optimizeGeometry(object.geometry);
                }

                if (object.material) {
                    this.optimizeMaterial(object.material);
                }
            }
        });
    }
}
```

## 实战：性能监控系统

让我们创建一个完整的性能监控系统：

```javascript
// 创建场景
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.z = 5;

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 创建性能监控系统
const performanceMonitor = new PerformanceMonitor();
const performanceAnalyzer = new PerformanceAnalyzer();
const resourceManager = new ResourceManager();
const rendererOptimizer = new RendererOptimizer(renderer);
const sceneOptimizer = new SceneOptimizer(scene);

// 创建对象池
const particlePool = new ObjectPool(
    () => new THREE.Mesh(
        new THREE.SphereGeometry(0.1),
        new THREE.MeshBasicMaterial({ color: 0xff0000 })
    ),
    (particle) => {
        particle.position.set(0, 0, 0);
        particle.visible = false;
    }
);

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 开始性能监控
    performanceMonitor.begin();
    const endMeasure = performanceAnalyzer.startMeasure('frame');

    // 更新场景
    scene.traverse(object => {
        if (object instanceof THREE.Mesh) {
            object.rotation.x += 0.01;
            object.rotation.y += 0.01;
        }
    });

    // 渲染场景
    renderer.render(scene, camera);

    // 结束性能监控
    endMeasure();
    performanceMonitor.end();

    // 更新性能指标
    const metrics = performanceMonitor.update(renderer);
    console.log('Performance Metrics:', metrics);
}

animate();

// 窗口大小改变时更新
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    rendererOptimizer.updateSize(window.innerWidth, window.innerHeight);
});

// 性能监控面板
const panel = document.createElement('div');
panel.style.position = 'fixed';
panel.style.top = '0';
panel.style.right = '0';
panel.style.background = 'rgba(0, 0, 0, 0.7)';
panel.style.color = 'white';
panel.style.padding = '10px';
panel.style.fontFamily = 'monospace';
document.body.appendChild(panel);

// 更新性能面板
function updatePanel() {
    const metrics = performanceMonitor.update(renderer);
    panel.innerHTML = `
        FPS: ${metrics.fps.toFixed(1)}<br>
        Frame Time: ${metrics.frameTime}ms<br>
        Draw Calls: ${metrics.drawCalls}<br>
        Triangles: ${metrics.triangles}<br>
        Geometries: ${metrics.geometries}<br>
        Textures: ${metrics.textures}
    `;
}

setInterval(updatePanel, 1000);
```

## 练习

1. 实现基础性能监控
2. 添加内存管理
3. 优化渲染性能
4. 创建性能监控面板

## 下一步学习

在下一章中，我们将学习：
- Shader 编程入门
- 自定义着色器
- 后处理效果
- 高级渲染技术

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！
