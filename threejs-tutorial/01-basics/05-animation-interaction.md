# 动画与交互基础

在 Three.js 中，动画和交互是创建生动 3D 应用的关键要素。通过合理的动画设计和交互处理，可以让场景更加生动有趣。

## 基础动画原理

### 1. 动画循环

动画循环是 Three.js 中实现动画的基础，使用 `requestAnimationFrame` 来创建平滑的动画效果。

```javascript
// 创建动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新场景
    updateScene();

    // 渲染场景
    renderer.render(scene, camera);
}

// 开始动画
animate();
```

### 2. 简单动画

![简单动画示例](https://threejs.org/examples/screenshots/webgl_animation_skinning_morph.jpg)

*图 5.1: 简单动画效果示例*

```javascript
// 旋转动画
function updateScene() {
    // 旋转物体
    mesh.rotation.x += 0.01;
    mesh.rotation.y += 0.01;

    // 移动物体
    mesh.position.y = Math.sin(Date.now() * 0.001) * 0.5;

    // 缩放物体
    mesh.scale.x = 1 + Math.sin(Date.now() * 0.002) * 0.2;
}
```

### 3. 补间动画

使用 GSAP 或 Tween.js 实现更复杂的补间动画。

```javascript
import * as TWEEN from 'three/examples/jsm/libs/tween.module.min.js';

// 创建补间动画
const tween = new TWEEN.Tween(mesh.position)
    .to({ x: 2, y: 2, z: 2 }, 1000)  // 目标位置和持续时间
    .easing(TWEEN.Easing.Quadratic.Out)  // 缓动函数
    .onUpdate(() => {
        // 更新回调
    })
    .start();

// 在动画循环中更新
function animate() {
    requestAnimationFrame(animate);
    TWEEN.update();
    renderer.render(scene, camera);
}
```

## 用户交互处理

### 1. 鼠标交互

![鼠标交互示例](https://threejs.org/examples/screenshots/webgl_interactive_cubes.jpg)

*图 5.2: 鼠标交互效果示例*

```javascript
// 创建射线投射器
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

// 监听鼠标移动
window.addEventListener('mousemove', (event) => {
    // 计算鼠标位置
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
});

// 在动画循环中检测
function updateScene() {
    // 更新射线
    raycaster.setFromCamera(mouse, camera);

    // 检测相交
    const intersects = raycaster.intersectObjects(scene.children);

    // 处理相交结果
    if (intersects.length > 0) {
        // 高亮显示
        intersects[0].object.material.emissive.setHex(0x666666);
    }
}
```

### 2. 键盘控制

```javascript
// 监听键盘事件
window.addEventListener('keydown', (event) => {
    switch(event.key) {
        case 'ArrowUp':
            // 向上移动
            mesh.position.y += 0.1;
            break;
        case 'ArrowDown':
            // 向下移动
            mesh.position.y -= 0.1;
            break;
        case 'ArrowLeft':
            // 向左移动
            mesh.position.x -= 0.1;
            break;
        case 'ArrowRight':
            // 向右移动
            mesh.position.x += 0.1;
            break;
    }
});
```

### 3. 触摸交互

```javascript
// 监听触摸事件
window.addEventListener('touchstart', (event) => {
    event.preventDefault();

    // 获取触摸位置
    const touch = event.touches[0];
    mouse.x = (touch.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(touch.clientY / window.innerHeight) * 2 + 1;

    // 检测相交
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(scene.children);

    if (intersects.length > 0) {
        // 处理触摸交互
    }
});
```

## 事件系统

### 1. 自定义事件

```javascript
// 创建事件分发器
const eventDispatcher = new THREE.EventDispatcher();

// 添加事件监听
eventDispatcher.addEventListener('objectSelected', (event) => {
    console.log('Object selected:', event.object);
});

// 触发事件
eventDispatcher.dispatchEvent({
    type: 'objectSelected',
    object: mesh
});
```

### 2. 对象事件

```javascript
// 为对象添加事件监听
mesh.addEventListener('click', (event) => {
    console.log('Mesh clicked');
});

// 在射线检测中触发事件
if (intersects.length > 0) {
    intersects[0].object.dispatchEvent({
        type: 'click',
        point: intersects[0].point
    });
}
```

## 性能优化

### 1. 动画优化

```javascript
// 使用 delta 时间
let clock = new THREE.Clock();

function animate() {
    requestAnimationFrame(animate);

    const delta = clock.getDelta();

    // 使用 delta 时间更新动画
    mesh.rotation.x += 0.5 * delta;
    mesh.rotation.y += 0.5 * delta;

    renderer.render(scene, camera);
}
```

### 2. 交互优化

```javascript
// 使用节流函数
function throttle(func, limit) {
    let inThrottle;
    return function() {
        const args = arguments;
        const context = this;
        if (!inThrottle) {
            func.apply(context, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    }
}

// 应用节流
window.addEventListener('mousemove', throttle((event) => {
    // 处理鼠标移动
}, 16));  // 约60fps
```

### 3. 事件优化

```javascript
// 使用事件委托
scene.addEventListener('click', (event) => {
    // 处理所有对象的点击事件
    if (event.object) {
        // 处理特定对象
    }
});

// 移除事件监听
function dispose() {
    // 移除所有事件监听
    window.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('keydown', onKeyDown);
    window.removeEventListener('touchstart', onTouchStart);
}
```

## 实战：创建一个交互式场景

让我们创建一个包含动画和交互的完整场景：

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
document.body.appendChild(renderer.domElement);

// 创建物体
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshStandardMaterial({
    color: 0x00ff00,
    metalness: 0.3,
    roughness: 0.4
});
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

// 创建射线投射器
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

// 监听鼠标移动
window.addEventListener('mousemove', (event) => {
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
});

// 创建时钟
const clock = new THREE.Clock();

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    const delta = clock.getDelta();

    // 更新射线
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(scene.children);

    // 处理相交
    if (intersects.length > 0) {
        // 高亮显示
        intersects[0].object.material.emissive.setHex(0x666666);

        // 旋转动画
        intersects[0].object.rotation.x += 0.5 * delta;
        intersects[0].object.rotation.y += 0.5 * delta;
    } else {
        // 恢复默认
        mesh.material.emissive.setHex(0x000000);
    }

    renderer.render(scene, camera);
}

// 开始动画
animate();
```

## 练习

1. 创建一个包含多个可交互对象的场景
2. 实现对象的拖拽功能
3. 添加键盘控制功能
4. 实现触摸屏交互

## 下一步学习

在下一章中，我们将学习：
- 高级材质与纹理
- 自定义着色器
- 纹理映射
- 材质优化
