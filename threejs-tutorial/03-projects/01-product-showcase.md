# 3D 产品展示

在 Three.js 中，创建 3D 产品展示是一个常见的应用场景。本章将介绍如何创建一个专业的产品展示系统，包括模型加载、交互控制、动画效果等。

## 基础场景搭建

### 1. 场景配置

![产品展示示例](https://threejs.org/examples/screenshots/webgl_loader_gltf.jpg)

*图 11.1: 3D 产品展示示例*

```javascript
// 创建场景
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xf0f0f0);

// 创建相机
const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.set(0, 2, 5);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

// 创建环境光
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

// 创建主光源
const mainLight = new THREE.DirectionalLight(0xffffff, 1);
mainLight.position.set(5, 5, 5);
mainLight.castShadow = true;
scene.add(mainLight);

// 创建补光
const fillLight = new THREE.DirectionalLight(0xffffff, 0.3);
fillLight.position.set(-5, 3, -5);
scene.add(fillLight);
```

### 2. 相机控制

```javascript
// 创建轨道控制器
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.screenSpacePanning = false;
controls.minDistance = 3;
controls.maxDistance = 10;
controls.maxPolarAngle = Math.PI / 2;

// 创建自动旋转控制器
class AutoRotateController {
    constructor(controls) {
        this.controls = controls;
        this.enabled = false;
        this.speed = 0.5;
    }

    enable() {
        this.enabled = true;
    }

    disable() {
        this.enabled = false;
    }

    update() {
        if (this.enabled) {
            this.controls.autoRotate = true;
            this.controls.autoRotateSpeed = this.speed;
        } else {
            this.controls.autoRotate = false;
        }
    }
}
```

## 产品展示系统

### 1. 产品加载器

```javascript
// 产品加载器
class ProductLoader {
    constructor() {
        this.loader = new GLTFLoader();
        this.dracoLoader = new DRACOLoader();
        this.dracoLoader.setDecoderPath('/path/to/draco/');
        this.loader.setDRACOLoader(this.dracoLoader);
    }

    loadProduct(url) {
        return new Promise((resolve, reject) => {
            this.loader.load(
                url,
                (gltf) => {
                    const model = gltf.scene;
                    this.setupModel(model);
                    resolve(model);
                },
                undefined,
                (error) => reject(error)
            );
        });
    }

    setupModel(model) {
        model.traverse((child) => {
            if (child.isMesh) {
                child.castShadow = true;
                child.receiveShadow = true;

                // 设置材质
                if (child.material) {
                    child.material.envMapIntensity = 1;
                    child.material.needsUpdate = true;
                }
            }
        });
    }
}
```

### 2. 产品控制器

```javascript
// 产品控制器
class ProductController {
    constructor(model) {
        this.model = model;
        this.animations = new Map();
        this.currentAnimation = null;
        this.setupAnimations();
    }

    setupAnimations() {
        // 创建旋转动画
        this.animations.set('rotate', {
            update: (time) => {
                this.model.rotation.y = time * 0.5;
            }
        });

        // 创建缩放动画
        this.animations.set('scale', {
            update: (time) => {
                const scale = 1 + Math.sin(time) * 0.1;
                this.model.scale.set(scale, scale, scale);
            }
        });
    }

    playAnimation(name) {
        this.currentAnimation = this.animations.get(name);
    }

    update(time) {
        if (this.currentAnimation) {
            this.currentAnimation.update(time);
        }
    }
}
```

## 交互系统

### 1. 热点标记

```javascript
// 热点标记系统
class HotspotSystem {
    constructor() {
        this.hotspots = new Map();
        this.raycaster = new THREE.Raycaster();
        this.mouse = new THREE.Vector2();
    }

    createHotspot(position, data) {
        const geometry = new THREE.SphereGeometry(0.1, 32, 32);
        const material = new THREE.MeshBasicMaterial({
            color: 0xff0000,
            transparent: true,
            opacity: 0.8
        });

        const hotspot = new THREE.Mesh(geometry, material);
        hotspot.position.copy(position);
        hotspot.userData = data;

        this.hotspots.set(data.id, hotspot);
        return hotspot;
    }

    update(camera, mouse) {
        this.raycaster.setFromCamera(mouse, camera);
        const intersects = this.raycaster.intersectObjects(
            Array.from(this.hotspots.values())
        );

        if (intersects.length > 0) {
            const hotspot = intersects[0].object;
            this.onHotspotHover(hotspot);
        }
    }

    onHotspotHover(hotspot) {
        // 处理热点悬停事件
        console.log('Hotspot hover:', hotspot.userData);
    }
}
```

### 2. 交互控制器

```javascript
// 交互控制器
class InteractionController {
    constructor(camera, renderer) {
        this.camera = camera;
        this.renderer = renderer;
        this.raycaster = new THREE.Raycaster();
        this.mouse = new THREE.Vector2();
        this.selectedObject = null;
        this.setupEventListeners();
    }

    setupEventListeners() {
        this.renderer.domElement.addEventListener('mousemove', (event) => {
            this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
        });

        this.renderer.domElement.addEventListener('click', (event) => {
            this.raycaster.setFromCamera(this.mouse, this.camera);
            const intersects = this.raycaster.intersectObjects(scene.children, true);

            if (intersects.length > 0) {
                this.onObjectSelected(intersects[0].object);
            }
        });
    }

    onObjectSelected(object) {
        if (this.selectedObject) {
            // 重置之前选中的对象
            this.selectedObject.material.emissive.setHex(0x000000);
        }

        this.selectedObject = object;
        object.material.emissive.setHex(0x333333);
    }
}
```

## 特效系统

### 1. 环境效果

```javascript
// 环境效果系统
class EnvironmentSystem {
    constructor(scene) {
        this.scene = scene;
        this.envMap = null;
        this.setupEnvironment();
    }

    setupEnvironment() {
        // 加载环境贴图
        const envLoader = new THREE.CubeTextureLoader();
        this.envMap = envLoader.load([
            'px.jpg', 'nx.jpg',
            'py.jpg', 'ny.jpg',
            'pz.jpg', 'nz.jpg'
        ]);

        this.scene.environment = this.envMap;
        this.scene.background = this.envMap;
    }

    updateLighting(time) {
        // 更新光照效果
        const mainLight = this.scene.getObjectByName('mainLight');
        if (mainLight) {
            mainLight.position.x = Math.sin(time) * 5;
            mainLight.position.z = Math.cos(time) * 5;
        }
    }
}
```

### 2. 后期处理

```javascript
// 后期处理系统
class PostProcessingSystem {
    constructor(renderer) {
        this.renderer = renderer;
        this.composer = new EffectComposer(renderer);
        this.setupPostProcessing();
    }

    setupPostProcessing() {
        // 添加渲染通道
        const renderPass = new RenderPass(scene, camera);
        this.composer.addPass(renderPass);

        // 添加泛光效果
        const bloomPass = new UnrealBloomPass(
            new THREE.Vector2(window.innerWidth, window.innerHeight),
            1.5,
            0.4,
            0.85
        );
        this.composer.addPass(bloomPass);

        // 添加色彩调整
        const colorPass = new ShaderPass(ColorCorrectionShader);
        this.composer.addPass(colorPass);
    }

    render() {
        this.composer.render();
    }
}
```

## 实战：创建产品展示场景

让我们创建一个完整的产品展示场景：

```javascript
// 创建场景
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 创建控制器
const controls = new THREE.OrbitControls(camera, renderer.domElement);
const autoRotateController = new AutoRotateController(controls);

// 创建产品加载器
const productLoader = new ProductLoader();

// 创建交互系统
const interactionController = new InteractionController(camera, renderer);
const hotspotSystem = new HotspotSystem();

// 创建特效系统
const environmentSystem = new EnvironmentSystem(scene);
const postProcessingSystem = new PostProcessingSystem(renderer);

// 加载产品模型
let productController;
productLoader.loadProduct('product.gltf').then(model => {
    scene.add(model);
    productController = new ProductController(model);

    // 添加热点
    const hotspot1 = hotspotSystem.createHotspot(
        new THREE.Vector3(1, 1, 0),
        { id: 'feature1', title: 'Feature 1' }
    );
    scene.add(hotspot1);
});

// 动画循环
const clock = new THREE.Clock();
function animate() {
    requestAnimationFrame(animate);

    const time = clock.getElapsedTime();

    // 更新控制器
    controls.update();
    autoRotateController.update();

    // 更新产品控制器
    if (productController) {
        productController.update(time);
    }

    // 更新环境系统
    environmentSystem.updateLighting(time);

    // 更新交互系统
    interactionController.update();
    hotspotSystem.update(camera, interactionController.mouse);

    // 渲染场景
    postProcessingSystem.render();
}

animate();

// 窗口大小改变时更新
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
    postProcessingSystem.composer.setSize(window.innerWidth, window.innerHeight);
});
```

## 练习

1. 创建一个基础的产品展示场景
2. 实现产品旋转和缩放动画
3. 添加交互热点和说明
4. 实现环境效果和后期处理

## 下一步学习

在下一章中，我们将学习：
- 数据可视化
- 图表创建
- 动态数据更新
- 交互控制
