# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 物理引擎集成

在 Three.js 中，我们可以集成物理引擎来实现真实的物理效果。本章将介绍如何使用 Cannon.js 物理引擎来创建物理模拟。

## 物理引擎基础

### 1. 物理世界设置

![物理引擎示例](https://threejs.org/examples/screenshots/physics_ammo_break.jpg)

*图 17.1: 物理引擎示例*

```javascript
// 物理世界管理器
class PhysicsWorld {
    constructor() {
        // 创建物理世界
        this.world = new CANNON.World();
        this.world.gravity.set(0, -9.82, 0); // 设置重力
        this.world.broadphase = new CANNON.NaiveBroadphase();
        this.world.solver.iterations = 10;

        // 存储物理对象
        this.bodies = new Map();
        this.meshes = new Map();
    }

    addBody(mesh, body) {
        this.bodies.set(mesh.uuid, body);
        this.meshes.set(mesh.uuid, mesh);
        this.world.addBody(body);
    }

    removeBody(mesh) {
        const body = this.bodies.get(mesh.uuid);
        if (body) {
            this.world.removeBody(body);
            this.bodies.delete(mesh.uuid);
            this.meshes.delete(mesh.uuid);
        }
    }

    update(deltaTime) {
        // 更新物理世界
        this.world.step(deltaTime);

        // 更新网格位置
        this.bodies.forEach((body, uuid) => {
            const mesh = this.meshes.get(uuid);
            if (mesh) {
                mesh.position.copy(body.position);
                mesh.quaternion.copy(body.quaternion);
            }
        });
    }
}
```

### 2. 物理对象创建

```javascript
// 物理对象管理器
class PhysicsObjectManager {
    constructor(world) {
        this.world = world;
    }

    createBox(size, position, mass = 1) {
        // 创建物理体
        const shape = new CANNON.Box(new CANNON.Vec3(size.x/2, size.y/2, size.z/2));
        const body = new CANNON.Body({
            mass: mass,
            position: new CANNON.Vec3(position.x, position.y, position.z),
            shape: shape
        });

        // 创建网格
        const geometry = new THREE.BoxGeometry(size.x, size.y, size.z);
        const material = new THREE.MeshStandardMaterial({ color: 0xff0000 });
        const mesh = new THREE.Mesh(geometry, material);

        // 添加到物理世界
        this.world.addBody(mesh, body);

        return { mesh, body };
    }

    createSphere(radius, position, mass = 1) {
        // 创建物理体
        const shape = new CANNON.Sphere(radius);
        const body = new CANNON.Body({
            mass: mass,
            position: new CANNON.Vec3(position.x, position.y, position.z),
            shape: shape
        });

        // 创建网格
        const geometry = new THREE.SphereGeometry(radius, 32, 32);
        const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 });
        const mesh = new THREE.Mesh(geometry, material);

        // 添加到物理世界
        this.world.addBody(mesh, body);

        return { mesh, body };
    }

    createPlane(size, position) {
        // 创建物理体
        const shape = new CANNON.Plane();
        const body = new CANNON.Body({
            mass: 0, // 静态物体
            position: new CANNON.Vec3(position.x, position.y, position.z),
            shape: shape
        });
        body.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), -Math.PI/2);

        // 创建网格
        const geometry = new THREE.PlaneGeometry(size.x, size.z);
        const material = new THREE.MeshStandardMaterial({
            color: 0x808080,
            side: THREE.DoubleSide
        });
        const mesh = new THREE.Mesh(geometry, material);
        mesh.rotation.x = -Math.PI/2;

        // 添加到物理世界
        this.world.addBody(mesh, body);

        return { mesh, body };
    }
}
```

## 碰撞检测

### 1. 碰撞管理器

```javascript
// 碰撞管理器
class CollisionManager {
    constructor(world) {
        this.world = world;
        this.collisions = new Map();

        // 设置碰撞回调
        this.world.addEventListener('beginContact', this.onBeginContact.bind(this));
        this.world.addEventListener('endContact', this.onEndContact.bind(this));
    }

    onBeginContact(event) {
        const bodyA = event.bodyA;
        const bodyB = event.bodyB;

        // 获取碰撞的网格
        const meshA = this.world.meshes.get(bodyA.uuid);
        const meshB = this.world.meshes.get(bodyB.uuid);

        if (meshA && meshB) {
            // 处理碰撞开始
            this.handleCollision(meshA, meshB, event);
        }
    }

    onEndContact(event) {
        const bodyA = event.bodyA;
        const bodyB = event.bodyB;

        // 获取碰撞的网格
        const meshA = this.world.meshes.get(bodyA.uuid);
        const meshB = this.world.meshes.get(bodyB.uuid);

        if (meshA && meshB) {
            // 处理碰撞结束
            this.handleCollisionEnd(meshA, meshB);
        }
    }

    handleCollision(meshA, meshB, event) {
        // 计算碰撞点
        const contact = event.contact;
        const point = contact.bi === meshA.body ?
            contact.ri : contact.rj;

        // 创建碰撞效果
        this.createCollisionEffect(point);

        // 播放碰撞音效
        this.playCollisionSound(contact.getImpactVelocityAlongNormal());
    }

    handleCollisionEnd(meshA, meshB) {
        // 处理碰撞结束逻辑
    }

    createCollisionEffect(position) {
        // 创建粒子效果
        const geometry = new THREE.SphereGeometry(0.1, 8, 8);
        const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });
        const particle = new THREE.Mesh(geometry, material);

        particle.position.copy(position);
        this.world.scene.add(particle);

        // 动画效果
        gsap.to(particle.scale, {
            x: 0,
            y: 0,
            z: 0,
            duration: 0.5,
            onComplete: () => {
                this.world.scene.remove(particle);
                geometry.dispose();
                material.dispose();
            }
        });
    }

    playCollisionSound(velocity) {
        // 根据碰撞速度播放音效
        const volume = Math.min(Math.abs(velocity) / 10, 1);
        // 播放音效
    }
}
```

### 2. 触发器系统

```javascript
// 触发器系统
class TriggerSystem {
    constructor(world) {
        this.world = world;
        this.triggers = new Map();
    }

    createTrigger(position, size) {
        // 创建触发器物理体
        const shape = new CANNON.Box(new CANNON.Vec3(size.x/2, size.y/2, size.z/2));
        const body = new CANNON.Body({
            mass: 0,
            position: new CANNON.Vec3(position.x, position.y, position.z),
            shape: shape,
            isTrigger: true
        });

        // 创建触发器网格
        const geometry = new THREE.BoxGeometry(size.x, size.y, size.z);
        const material = new THREE.MeshBasicMaterial({
            color: 0x00ff00,
            transparent: true,
            opacity: 0.3
        });
        const mesh = new THREE.Mesh(geometry, material);

        // 添加到物理世界
        this.world.addBody(mesh, body);
        this.triggers.set(mesh.uuid, {
            mesh,
            body,
            callbacks: new Set()
        });

        return mesh.uuid;
    }

    addCallback(triggerId, callback) {
        const trigger = this.triggers.get(triggerId);
        if (trigger) {
            trigger.callbacks.add(callback);
        }
    }

    removeCallback(triggerId, callback) {
        const trigger = this.triggers.get(triggerId);
        if (trigger) {
            trigger.callbacks.delete(callback);
        }
    }

    checkTriggers() {
        this.triggers.forEach((trigger, uuid) => {
            const body = trigger.body;

            // 检查触发器内的物体
            this.world.bodies.forEach((otherBody, otherUuid) => {
                if (otherBody !== body) {
                    const contact = this.world.world.getContact(body, otherBody);
                    if (contact) {
                        // 触发回调
                        trigger.callbacks.forEach(callback => {
                            callback(otherBody);
                        });
                    }
                }
            });
        });
    }
}
```

## 实战：物理模拟场景

让我们创建一个完整的物理模拟场景：

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

// 创建物理世界
const physicsWorld = new PhysicsWorld();
const objectManager = new PhysicsObjectManager(physicsWorld);
const collisionManager = new CollisionManager(physicsWorld);
const triggerSystem = new TriggerSystem(physicsWorld);

// 添加光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
scene.add(directionalLight);

// 创建地面
const ground = objectManager.createPlane(
    new THREE.Vector3(20, 1, 20),
    new THREE.Vector3(0, -0.5, 0)
);
ground.mesh.receiveShadow = true;

// 创建触发器
const triggerId = triggerSystem.createTrigger(
    new THREE.Vector3(0, 2, 0),
    new THREE.Vector3(2, 2, 2)
);

// 添加触发器回调
triggerSystem.addCallback(triggerId, (body) => {
    console.log('Object entered trigger zone:', body);
});

// 添加物体
function addObject() {
    const position = new THREE.Vector3(
        (Math.random() - 0.5) * 5,
        5,
        (Math.random() - 0.5) * 5
    );

    const object = Math.random() > 0.5 ?
        objectManager.createBox(
            new THREE.Vector3(1, 1, 1),
            position
        ) :
        objectManager.createSphere(
            0.5,
            position
        );

    object.mesh.castShadow = true;
    object.mesh.receiveShadow = true;
}

// 点击添加物体
window.addEventListener('click', addObject);

// 动画循环
const clock = new THREE.Clock();
function animate() {
    requestAnimationFrame(animate);

    const deltaTime = Math.min(clock.getDelta(), 0.1);

    // 更新物理世界
    physicsWorld.update(deltaTime);

    // 检查触发器
    triggerSystem.checkTriggers();

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
```

## 练习

1. 创建基础物理世界
2. 实现碰撞检测
3. 添加触发器系统
4. 创建物理模拟场景

## 下一步学习

在下一章中，我们将学习：
- 自定义几何体
- 高级动画系统
- 跨平台适配
- 性能优化

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！