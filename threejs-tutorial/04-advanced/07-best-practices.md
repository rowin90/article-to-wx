# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 最佳实践

在 Three.js 开发中，遵循最佳实践可以帮助我们构建更高质量、更易维护的应用。本章将介绍一些重要的最佳实践。

## 代码组织

### 1. 项目结构

![最佳实践示例](https://threejs.org/examples/screenshots/webgl_loader_gltf.jpg)

*图 22.1: 最佳实践示例*

```javascript
// 项目结构示例
project/
├── src/
│   ├── core/
│   │   ├── Scene.js
│   │   ├── Camera.js
│   │   └── Renderer.js
│   ├── objects/
│   │   ├── geometries/
│   │   ├── materials/
│   │   └── lights/
│   ├── utils/
│   │   ├── loaders/
│   │   ├── helpers/
│   │   └── controls/
│   └── main.js
├── assets/
│   ├── models/
│   ├── textures/
│   └── sounds/
└── index.html
```

### 2. 模块化设计

```javascript
// 场景管理器
class SceneManager {
    constructor() {
        this.scene = new THREE.Scene();
        this.camera = new THREE.PerspectiveCamera(
            75,
            window.innerWidth / window.innerHeight,
            0.1,
            1000
        );
        this.renderer = new THREE.WebGLRenderer({ antialias: true });
        this.controls = null;
        this.objects = new Map();
    }

    init() {
        this.setupRenderer();
        this.setupCamera();
        this.setupControls();
        this.setupLights();
        this.setupEventListeners();
    }

    setupRenderer() {
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        this.renderer.shadowMap.enabled = true;
        document.body.appendChild(this.renderer.domElement);
    }

    setupCamera() {
        this.camera.position.set(0, 5, 10);
        this.camera.lookAt(0, 0, 0);
    }

    setupControls() {
        this.controls = new OrbitControls(this.camera, this.renderer.domElement);
        this.controls.enableDamping = true;
    }

    setupLights() {
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
        this.scene.add(ambientLight);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
        directionalLight.position.set(5, 5, 5);
        directionalLight.castShadow = true;
        this.scene.add(directionalLight);
    }

    setupEventListeners() {
        window.addEventListener('resize', () => this.onWindowResize());
    }

    onWindowResize() {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
    }

    addObject(name, object) {
        this.objects.set(name, object);
        this.scene.add(object);
    }

    removeObject(name) {
        const object = this.objects.get(name);
        if (object) {
            this.scene.remove(object);
            this.objects.delete(name);
        }
    }

    update() {
        this.controls.update();
    }

    render() {
        this.renderer.render(this.scene, this.camera);
    }

    dispose() {
        this.objects.forEach(object => {
            if (object.geometry) object.geometry.dispose();
            if (object.material) {
                if (Array.isArray(object.material)) {
                    object.material.forEach(material => material.dispose());
                } else {
                    object.material.dispose();
                }
            }
        });
        this.renderer.dispose();
    }
}
```

## 错误处理

### 1. 错误管理器

```javascript
// 错误管理器
class ErrorManager {
    constructor() {
        this.errors = [];
        this.maxErrors = 100;
    }

    handleError(error, context = '') {
        const errorInfo = {
            message: error.message,
            stack: error.stack,
            context,
            timestamp: new Date().toISOString()
        };

        this.errors.push(errorInfo);
        if (this.errors.length > this.maxErrors) {
            this.errors.shift();
        }

        console.error(`[${context}] ${error.message}`, error);
    }

    getErrors() {
        return this.errors;
    }

    clearErrors() {
        this.errors = [];
    }
}
```

### 2. 资源加载错误处理

```javascript
// 资源加载管理器
class ResourceLoader {
    constructor(errorManager) {
        this.errorManager = errorManager;
        this.loadingManager = new THREE.LoadingManager(
            // 加载完成回调
            () => console.log('Loading complete'),
            // 加载进度回调
            (url, itemsLoaded, itemsTotal) => {
                console.log(`Loading file: ${url} (${itemsLoaded}/${itemsTotal})`);
            },
            // 加载错误回调
            (url) => {
                this.errorManager.handleError(
                    new Error(`Error loading ${url}`),
                    'ResourceLoader'
                );
            }
        );
    }

    loadTexture(url) {
        return new Promise((resolve, reject) => {
            const loader = new THREE.TextureLoader(this.loadingManager);
            loader.load(
                url,
                (texture) => resolve(texture),
                undefined,
                (error) => reject(error)
            );
        });
    }

    loadModel(url) {
        return new Promise((resolve, reject) => {
            const loader = new THREE.GLTFLoader(this.loadingManager);
            loader.load(
                url,
                (gltf) => resolve(gltf),
                undefined,
                (error) => reject(error)
            );
        });
    }
}
```

## 性能优化

### 1. 性能监控

```javascript
// 性能监控器
class PerformanceMonitor {
    constructor() {
        this.metrics = {
            fps: 0,
            frameTime: 0,
            memory: 0,
            drawCalls: 0,
            triangles: 0
        };
        this.frameCount = 0;
        this.lastTime = performance.now();
    }

    update(renderer) {
        const currentTime = performance.now();
        const deltaTime = currentTime - this.lastTime;

        this.frameCount++;

        if (deltaTime >= 1000) {
            this.metrics.fps = Math.round((this.frameCount * 1000) / deltaTime);
            this.metrics.frameTime = deltaTime / this.frameCount;
            this.metrics.drawCalls = renderer.info.render.calls;
            this.metrics.triangles = renderer.info.render.triangles;

            if (performance.memory) {
                this.metrics.memory = performance.memory.usedJSHeapSize / 1048576;
            }

            this.frameCount = 0;
            this.lastTime = currentTime;
        }
    }

    getMetrics() {
        return this.metrics;
    }
}
```

### 2. 资源管理

```javascript
// 资源管理器
class ResourceManager {
    constructor() {
        this.resources = new Map();
        this.loadingPromises = new Map();
    }

    async loadResource(key, loader) {
        if (this.resources.has(key)) {
            return this.resources.get(key);
        }

        if (this.loadingPromises.has(key)) {
            return this.loadingPromises.get(key);
        }

        const promise = loader().catch(error => {
            this.loadingPromises.delete(key);
            throw error;
        });

        this.loadingPromises.set(key, promise);
        const resource = await promise;
        this.resources.set(key, resource);
        this.loadingPromises.delete(key);

        return resource;
    }

    getResource(key) {
        return this.resources.get(key);
    }

    dispose() {
        this.resources.forEach(resource => {
            if (resource.dispose) {
                resource.dispose();
            }
        });
        this.resources.clear();
        this.loadingPromises.clear();
    }
}
```

## 实战：最佳实践应用

让我们创建一个完整的应用示例：

```javascript
// 创建应用
class ThreeJSApp {
    constructor() {
        this.sceneManager = new SceneManager();
        this.errorManager = new ErrorManager();
        this.resourceLoader = new ResourceLoader(this.errorManager);
        this.resourceManager = new ResourceManager();
        this.performanceMonitor = new PerformanceMonitor();
    }

    async init() {
        try {
            // 初始化场景
            this.sceneManager.init();

            // 加载资源
            await this.loadResources();

            // 创建场景对象
            this.createSceneObjects();

            // 开始动画循环
            this.animate();
        } catch (error) {
            this.errorManager.handleError(error, 'ThreeJSApp.init');
        }
    }

    async loadResources() {
        try {
            // 加载纹理
            const texture = await this.resourceLoader.loadTexture('texture.jpg');
            this.resourceManager.resources.set('texture', texture);

            // 加载模型
            const model = await this.resourceLoader.loadModel('model.glb');
            this.resourceManager.resources.set('model', model);
        } catch (error) {
            this.errorManager.handleError(error, 'ThreeJSApp.loadResources');
        }
    }

    createSceneObjects() {
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
        this.sceneManager.addObject('ground', ground);

        // 添加模型
        const model = this.resourceManager.getResource('model');
        if (model) {
            this.sceneManager.addObject('model', model.scene);
        }
    }

    animate() {
        requestAnimationFrame(() => this.animate());

        try {
            // 更新场景
            this.sceneManager.update();

            // 更新性能监控
            this.performanceMonitor.update(this.sceneManager.renderer);

            // 渲染场景
            this.sceneManager.render();
        } catch (error) {
            this.errorManager.handleError(error, 'ThreeJSApp.animate');
        }
    }

    dispose() {
        this.sceneManager.dispose();
        this.resourceManager.dispose();
    }
}

// 创建并初始化应用
const app = new ThreeJSApp();
app.init().catch(error => {
    console.error('Failed to initialize app:', error);
});

// 页面卸载时清理资源
window.addEventListener('unload', () => {
    app.dispose();
});
```
感谢大家阅读到这里，本系列的 Three.js 学习已经结束，希望对大家有所帮助。

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！