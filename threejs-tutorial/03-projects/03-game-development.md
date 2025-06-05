# 游戏开发基础

在 Three.js 中，我们可以创建各种类型的 3D 游戏。本章将介绍游戏开发的基础知识，包括物理引擎集成、碰撞检测、游戏逻辑实现等。

## 物理引擎集成

### 1. Cannon.js 基础

![物理引擎示例](https://threejs.org/examples/screenshots/physics_ammo_break.jpg)

*图 13.1: 物理引擎示例*

```javascript
// 物理世界管理器
class PhysicsWorld {
    constructor() {
        this.world = new CANNON.World();
        this.world.gravity.set(0, -9.82, 0);
        this.world.broadphase = new CANNON.NaiveBroadphase();
        this.world.solver.iterations = 10;

        this.bodies = new Map();
        this.setupContactMaterial();
    }

    setupContactMaterial() {
        const defaultMaterial = new CANNON.Material('default');
        const defaultContactMaterial = new CANNON.ContactMaterial(
            defaultMaterial,
            defaultMaterial,
            {
                friction: 0.3,
                restitution: 0.3
            }
        );
        this.world.addContactMaterial(defaultContactMaterial);
        this.world.defaultContactMaterial = defaultContactMaterial;
    }

    addBody(mesh, mass = 0) {
        const shape = this.createShape(mesh);
        const body = new CANNON.Body({
            mass,
            shape,
            material: this.world.defaultMaterial
        });

        body.position.copy(mesh.position);
        body.quaternion.copy(mesh.quaternion);

        this.world.addBody(body);
        this.bodies.set(mesh.uuid, { body, mesh });

        return body;
    }

    createShape(mesh) {
        if (mesh.geometry instanceof THREE.BoxGeometry) {
            const size = mesh.geometry.parameters;
            return new CANNON.Box(new CANNON.Vec3(
                size.width / 2,
                size.height / 2,
                size.depth / 2
            ));
        } else if (mesh.geometry instanceof THREE.SphereGeometry) {
            return new CANNON.Sphere(mesh.geometry.parameters.radius);
        }
        return null;
    }

    update() {
        this.world.step(1/60);

        this.bodies.forEach(({ body, mesh }) => {
            mesh.position.copy(body.position);
            mesh.quaternion.copy(body.quaternion);
        });
    }
}
```

### 2. 物理对象

```javascript
// 物理对象基类
class PhysicsObject {
    constructor(scene, physicsWorld, options = {}) {
        this.scene = scene;
        this.physicsWorld = physicsWorld;
        this.options = options;

        this.mesh = null;
        this.body = null;
        this.create();
    }

    create() {
        // 由子类实现
    }

    update() {
        // 由子类实现
    }

    applyForce(force, point) {
        if (this.body) {
            this.body.applyForce(force, point);
        }
    }

    applyImpulse(impulse, point) {
        if (this.body) {
            this.body.applyImpulse(impulse, point);
        }
    }
}

// 球体对象
class PhysicsSphere extends PhysicsObject {
    create() {
        const { radius = 1, color = 0xff0000 } = this.options;

        // 创建网格
        const geometry = new THREE.SphereGeometry(radius);
        const material = new THREE.MeshPhongMaterial({ color });
        this.mesh = new THREE.Mesh(geometry, material);
        this.scene.add(this.mesh);

        // 创建物理体
        this.body = this.physicsWorld.addBody(this.mesh, 1);
    }
}

// 盒子对象
class PhysicsBox extends PhysicsObject {
    create() {
        const { width = 1, height = 1, depth = 1, color = 0x00ff00 } = this.options;

        // 创建网格
        const geometry = new THREE.BoxGeometry(width, height, depth);
        const material = new THREE.MeshPhongMaterial({ color });
        this.mesh = new THREE.Mesh(geometry, material);
        this.scene.add(this.mesh);

        // 创建物理体
        this.body = this.physicsWorld.addBody(this.mesh, 1);
    }
}
```

## 碰撞检测

### 1. 碰撞管理器

```javascript
// 碰撞管理器
class CollisionManager {
    constructor(physicsWorld) {
        this.physicsWorld = physicsWorld;
        this.collisionCallbacks = new Map();
        this.setupCollisionEvents();
    }

    setupCollisionEvents() {
        this.physicsWorld.world.addEventListener('beginContact', (event) => {
            const bodyA = event.bodyA;
            const bodyB = event.bodyB;

            this.handleCollision(bodyA, bodyB, 'begin');
        });

        this.physicsWorld.world.addEventListener('endContact', (event) => {
            const bodyA = event.bodyA;
            const bodyB = event.bodyB;

            this.handleCollision(bodyA, bodyB, 'end');
        });
    }

    handleCollision(bodyA, bodyB, type) {
        const callback = this.collisionCallbacks.get(`${bodyA.id}-${bodyB.id}`);
        if (callback) {
            callback(type, bodyA, bodyB);
        }
    }

    onCollision(bodyA, bodyB, callback) {
        this.collisionCallbacks.set(`${bodyA.id}-${bodyB.id}`, callback);
    }
}
```

### 2. 触发器系统

```javascript
// 触发器系统
class TriggerSystem {
    constructor(scene, physicsWorld) {
        this.scene = scene;
        this.physicsWorld = physicsWorld;
        this.triggers = new Map();
    }

    createTrigger(position, size, callback) {
        const geometry = new THREE.BoxGeometry(size.x, size.y, size.z);
        const material = new THREE.MeshBasicMaterial({
            color: 0xffff00,
            transparent: true,
            opacity: 0.3
        });

        const mesh = new THREE.Mesh(geometry, material);
        mesh.position.copy(position);
        this.scene.add(mesh);

        const body = this.physicsWorld.addBody(mesh, 0);
        body.isTrigger = true;

        this.triggers.set(mesh.uuid, { mesh, body, callback });

        return mesh;
    }

    checkTriggers(object) {
        this.triggers.forEach(({ body, callback }) => {
            if (this.physicsWorld.world.broadphase.aabbOverlap(body, object.body)) {
                callback(object);
            }
        });
    }
}
```

## 游戏逻辑

### 1. 游戏管理器

```javascript
// 游戏管理器
class GameManager {
    constructor(scene, physicsWorld) {
        this.scene = scene;
        this.physicsWorld = physicsWorld;
        this.collisionManager = new CollisionManager(physicsWorld);
        this.triggerSystem = new TriggerSystem(scene, physicsWorld);

        this.objects = new Map();
        this.score = 0;
        this.gameOver = false;
    }

    addObject(object) {
        this.objects.set(object.mesh.uuid, object);
        return object;
    }

    removeObject(object) {
        this.objects.delete(object.mesh.uuid);
        this.scene.remove(object.mesh);
        this.physicsWorld.world.removeBody(object.body);
    }

    update() {
        if (this.gameOver) return;

        this.objects.forEach(object => {
            object.update();
            this.triggerSystem.checkTriggers(object);
        });

        this.checkGameState();
    }

    checkGameState() {
        // 由子类实现
    }

    endGame() {
        this.gameOver = true;
        console.log('Game Over! Score:', this.score);
    }
}
```

### 2. 玩家控制器

```javascript
// 玩家控制器
class PlayerController {
    constructor(player, options = {}) {
        this.player = player;
        this.options = {
            moveSpeed: options.moveSpeed || 5,
            jumpForce: options.jumpForce || 10,
            ...options
        };

        this.keys = new Set();
        this.setupControls();
    }

    setupControls() {
        window.addEventListener('keydown', (e) => {
            this.keys.add(e.key);
        });

        window.addEventListener('keyup', (e) => {
            this.keys.delete(e.key);
        });
    }

    update() {
        const moveForce = new CANNON.Vec3();

        if (this.keys.has('w')) moveForce.z = -this.options.moveSpeed;
        if (this.keys.has('s')) moveForce.z = this.options.moveSpeed;
        if (this.keys.has('a')) moveForce.x = -this.options.moveSpeed;
        if (this.keys.has('d')) moveForce.x = this.options.moveSpeed;

        this.player.applyForce(moveForce, this.player.body.position);

        if (this.keys.has(' ') && this.isGrounded()) {
            this.player.applyImpulse(
                new CANNON.Vec3(0, this.options.jumpForce, 0),
                this.player.body.position
            );
        }
    }

    isGrounded() {
        const ray = new CANNON.Ray(
            this.player.body.position,
            new CANNON.Vec3(0, -1, 0)
        );
        const result = new CANNON.RaycastResult();

        ray.intersectWorld(this.player.physicsWorld.world, result);
        return result.hasHit && result.distance < 1.1;
    }
}
```

## 实战：创建简单游戏

让我们创建一个简单的 3D 平台游戏：

```javascript
// 创建场景
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

// 创建相机
const camera = new THREE.PerspectiveCamera(
    75,
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

// 创建物理世界
const physicsWorld = new PhysicsWorld();

// 创建游戏管理器
const gameManager = new GameManager(scene, physicsWorld);

// 创建玩家
const player = new PhysicsSphere(scene, physicsWorld, {
    radius: 0.5,
    color: 0xff0000
});
gameManager.addObject(player);

// 创建玩家控制器
const playerController = new PlayerController(player);

// 创建平台
const platform = new PhysicsBox(scene, physicsWorld, {
    width: 10,
    height: 1,
    depth: 10,
    color: 0x00ff00
});
platform.mesh.position.y = -2;
gameManager.addObject(platform);

// 创建触发器
gameManager.triggerSystem.createTrigger(
    new THREE.Vector3(0, 0, -5),
    new THREE.Vector3(2, 2, 2),
    (object) => {
        if (object === player) {
            gameManager.score += 100;
            console.log('Score:', gameManager.score);
        }
    }
);

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新物理世界
    physicsWorld.update();

    // 更新玩家控制器
    playerController.update();

    // 更新游戏管理器
    gameManager.update();

    // 更新相机位置
    camera.position.x = player.mesh.position.x;
    camera.position.z = player.mesh.position.z + 5;
    camera.lookAt(player.mesh.position);

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

1. 创建一个基础的物理场景
2. 实现玩家移动和跳跃
3. 添加碰撞检测和触发器
4. 实现简单的游戏逻辑

## 下一步学习

在下一章中，我们将学习：
- VR/AR 应用开发
- 设备交互
- 空间定位
- 手势识别
