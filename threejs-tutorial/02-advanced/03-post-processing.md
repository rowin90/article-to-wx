# 后期处理与滤镜

在 Three.js 中，后期处理（Post-processing）是一个强大的功能，它允许我们在场景渲染完成后对画面进行额外的处理，从而创建各种视觉效果。

## 基础后处理

### 1. EffectComposer 基础

![后处理效果示例](https://threejs.org/examples/screenshots/webgl_postprocessing_unreal_bloom.jpg)

*图 8.1: 后处理效果示例*

```javascript
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';

// 创建后处理器
const composer = new EffectComposer(renderer);

// 添加渲染通道
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// 在动画循环中使用后处理器
function animate() {
    requestAnimationFrame(animate);
    composer.render();
}
```

### 2. 基础滤镜

```javascript
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader.js';

// 添加抗锯齿
const fxaaPass = new ShaderPass(FXAAShader);
fxaaPass.uniforms['resolution'].value.set(
    1 / (window.innerWidth * renderer.getPixelRatio()),
    1 / (window.innerHeight * renderer.getPixelRatio())
);
composer.addPass(fxaaPass);
```

## 高级后处理效果

### 1. 泛光效果（Bloom）

```javascript
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';

// 创建泛光效果
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    1.5,  // 强度
    0.4,  // 半径
    0.85  // 阈值
);

// 添加到后处理器
composer.addPass(bloomPass);
```

### 2. 景深效果（Depth of Field）

```javascript
import { BokehPass } from 'three/examples/jsm/postprocessing/BokehPass.js';

// 创建景深效果
const bokehPass = new BokehPass(
    scene,
    camera,
    {
        focus: 10.0,      // 焦距
        aperture: 0.00002, // 光圈
        maxblur: 1.0      // 最大模糊
    }
);

// 添加到后处理器
composer.addPass(bokehPass);
```

### 3. 色彩调整

```javascript
import { ColorCorrectionShader } from 'three/examples/jsm/shaders/ColorCorrectionShader.js';

// 创建色彩调整通道
const colorPass = new ShaderPass(ColorCorrectionShader);
colorPass.uniforms.powRGB.value = new THREE.Vector3(1.2, 1.2, 1.2);
colorPass.uniforms.mulRGB.value = new THREE.Vector3(1.1, 1.1, 1.1);

// 添加到后处理器
composer.addPass(colorPass);
```

## 自定义后处理

### 1. 自定义着色器通道

```javascript
// 创建自定义着色器
const customShader = {
    uniforms: {
        tDiffuse: { value: null },
        time: { value: 0 }
    },
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    fragmentShader: `
        uniform sampler2D tDiffuse;
        uniform float time;
        varying vec2 vUv;

        void main() {
            vec2 uv = vUv;
            uv.x += sin(uv.y * 10.0 + time) * 0.01;
            gl_FragColor = texture2D(tDiffuse, uv);
        }
    `
};

// 创建自定义通道
const customPass = new ShaderPass(customShader);

// 在动画循环中更新
function animate() {
    requestAnimationFrame(animate);
    customPass.uniforms.time.value += 0.01;
    composer.render();
}
```

### 2. 多通道组合

```javascript
// 创建多个后处理通道
const passes = [
    new RenderPass(scene, camera),
    new UnrealBloomPass(
        new THREE.Vector2(window.innerWidth, window.innerHeight),
        1.5, 0.4, 0.85
    ),
    new BokehPass(scene, camera, {
        focus: 10.0,
        aperture: 0.00002,
        maxblur: 1.0
    }),
    new ShaderPass(ColorCorrectionShader)
];

// 添加到后处理器
passes.forEach(pass => composer.addPass(pass));
```

## 性能优化

### 1. 通道管理

```javascript
class PostProcessingManager {
    constructor(renderer, scene, camera) {
        this.composer = new EffectComposer(renderer);
        this.passes = new Map();
        this.init(scene, camera);
    }

    init(scene, camera) {
        // 添加基础渲染通道
        this.addPass('render', new RenderPass(scene, camera));
    }

    addPass(name, pass) {
        this.passes.set(name, pass);
        this.composer.addPass(pass);
    }

    removePass(name) {
        const pass = this.passes.get(name);
        if (pass) {
            this.composer.removePass(pass);
            this.passes.delete(name);
        }
    }

    update() {
        this.composer.render();
    }
}
```

### 2. 动态质量调整

```javascript
class AdaptivePostProcessing {
    constructor(composer) {
        this.composer = composer;
        this.quality = 'high';
        this.fps = 60;
        this.lastTime = performance.now();
        this.frameCount = 0;
    }

    update() {
        // 计算 FPS
        const now = performance.now();
        this.frameCount++;

        if (now - this.lastTime >= 1000) {
            this.fps = this.frameCount;
            this.frameCount = 0;
            this.lastTime = now;

            // 根据 FPS 调整质量
            this.adjustQuality();
        }
    }

    adjustQuality() {
        if (this.fps < 30 && this.quality === 'high') {
            this.setQuality('medium');
        } else if (this.fps < 20 && this.quality === 'medium') {
            this.setQuality('low');
        } else if (this.fps > 50 && this.quality === 'low') {
            this.setQuality('medium');
        } else if (this.fps > 55 && this.quality === 'medium') {
            this.setQuality('high');
        }
    }

    setQuality(quality) {
        this.quality = quality;
        // 根据质量设置调整后处理参数
        switch (quality) {
            case 'high':
                this.setHighQuality();
                break;
            case 'medium':
                this.setMediumQuality();
                break;
            case 'low':
                this.setLowQuality();
                break;
        }
    }
}
```

## 实战：创建一个后处理场景

让我们创建一个展示各种后处理效果的场景：

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
camera.position.set(0, 0, 5);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 创建后处理器
const composer = new EffectComposer(renderer);

// 添加基础渲染通道
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// 添加泛光效果
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    1.5,
    0.4,
    0.85
);
composer.addPass(bloomPass);

// 添加景深效果
const bokehPass = new BokehPass(
    scene,
    camera,
    {
        focus: 5.0,
        aperture: 0.00002,
        maxblur: 1.0
    }
);
composer.addPass(bokehPass);

// 创建物体
const geometry = new THREE.TorusKnotGeometry(1, 0.3, 100, 16);
const material = new THREE.MeshStandardMaterial({
    color: 0x00ff00,
    metalness: 0.5,
    roughness: 0.5,
    emissive: 0x00ff00,
    emissiveIntensity: 0.5
});
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

// 添加光源
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1.0);
directionalLight.position.set(5, 5, 5);
scene.add(directionalLight);

// 创建后处理管理器
const postProcessingManager = new PostProcessingManager(renderer, scene, camera);
const adaptivePostProcessing = new AdaptivePostProcessing(composer);

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新物体
    mesh.rotation.x += 0.01;
    mesh.rotation.y += 0.01;

    // 更新后处理
    adaptivePostProcessing.update();
    postProcessingManager.update();
}

animate();
```

## 练习

1. 创建一个基础的后处理场景
2. 实现多种后处理效果的组合
3. 添加自定义后处理效果
4. 实现后处理性能优化

## 下一步学习

在下一章中，我们将学习：
- 性能优化技巧
- 渲染优化
- 内存管理
- 调试工具
