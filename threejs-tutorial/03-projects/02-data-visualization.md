# 数据可视化

在 Three.js 中，我们可以创建各种形式的数据可视化效果。本章将介绍如何使用 Three.js 创建 3D 数据可视化，包括基础图表、动态数据更新和交互控制等。

## 基础图表系统

### 1. 柱状图

![3D 柱状图示例](https://threejs.org/examples/screenshots/webgl_geometry_shapes.jpg)

*图 12.1: 3D 柱状图示例*

```javascript
// 柱状图类
class BarChart {
    constructor(scene, data, options = {}) {
        this.scene = scene;
        this.data = data;
        this.options = {
            width: options.width || 10,
            height: options.height || 5,
            depth: options.depth || 0.5,
            spacing: options.spacing || 0.2,
            colors: options.colors || [0x2194ce, 0x32cd32, 0xffd700],
            ...options
        };

        this.bars = [];
        this.createBars();
    }

    createBars() {
        const { width, height, depth, spacing, colors } = this.options;
        const barWidth = (width - (this.data.length - 1) * spacing) / this.data.length;

        this.data.forEach((value, index) => {
            const barHeight = (value / Math.max(...this.data)) * height;
            const geometry = new THREE.BoxGeometry(barWidth, barHeight, depth);
            const material = new THREE.MeshPhongMaterial({
                color: colors[index % colors.length],
                shininess: 30
            });

            const bar = new THREE.Mesh(geometry, material);
            bar.position.x = (index * (barWidth + spacing)) - (width / 2) + (barWidth / 2);
            bar.position.y = barHeight / 2;

            this.bars.push(bar);
            this.scene.add(bar);
        });
    }

    updateData(newData) {
        const { height } = this.options;
        const maxValue = Math.max(...newData);

        this.data = newData;
        this.bars.forEach((bar, index) => {
            const newHeight = (newData[index] / maxValue) * height;
            bar.scale.y = newHeight / bar.geometry.parameters.height;
            bar.position.y = newHeight / 2;
        });
    }
}
```

### 2. 折线图

```javascript
// 折线图类
class LineChart {
    constructor(scene, data, options = {}) {
        this.scene = scene;
        this.data = data;
        this.options = {
            width: options.width || 10,
            height: options.height || 5,
            lineWidth: options.lineWidth || 0.05,
            color: options.color || 0x2194ce,
            ...options
        };

        this.line = null;
        this.points = [];
        this.createLine();
    }

    createLine() {
        const { width, height, lineWidth, color } = this.options;
        const points = [];

        this.data.forEach((value, index) => {
            const x = (index / (this.data.length - 1) - 0.5) * width;
            const y = (value / Math.max(...this.data) - 0.5) * height;
            points.push(new THREE.Vector3(x, y, 0));
        });

        const curve = new THREE.CatmullRomCurve3(points);
        const geometry = new THREE.TubeGeometry(curve, 64, lineWidth, 8, false);
        const material = new THREE.MeshPhongMaterial({ color });

        this.line = new THREE.Mesh(geometry, material);
        this.scene.add(this.line);

        // 创建数据点
        points.forEach(point => {
            const sphereGeometry = new THREE.SphereGeometry(lineWidth * 1.5, 16, 16);
            const sphereMaterial = new THREE.MeshPhongMaterial({ color });
            const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
            sphere.position.copy(point);
            this.points.push(sphere);
            this.scene.add(sphere);
        });
    }

    updateData(newData) {
        const { width, height } = this.options;
        const maxValue = Math.max(...newData);
        const points = [];

        newData.forEach((value, index) => {
            const x = (index / (newData.length - 1) - 0.5) * width;
            const y = (value / maxValue - 0.5) * height;
            points.push(new THREE.Vector3(x, y, 0));
        });

        // 更新线条
        const curve = new THREE.CatmullRomCurve3(points);
        const newGeometry = new THREE.TubeGeometry(curve, 64, this.options.lineWidth, 8, false);
        this.line.geometry.dispose();
        this.line.geometry = newGeometry;

        // 更新数据点
        points.forEach((point, index) => {
            this.points[index].position.copy(point);
        });
    }
}
```

## 动态数据系统

### 1. 数据管理器

```javascript
// 数据管理器
class DataManager {
    constructor() {
        this.data = new Map();
        this.subscribers = new Map();
    }

    setData(key, value) {
        this.data.set(key, value);
        this.notifySubscribers(key);
    }

    getData(key) {
        return this.data.get(key);
    }

    subscribe(key, callback) {
        if (!this.subscribers.has(key)) {
            this.subscribers.set(key, new Set());
        }
        this.subscribers.get(key).add(callback);
    }

    unsubscribe(key, callback) {
        if (this.subscribers.has(key)) {
            this.subscribers.get(key).delete(callback);
        }
    }

    notifySubscribers(key) {
        if (this.subscribers.has(key)) {
            const value = this.data.get(key);
            this.subscribers.get(key).forEach(callback => callback(value));
        }
    }
}
```

### 2. 动画控制器

```javascript
// 动画控制器
class AnimationController {
    constructor(duration = 1000) {
        this.duration = duration;
        this.animations = new Map();
    }

    animate(startValue, endValue, onUpdate, onComplete) {
        const startTime = performance.now();
        const animation = {
            startValue,
            endValue,
            onUpdate,
            onComplete
        };

        const id = Math.random().toString(36).substr(2, 9);
        this.animations.set(id, animation);

        return id;
    }

    update() {
        const currentTime = performance.now();

        this.animations.forEach((animation, id) => {
            const elapsed = currentTime - animation.startTime;
            const progress = Math.min(elapsed / this.duration, 1);

            if (progress < 1) {
                const currentValue = this.interpolate(
                    animation.startValue,
                    animation.endValue,
                    progress
                );
                animation.onUpdate(currentValue);
            } else {
                animation.onUpdate(animation.endValue);
                animation.onComplete?.();
                this.animations.delete(id);
            }
        });
    }

    interpolate(start, end, progress) {
        if (typeof start === 'number') {
            return start + (end - start) * progress;
        } else if (start instanceof THREE.Vector3) {
            return new THREE.Vector3(
                this.interpolate(start.x, end.x, progress),
                this.interpolate(start.y, end.y, progress),
                this.interpolate(start.z, end.z, progress)
            );
        }
        return end;
    }
}
```

## 交互系统

### 1. 图表交互器

```javascript
// 图表交互器
class ChartInteractor {
    constructor(camera, renderer) {
        this.camera = camera;
        this.renderer = renderer;
        this.raycaster = new THREE.Raycaster();
        this.mouse = new THREE.Vector2();
        this.hoveredObject = null;
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
                this.onObjectClick(intersects[0].object);
            }
        });
    }

    update(scene) {
        this.raycaster.setFromCamera(this.mouse, this.camera);
        const intersects = this.raycaster.intersectObjects(scene.children, true);

        if (intersects.length > 0) {
            const object = intersects[0].object;
            if (this.hoveredObject !== object) {
                this.onObjectHover(object);
            }
            this.hoveredObject = object;
        } else if (this.hoveredObject) {
            this.onObjectUnhover(this.hoveredObject);
            this.hoveredObject = null;
        }
    }

    onObjectHover(object) {
        if (object.material) {
            object.material.emissive.setHex(0x333333);
        }
    }

    onObjectUnhover(object) {
        if (object.material) {
            object.material.emissive.setHex(0x000000);
        }
    }

    onObjectClick(object) {
        console.log('Clicked object:', object);
    }
}
```

### 2. 相机控制器

```javascript
// 相机控制器
class CameraController {
    constructor(camera, renderer) {
        this.camera = camera;
        this.renderer = renderer;
        this.controls = new THREE.OrbitControls(camera, renderer.domElement);
        this.setupControls();
    }

    setupControls() {
        this.controls.enableDamping = true;
        this.controls.dampingFactor = 0.05;
        this.controls.screenSpacePanning = false;
        this.controls.minDistance = 5;
        this.controls.maxDistance = 20;
        this.controls.maxPolarAngle = Math.PI / 2;
    }

    update() {
        this.controls.update();
    }

    focusOnObject(object) {
        const box = new THREE.Box3().setFromObject(object);
        const center = box.getCenter(new THREE.Vector3());
        const size = box.getSize(new THREE.Vector3());

        const maxDim = Math.max(size.x, size.y, size.z);
        const fov = this.camera.fov * (Math.PI / 180);
        let cameraZ = Math.abs(maxDim / Math.sin(fov / 2));

        const cameraPosition = new THREE.Vector3(
            center.x,
            center.y,
            center.z + cameraZ
        );

        this.camera.position.copy(cameraPosition);
        this.controls.target.copy(center);
    }
}
```

## 实战：创建数据可视化场景

让我们创建一个完整的数据可视化场景：

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
camera.position.set(0, 5, 10);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

// 创建光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
scene.add(directionalLight);

// 创建数据管理器
const dataManager = new DataManager();

// 创建图表
const barChart = new BarChart(scene, [10, 20, 15, 25, 30]);
const lineChart = new LineChart(scene, [5, 15, 10, 20, 25]);

// 创建控制器
const cameraController = new CameraController(camera, renderer);
const chartInteractor = new ChartInteractor(camera, renderer);
const animationController = new AnimationController();

// 订阅数据更新
dataManager.subscribe('chartData', (newData) => {
    animationController.animate(
        barChart.data,
        newData,
        (value) => barChart.updateData(value),
        () => console.log('Animation complete')
    );
});

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新控制器
    cameraController.update();
    chartInteractor.update(scene);
    animationController.update();

    // 渲染场景
    renderer.render(scene, camera);
}

animate();

// 窗口大小改变时更新
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});

// 模拟数据更新
setInterval(() => {
    const newData = Array.from({ length: 5 }, () => Math.random() * 30);
    dataManager.setData('chartData', newData);
}, 3000);
```

## 练习

1. 创建一个基础的 3D 柱状图
2. 实现折线图的数据更新动画
3. 添加图表交互功能
4. 实现相机聚焦效果

## 下一步学习

在下一章中，我们将学习：
- 游戏开发基础
- 物理引擎集成
- 碰撞检测
- 游戏逻辑实现
