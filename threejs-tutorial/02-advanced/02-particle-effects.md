# 粒子系统与特效

在 Three.js 中，粒子系统是创建各种视觉效果的重要工具，从简单的雨雪效果到复杂的爆炸和魔法效果都可以实现。

## 基础粒子系统

### 1. Points 基础

![基础粒子效果](https://threejs.org/examples/screenshots/webgl_points_basic.jpg)

*图 7.1: 基础粒子效果示例*

```javascript
// 创建粒子几何体
const geometry = new THREE.BufferGeometry();
const count = 5000;

// 创建顶点位置数组
const positions = new Float32Array(count * 3);
const colors = new Float32Array(count * 3);

// 随机生成粒子位置和颜色
for (let i = 0; i < count; i++) {
    // 位置
    positions[i * 3] = (Math.random() - 0.5) * 10;     // x
    positions[i * 3 + 1] = (Math.random() - 0.5) * 10; // y
    positions[i * 3 + 2] = (Math.random() - 0.5) * 10; // z

    // 颜色
    colors[i * 3] = Math.random();     // r
    colors[i * 3 + 1] = Math.random(); // g
    colors[i * 3 + 2] = Math.random(); // b
}

// 设置属性
geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

// 创建粒子材质
const material = new THREE.PointsMaterial({
    size: 0.05,
    vertexColors: true,
    transparent: true,
    opacity: 0.8
});

// 创建粒子系统
const particles = new THREE.Points(geometry, material);
scene.add(particles);
```

### 2. 粒子动画

```javascript
// 在动画循环中更新粒子
function animate() {
    requestAnimationFrame(animate);

    // 更新粒子位置
    const positions = particles.geometry.attributes.position.array;
    for (let i = 0; i < positions.length; i += 3) {
        positions[i + 1] += Math.sin(Date.now() * 0.001 + i) * 0.01;
    }
    particles.geometry.attributes.position.needsUpdate = true;

    renderer.render(scene, camera);
}
```

## 高级粒子效果

### 1. 纹理粒子

```javascript
// 加载粒子纹理
const textureLoader = new THREE.TextureLoader();
const particleTexture = textureLoader.load('particle.png');

// 创建粒子材质
const material = new THREE.PointsMaterial({
    size: 0.1,
    map: particleTexture,
    transparent: true,
    blending: THREE.AdditiveBlending,
    depthWrite: false
});

// 创建粒子系统
const particles = new THREE.Points(geometry, material);
```

### 2. 动态粒子系统

```javascript
// 创建动态粒子系统
class ParticleSystem {
    constructor(count) {
        this.count = count;
        this.particles = [];
        this.init();
    }

    init() {
        // 创建粒子
        for (let i = 0; i < this.count; i++) {
            const particle = {
                position: new THREE.Vector3(
                    (Math.random() - 0.5) * 10,
                    (Math.random() - 0.5) * 10,
                    (Math.random() - 0.5) * 10
                ),
                velocity: new THREE.Vector3(
                    (Math.random() - 0.5) * 0.02,
                    (Math.random() - 0.5) * 0.02,
                    (Math.random() - 0.5) * 0.02
                ),
                life: 1.0
            };
            this.particles.push(particle);
        }
    }

    update() {
        // 更新粒子
        for (let i = 0; i < this.particles.length; i++) {
            const particle = this.particles[i];

            // 更新位置
            particle.position.add(particle.velocity);

            // 更新生命值
            particle.life -= 0.01;

            // 重置死亡粒子
            if (particle.life <= 0) {
                particle.position.set(
                    (Math.random() - 0.5) * 10,
                    (Math.random() - 0.5) * 10,
                    (Math.random() - 0.5) * 10
                );
                particle.life = 1.0;
            }
        }
    }
}
```

## 特效实现

### 1. 爆炸效果

```javascript
// 创建爆炸效果
function createExplosion(position, count = 1000) {
    const geometry = new THREE.BufferGeometry();
    const positions = new Float32Array(count * 3);
    const velocities = [];

    // 初始化粒子
    for (let i = 0; i < count; i++) {
        // 位置
        positions[i * 3] = position.x;
        positions[i * 3 + 1] = position.y;
        positions[i * 3 + 2] = position.z;

        // 速度
        velocities.push(new THREE.Vector3(
            (Math.random() - 0.5) * 0.2,
            (Math.random() - 0.5) * 0.2,
            (Math.random() - 0.5) * 0.2
        ));
    }

    geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));

    // 创建粒子系统
    const material = new THREE.PointsMaterial({
        size: 0.05,
        color: 0xff0000,
        transparent: true,
        opacity: 1.0
    });

    const particles = new THREE.Points(geometry, material);
    scene.add(particles);

    // 动画
    function animate() {
        const positions = particles.geometry.attributes.position.array;

        for (let i = 0; i < count; i++) {
            positions[i * 3] += velocities[i].x;
            positions[i * 3 + 1] += velocities[i].y;
            positions[i * 3 + 2] += velocities[i].z;

            // 添加重力
            velocities[i].y -= 0.01;
        }

        particles.geometry.attributes.position.needsUpdate = true;
        material.opacity -= 0.01;

        if (material.opacity > 0) {
            requestAnimationFrame(animate);
        } else {
            scene.remove(particles);
        }
    }

    animate();
}
```

### 2. 魔法效果

```javascript
// 创建魔法效果
function createMagicEffect(position) {
    const geometry = new THREE.BufferGeometry();
    const count = 100;
    const positions = new Float32Array(count * 3);
    const colors = new Float32Array(count * 3);

    // 初始化粒子
    for (let i = 0; i < count; i++) {
        const angle = (i / count) * Math.PI * 2;
        const radius = 1;

        positions[i * 3] = position.x + Math.cos(angle) * radius;
        positions[i * 3 + 1] = position.y;
        positions[i * 3 + 2] = position.z + Math.sin(angle) * radius;

        colors[i * 3] = 0.5 + Math.cos(angle) * 0.5;     // r
        colors[i * 3 + 1] = 0.5 + Math.sin(angle) * 0.5; // g
        colors[i * 3 + 2] = 1.0;                         // b
    }

    geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

    const material = new THREE.PointsMaterial({
        size: 0.1,
        vertexColors: true,
        transparent: true,
        blending: THREE.AdditiveBlending
    });

    const particles = new THREE.Points(geometry, material);
    scene.add(particles);

    // 动画
    let time = 0;
    function animate() {
        time += 0.01;
        const positions = particles.geometry.attributes.position.array;

        for (let i = 0; i < count; i++) {
            const angle = (i / count) * Math.PI * 2 + time;
            const radius = 1 + Math.sin(time * 2) * 0.2;

            positions[i * 3] = position.x + Math.cos(angle) * radius;
            positions[i * 3 + 1] = position.y + Math.sin(time * 3) * 0.2;
            positions[i * 3 + 2] = position.z + Math.sin(angle) * radius;
        }

        particles.geometry.attributes.position.needsUpdate = true;

        if (time < 5) {
            requestAnimationFrame(animate);
        } else {
            scene.remove(particles);
        }
    }

    animate();
}
```

## 性能优化

### 1. 粒子数量控制

```javascript
// 根据设备性能调整粒子数量
function getOptimalParticleCount() {
    const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
    return isMobile ? 1000 : 5000;
}

// 使用实例化渲染
const instancedGeometry = new THREE.InstancedBufferGeometry();
const instanceCount = getOptimalParticleCount();
const dummy = new THREE.Object3D();

// 设置实例化属性
instancedGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
instancedGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));
```

### 2. 内存管理

```javascript
// 粒子系统资源管理
class ParticleSystem {
    constructor() {
        this.particles = [];
        this.geometries = [];
        this.materials = [];
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

        // 清空数组
        this.particles = [];
        this.geometries = [];
        this.materials = [];
    }
}
```

## 实战：创建一个粒子效果场景

让我们创建一个展示各种粒子效果的场景：

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
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 创建基础粒子系统
const particleCount = 1000;
const geometry = new THREE.BufferGeometry();
const positions = new Float32Array(particleCount * 3);
const colors = new Float32Array(particleCount * 3);

// 初始化粒子
for (let i = 0; i < particleCount; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 10;
    positions[i * 3 + 1] = (Math.random() - 0.5) * 10;
    positions[i * 3 + 2] = (Math.random() - 0.5) * 10;

    colors[i * 3] = Math.random();
    colors[i * 3 + 1] = Math.random();
    colors[i * 3 + 2] = Math.random();
}

geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

// 创建粒子材质
const material = new THREE.PointsMaterial({
    size: 0.05,
    vertexColors: true,
    transparent: true,
    opacity: 0.8
});

// 创建粒子系统
const particles = new THREE.Points(geometry, material);
scene.add(particles);

// 添加交互
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

window.addEventListener('click', (event) => {
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(scene.children);

    if (intersects.length > 0) {
        createExplosion(intersects[0].point);
    }
});

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新粒子
    const positions = particles.geometry.attributes.position.array;
    for (let i = 0; i < particleCount; i++) {
        positions[i * 3 + 1] += Math.sin(Date.now() * 0.001 + i) * 0.01;
    }
    particles.geometry.attributes.position.needsUpdate = true;

    renderer.render(scene, camera);
}

animate();
```

## 练习

1. 创建一个基础的粒子系统
2. 实现粒子的动态效果
3. 添加交互式粒子效果
4. 优化粒子系统性能

## 下一步学习

在下一章中，我们将学习：
- 后期处理与滤镜
- 渲染通道
- 自定义后处理
- 性能优化