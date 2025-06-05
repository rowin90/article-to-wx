# 模型加载与动画

在 Three.js 中，模型加载和动画是创建复杂 3D 场景的重要组成部分。本章将介绍如何加载和管理 3D 模型，以及如何实现和控制动画效果。

## 模型加载

### 1. 基础模型加载

![模型加载示例](https://threejs.org/examples/screenshots/webgl_loader_gltf.jpg)

*图 10.1: GLTF 模型加载示例*

```javascript
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';

// 创建加载器
const loader = new GLTFLoader();

// 配置 DRACO 加载器（用于压缩模型）
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('/path/to/draco/');
loader.setDRACOLoader(dracoLoader);

// 加载模型
loader.load(
    'model.gltf',
    (gltf) => {
        const model = gltf.scene;
        scene.add(model);
    },
    (xhr) => {
        console.log((xhr.loaded / xhr.total * 100) + '% loaded');
    },
    (error) => {
        console.error('An error happened:', error);
    }
);
```

### 2. 模型管理器

```javascript
// 模型管理器
class ModelManager {
    constructor() {
        this.loader = new GLTFLoader();
        this.models = new Map();
        this.loadingManager = new THREE.LoadingManager();
        this.setupLoadingManager();
    }

    setupLoadingManager() {
        this.loadingManager.onProgress = (url, itemsLoaded, itemsTotal) => {
            console.log(`Loading: ${itemsLoaded}/${itemsTotal}`);
        };

        this.loadingManager.onError = (url) => {
            console.error('Error loading:', url);
        };
    }

    loadModel(name, url) {
        return new Promise((resolve, reject) => {
            this.loader.load(
                url,
                (gltf) => {
                    this.models.set(name, gltf);
                    resolve(gltf);
                },
                undefined,
                (error) => reject(error)
            );
        });
    }

    getModel(name) {
        return this.models.get(name);
    }

    dispose() {
        this.models.forEach(model => {
            model.scene.traverse((object) => {
                if (object.isMesh) {
                    object.geometry.dispose();
                    object.material.dispose();
                }
            });
        });
        this.models.clear();
    }
}
```

## 骨骼动画

### 1. 基础骨骼动画

```javascript
// 骨骼动画控制器
class SkeletonAnimation {
    constructor(model) {
        this.model = model;
        this.mixer = new THREE.AnimationMixer(model);
        this.actions = new Map();
        this.currentAction = null;
        this.init();
    }

    init() {
        // 获取所有动画
        this.model.animations.forEach(clip => {
            const action = this.mixer.clipAction(clip);
            this.actions.set(clip.name, action);
        });
    }

    play(name, options = {}) {
        const action = this.actions.get(name);
        if (!action) return;

        // 停止当前动画
        if (this.currentAction) {
            this.currentAction.fadeOut(0.5);
        }

        // 设置新动画
        action.reset();
        action.fadeIn(0.5);
        action.play();

        // 设置动画参数
        if (options.loop !== undefined) {
            action.loop = options.loop;
        }
        if (options.timeScale !== undefined) {
            action.timeScale = options.timeScale;
        }

        this.currentAction = action;
    }

    update(deltaTime) {
        this.mixer.update(deltaTime);
    }
}
```

### 2. 动画混合

```javascript
// 动画混合器
class AnimationBlender {
    constructor(model) {
        this.model = model;
        this.mixer = new THREE.AnimationMixer(model);
        this.actions = new Map();
        this.activeActions = new Set();
    }

    addAction(name, clip, weight = 1.0) {
        const action = this.mixer.clipAction(clip);
        action.weight = weight;
        this.actions.set(name, action);
    }

    blend(name, weight, duration = 0.5) {
        const action = this.actions.get(name);
        if (!action) return;

        // 淡入新动画
        action.reset();
        action.fadeIn(duration);
        action.play();

        // 淡出其他动画
        this.activeActions.forEach(activeAction => {
            if (activeAction !== action) {
                activeAction.fadeOut(duration);
            }
        });

        this.activeActions.add(action);
    }

    update(deltaTime) {
        this.mixer.update(deltaTime);
    }
}
```

## 动画控制

### 1. 动画状态机

```javascript
// 动画状态机
class AnimationStateMachine {
    constructor(model) {
        this.model = model;
        this.blender = new AnimationBlender(model);
        this.states = new Map();
        this.currentState = null;
        this.transitions = new Map();
    }

    addState(name, clip, weight = 1.0) {
        this.blender.addAction(name, clip, weight);
        this.states.set(name, { clip, weight });
    }

    addTransition(from, to, condition) {
        if (!this.transitions.has(from)) {
            this.transitions.set(from, new Map());
        }
        this.transitions.get(from).set(to, condition);
    }

    update(deltaTime, input) {
        // 检查状态转换
        if (this.currentState) {
            const transitions = this.transitions.get(this.currentState);
            if (transitions) {
                for (const [nextState, condition] of transitions) {
                    if (condition(input)) {
                        this.transitionTo(nextState);
                        break;
                    }
                }
            }
        }

        // 更新动画
        this.blender.update(deltaTime);
    }

    transitionTo(state) {
        if (this.states.has(state)) {
            this.currentState = state;
            this.blender.blend(state, this.states.get(state).weight);
        }
    }
}
```

### 2. 动画事件系统

```javascript
// 动画事件系统
class AnimationEventSystem {
    constructor() {
        this.events = new Map();
    }

    addEvent(name, time, callback) {
        if (!this.events.has(name)) {
            this.events.set(name, new Map());
        }
        this.events.get(name).set(time, callback);
    }

    checkEvents(mixer, clip, time) {
        const events = this.events.get(clip.name);
        if (!events) return;

        events.forEach((callback, eventTime) => {
            if (Math.abs(time - eventTime) < 0.1) {
                callback();
            }
        });
    }
}
```

## 实战：创建一个动画场景

让我们创建一个展示模型加载和动画效果的场景：

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
camera.position.set(0, 2, 5);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

// 创建模型管理器
const modelManager = new ModelManager();

// 创建动画状态机
let animationStateMachine;

// 加载模型
modelManager.loadModel('character', 'character.gltf').then(gltf => {
    const model = gltf.scene;
    model.scale.set(1, 1, 1);
    model.position.set(0, 0, 0);
    scene.add(model);

    // 设置动画状态机
    animationStateMachine = new AnimationStateMachine(model);

    // 添加动画状态
    gltf.animations.forEach(clip => {
        animationStateMachine.addState(clip.name, clip);
    });

    // 添加状态转换
    animationStateMachine.addTransition('idle', 'walk', () => {
        return input.keys.has('w') || input.keys.has('a') ||
               input.keys.has('s') || input.keys.has('d');
    });

    animationStateMachine.addTransition('walk', 'idle', () => {
        return !input.keys.has('w') && !input.keys.has('a') &&
               !input.keys.has('s') && !input.keys.has('d');
    });

    // 设置初始状态
    animationStateMachine.transitionTo('idle');
});

// 创建输入控制器
const input = {
    keys: new Set(),
    init() {
        window.addEventListener('keydown', (e) => this.keys.add(e.key));
        window.addEventListener('keyup', (e) => this.keys.delete(e.key));
    }
};
input.init();

// 添加光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1.0);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
scene.add(directionalLight);

// 动画循环
const clock = new THREE.Clock();
function animate() {
    requestAnimationFrame(animate);

    const deltaTime = clock.getDelta();

    // 更新动画状态机
    if (animationStateMachine) {
        animationStateMachine.update(deltaTime, input);
    }

    renderer.render(scene, camera);
}

animate();
```

## 练习

1. 加载一个 3D 模型
2. 实现基本的骨骼动画
3. 创建动画状态机
4. 添加动画事件系统

## 下一步学习

在下一章中，我们将学习：
- 3D 产品展示
- 模型交互
- 相机控制
- 场景优化
