# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 跨平台适配

在 Three.js 中，我们需要确保应用在不同设备和平台上都能正常运行。本章将介绍如何进行跨平台适配。

## 设备检测

### 1. 设备能力检测

![跨平台适配示例](https://threejs.org/examples/screenshots/webgl_loader_gltf.jpg)

*图 20.1: 跨平台适配示例*

```javascript
// 设备能力检测器
class DeviceCapabilityDetector {
    constructor() {
        this.capabilities = {
            webgl: this.checkWebGL(),
            webgl2: this.checkWebGL2(),
            touch: this.checkTouch(),
            mobile: this.checkMobile(),
            highDPI: this.checkHighDPI()
        };
    }

    checkWebGL() {
        try {
            const canvas = document.createElement('canvas');
            return !!(window.WebGLRenderingContext &&
                (canvas.getContext('webgl') || canvas.getContext('experimental-webgl')));
        } catch (e) {
            return false;
        }
    }

    checkWebGL2() {
        try {
            const canvas = document.createElement('canvas');
            return !!(window.WebGL2RenderingContext && canvas.getContext('webgl2'));
        } catch (e) {
            return false;
        }
    }

    checkTouch() {
        return 'ontouchstart' in window || navigator.maxTouchPoints > 0;
    }

    checkMobile() {
        return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
    }

    checkHighDPI() {
        return window.devicePixelRatio > 1;
    }

    getCapabilities() {
        return this.capabilities;
    }
}
```

### 2. 性能检测

```javascript
// 性能检测器
class PerformanceDetector {
    constructor() {
        this.metrics = {
            fps: 0,
            frameTime: 0,
            memory: 0
        };
        this.frameCount = 0;
        this.lastTime = performance.now();
    }

    start() {
        this.update();
    }

    update() {
        const currentTime = performance.now();
        const deltaTime = currentTime - this.lastTime;

        this.frameCount++;

        if (deltaTime >= 1000) {
            this.metrics.fps = Math.round((this.frameCount * 1000) / deltaTime);
            this.metrics.frameTime = deltaTime / this.frameCount;

            if (performance.memory) {
                this.metrics.memory = performance.memory.usedJSHeapSize / 1048576;
            }

            this.frameCount = 0;
            this.lastTime = currentTime;
        }

        requestAnimationFrame(() => this.update());
    }

    getMetrics() {
        return this.metrics;
    }
}
```

## 响应式设计

### 1. 屏幕适配

```javascript
// 屏幕适配器
class ScreenAdapter {
    constructor(renderer, camera) {
        this.renderer = renderer;
        this.camera = camera;
        this.aspectRatio = window.innerWidth / window.innerHeight;
        this.updateSize();

        window.addEventListener('resize', () => this.updateSize());
    }

    updateSize() {
        const width = window.innerWidth;
        const height = window.innerHeight;

        // 更新渲染器
        this.renderer.setSize(width, height);

        // 更新相机
        if (this.camera.isPerspectiveCamera) {
            this.camera.aspect = width / height;
            this.camera.updateProjectionMatrix();
        }
    }

    getDevicePixelRatio() {
        return Math.min(window.devicePixelRatio, 2);
    }

    setPixelRatio() {
        this.renderer.setPixelRatio(this.getDevicePixelRatio());
    }
}
```

### 2. 触摸适配

```javascript
// 触摸适配器
class TouchAdapter {
    constructor(camera, domElement) {
        this.camera = camera;
        this.domElement = domElement;
        this.enabled = false;
        this.target = new THREE.Vector3();
        this.initialTouch = new THREE.Vector2();
        this.currentTouch = new THREE.Vector2();

        this.setupEventListeners();
    }

    setupEventListeners() {
        this.domElement.addEventListener('touchstart', this.onTouchStart.bind(this));
        this.domElement.addEventListener('touchmove', this.onTouchMove.bind(this));
        this.domElement.addEventListener('touchend', this.onTouchEnd.bind(this));
    }

    onTouchStart(event) {
        if (!this.enabled) return;

        const touch = event.touches[0];
        this.initialTouch.set(touch.clientX, touch.clientY);
        this.currentTouch.copy(this.initialTouch);
    }

    onTouchMove(event) {
        if (!this.enabled) return;

        const touch = event.touches[0];
        this.currentTouch.set(touch.clientX, touch.clientY);

        const deltaX = this.currentTouch.x - this.initialTouch.x;
        const deltaY = this.currentTouch.y - this.initialTouch.y;

        this.rotateCamera(deltaX, deltaY);
    }

    onTouchEnd() {
        if (!this.enabled) return;
        // 处理触摸结束事件
    }

    rotateCamera(deltaX, deltaY) {
        const rotationSpeed = 0.005;
        this.camera.rotation.y -= deltaX * rotationSpeed;
        this.camera.rotation.x -= deltaY * rotationSpeed;

        // 限制垂直旋转角度
        this.camera.rotation.x = Math.max(
            -Math.PI / 2,
            Math.min(Math.PI / 2, this.camera.rotation.x)
        );
    }

    enable() {
        this.enabled = true;
    }

    disable() {
        this.enabled = false;
    }
}
```

## 性能优化

### 1. 资源管理

```javascript
// 资源管理器
class ResourceManager {
    constructor() {
        this.textures = new Map();
        this.geometries = new Map();
        this.materials = new Map();
        this.loadingManager = new THREE.LoadingManager();
    }

    loadTexture(url) {
        if (this.textures.has(url)) {
            return this.textures.get(url);
        }

        const texture = new THREE.TextureLoader(this.loadingManager).load(url);
        this.textures.set(url, texture);
        return texture;
    }

    loadGeometry(url) {
        if (this.geometries.has(url)) {
            return this.geometries.get(url);
        }

        const geometry = new THREE.BufferGeometry();
        // 加载几何体
        this.geometries.set(url, geometry);
        return geometry;
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
        this.textures.forEach(texture => texture.dispose());
        this.geometries.forEach(geometry => geometry.dispose());
        this.materials.forEach(material => material.dispose());

        this.textures.clear();
        this.geometries.clear();
        this.materials.clear();
    }
}
```

### 2. 性能监控

```javascript
// 性能监控器
class PerformanceMonitor {
    constructor() {
        this.stats = new Stats();
        this.stats.showPanel(0);
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

    updateMetrics(renderer) {
        const info = renderer.info;

        this.metrics.drawCalls = info.render.calls;
        this.metrics.triangles = info.render.triangles;
        this.metrics.geometries = info.memory.geometries;
        this.metrics.textures = info.memory.textures;
    }

    getMetrics() {
        return this.metrics;
    }
}
```

## 实战：跨平台应用

让我们创建一个完整的跨平台应用示例：

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
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

// 设备检测
const deviceDetector = new DeviceCapabilityDetector();
const capabilities = deviceDetector.getCapabilities();

// 性能检测
const performanceDetector = new PerformanceDetector();
performanceDetector.start();

// 屏幕适配
const screenAdapter = new ScreenAdapter(renderer, camera);
screenAdapter.setPixelRatio();

// 触摸适配
const touchAdapter = new TouchAdapter(camera, renderer.domElement);
if (capabilities.touch) {
    touchAdapter.enable();
}

// 资源管理
const resourceManager = new ResourceManager();

// 性能监控
const performanceMonitor = new PerformanceMonitor();

// 添加光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
scene.add(directionalLight);

// 创建物体
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = resourceManager.createMaterial({ color: 0x00ff00 });
const cube = new THREE.Mesh(geometry, material);
cube.castShadow = true;
scene.add(cube);

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

    // 性能监控开始
    performanceMonitor.begin();

    // 更新物体
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;

    // 渲染场景
    renderer.render(scene, camera);

    // 更新性能指标
    performanceMonitor.updateMetrics(renderer);

    // 性能监控结束
    performanceMonitor.end();
}

animate();

// 窗口大小改变时更新
window.addEventListener('resize', () => {
    screenAdapter.updateSize();
});

// 页面卸载时清理资源
window.addEventListener('unload', () => {
    resourceManager.dispose();
});
```

## 练习

1. 实现设备检测
2. 添加响应式设计
3. 优化资源管理
4. 实现性能监控

## 下一步学习

在下一章中，我们将学习：
- 性能优化
- 最佳实践
- 项目部署
- 发布上线

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！