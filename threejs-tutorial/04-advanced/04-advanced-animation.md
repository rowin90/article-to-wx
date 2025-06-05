# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 高级动画系统

在 Three.js 中，我们可以创建复杂的动画系统来实现丰富的动画效果。本章将介绍如何构建高级动画系统。

## 动画系统基础

### 1. 动画管理器

![动画系统示例](https://threejs.org/examples/screenshots/webgl_animation_skinning_morph.jpg)

*图 19.1: 动画系统示例*

```javascript
// 动画管理器
class AnimationManager {
    constructor() {
        this.animations = new Map();
        this.mixers = new Map();
        this.clock = new THREE.Clock();
    }

    addAnimation(object, name, animation) {
        if (!this.mixers.has(object)) {
            this.mixers.set(object, new THREE.AnimationMixer(object));
        }

        const mixer = this.mixers.get(object);
        const action = mixer.clipAction(animation);
        this.animations.set(name, action);
    }

    play(name, options = {}) {
        const action = this.animations.get(name);
        if (action) {
            action.reset();
            action.setLoop(options.loop || THREE.LoopRepeat);
            action.clampWhenFinished = options.clampWhenFinished || false;
            action.timeScale = options.timeScale || 1;
            action.play();
        }
    }

    crossFade(fromName, toName, duration = 0.5) {
        const fromAction = this.animations.get(fromName);
        const toAction = this.animations.get(toName);

        if (fromAction && toAction) {
            fromAction.crossFadeTo(toAction, duration, true);
            toAction.play();
        }
    }

    update() {
        const delta = this.clock.getDelta();
        this.mixers.forEach(mixer => mixer.update(delta));
    }
}
```

### 2. 动画状态机

```javascript
// 动画状态机
class AnimationStateMachine {
    constructor(animationManager) {
        this.manager = animationManager;
        this.states = new Map();
        this.currentState = null;
        this.transitions = new Map();
    }

    addState(name, animation, options = {}) {
        this.states.set(name, {
            name,
            animation,
            options
        });
    }

    addTransition(fromState, toState, condition) {
        if (!this.transitions.has(fromState)) {
            this.transitions.set(fromState, new Map());
        }
        this.transitions.get(fromState).set(toState, condition);
    }

    setState(name) {
        const state = this.states.get(name);
        if (state) {
            if (this.currentState) {
                this.manager.crossFade(
                    this.currentState.name,
                    state.name,
                    0.5
                );
            } else {
                this.manager.play(state.name, state.options);
            }
            this.currentState = state;
        }
    }

    update() {
        if (this.currentState) {
            const transitions = this.transitions.get(this.currentState.name);
            if (transitions) {
                for (const [toState, condition] of transitions) {
                    if (condition()) {
                        this.setState(toState);
                        break;
                    }
                }
            }
        }
    }
}
```

## 高级动画效果

### 1. 骨骼动画

```javascript
// 骨骼动画控制器
class SkeletonAnimationController {
    constructor(skeleton) {
        this.skeleton = skeleton;
        this.bones = skeleton.bones;
        this.poses = new Map();
    }

    addPose(name, positions, rotations) {
        this.poses.set(name, {
            positions: positions,
            rotations: rotations
        });
    }

    applyPose(name, weight = 1) {
        const pose = this.poses.get(name);
        if (pose) {
            this.bones.forEach((bone, index) => {
                if (pose.positions[index]) {
                    bone.position.lerp(pose.positions[index], weight);
                }
                if (pose.rotations[index]) {
                    bone.quaternion.slerp(pose.rotations[index], weight);
                }
            });
        }
    }

    blendPoses(poseA, poseB, weight) {
        const pose1 = this.poses.get(poseA);
        const pose2 = this.poses.get(poseB);

        if (pose1 && pose2) {
            this.bones.forEach((bone, index) => {
                if (pose1.positions[index] && pose2.positions[index]) {
                    bone.position.lerpVectors(
                        pose1.positions[index],
                        pose2.positions[index],
                        weight
                    );
                }
                if (pose1.rotations[index] && pose2.rotations[index]) {
                    bone.quaternion.slerpQuaternions(
                        pose1.rotations[index],
                        pose2.rotations[index],
                        weight
                    );
                }
            });
        }
    }
}
```

### 2. 变形动画

```javascript
// 变形动画控制器
class MorphAnimationController {
    constructor(mesh) {
        this.mesh = mesh;
        this.morphTargetInfluences = mesh.morphTargetInfluences;
        this.morphTargetDictionary = mesh.morphTargetDictionary;
        this.weights = new Map();
    }

    setWeight(name, weight) {
        const index = this.morphTargetDictionary[name];
        if (index !== undefined) {
            this.weights.set(name, weight);
            this.morphTargetInfluences[index] = weight;
        }
    }

    animate(name, from, to, duration, easing = 'linear') {
        const index = this.morphTargetDictionary[name];
        if (index !== undefined) {
            const startTime = Date.now();
            const animate = () => {
                const elapsed = Date.now() - startTime;
                const progress = Math.min(elapsed / duration, 1);
                const easedProgress = this.ease(easing, progress);
                const weight = from + (to - from) * easedProgress;

                this.setWeight(name, weight);

                if (progress < 1) {
                    requestAnimationFrame(animate);
                }
            };
            animate();
        }
    }

    ease(type, t) {
        switch (type) {
            case 'linear':
                return t;
            case 'easeInQuad':
                return t * t;
            case 'easeOutQuad':
                return t * (2 - t);
            case 'easeInOutQuad':
                return t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
            default:
                return t;
        }
    }
}
```

## 实战：创建高级动画系统

让我们创建一个完整的动画系统示例：

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

// 添加光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
scene.add(directionalLight);

// 创建动画管理器
const animationManager = new AnimationManager();
const stateMachine = new AnimationStateMachine(animationManager);

// 创建角色
const geometry = new THREE.BoxGeometry(1, 2, 1);
const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 });
const character = new THREE.Mesh(geometry, material);
character.castShadow = true;
scene.add(character);

// 创建动画
const idleAnimation = new THREE.AnimationClip('idle', 1, [
    new THREE.KeyframeTrack(
        '.position[y]',
        [0, 0.5, 1],
        [0, 0.1, 0]
    )
]);

const walkAnimation = new THREE.AnimationClip('walk', 1, [
    new THREE.KeyframeTrack(
        '.position[y]',
        [0, 0.25, 0.5, 0.75, 1],
        [0, 0.1, 0, 0.1, 0]
    ),
    new THREE.KeyframeTrack(
        '.rotation[y]',
        [0, 0.5, 1],
        [0, Math.PI, Math.PI * 2]
    )
]);

// 添加动画
animationManager.addAnimation(character, 'idle', idleAnimation);
animationManager.addAnimation(character, 'walk', walkAnimation);

// 添加状态
stateMachine.addState('idle', 'idle', { loop: THREE.LoopRepeat });
stateMachine.addState('walk', 'walk', { loop: THREE.LoopRepeat });

// 添加状态转换
stateMachine.addTransition('idle', 'walk', () => {
    return Math.random() > 0.7;
});

stateMachine.addTransition('walk', 'idle', () => {
    return Math.random() > 0.7;
});

// 设置初始状态
stateMachine.setState('idle');

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新动画
    animationManager.update();
    stateMachine.update();

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

1. 创建基础动画系统
2. 实现状态机
3. 添加骨骼动画
4. 创建变形动画

## 下一步学习

在下一章中，我们将学习：
- 跨平台适配
- 性能优化
- 最佳实践
- 项目部署

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！