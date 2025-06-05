# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# Shader 编程入门

在 Three.js 中，Shader 编程是创建高级视觉效果的关键。本章将介绍 Shader 的基础知识和在 Three.js 中的应用。

## Shader 基础

### 1. 顶点着色器

![顶点着色器示例](https://threejs.org/examples/screenshots/webgl_geometry_shapes.jpg)

*图 16.1: 顶点着色器示例*

```javascript
// 基础顶点着色器
const vertexShader = `
    varying vec2 vUv;
    varying vec3 vPosition;

    void main() {
        vUv = uv;
        vPosition = position;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
`;

// 使用顶点着色器
const material = new THREE.ShaderMaterial({
    vertexShader: vertexShader,
    fragmentShader: fragmentShader,
    uniforms: {
        time: { value: 0 }
    }
});
```

### 2. 片元着色器

```javascript
// 基础片元着色器
const fragmentShader = `
    uniform float time;
    varying vec2 vUv;
    varying vec3 vPosition;

    void main() {
        // 创建波浪效果
        float wave = sin(vUv.x * 10.0 + time) * 0.5 + 0.5;

        // 设置颜色
        vec3 color = vec3(wave, wave * 0.5, 1.0);

        gl_FragColor = vec4(color, 1.0);
    }
`;
```

## 自定义着色器材质

### 1. 基础着色器材质

```javascript
// 自定义着色器材质
class CustomShaderMaterial {
    constructor() {
        this.uniforms = {
            time: { value: 0 },
            color: { value: new THREE.Color(0xff0000) },
            texture: { value: null }
        };

        this.vertexShader = `
            varying vec2 vUv;
            varying vec3 vNormal;
            varying vec3 vPosition;

            void main() {
                vUv = uv;
                vNormal = normal;
                vPosition = position;

                // 添加波浪效果
                vec3 newPosition = position;
                newPosition.y += sin(position.x * 2.0 + time) * 0.1;

                gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0);
            }
        `;

        this.fragmentShader = `
            uniform float time;
            uniform vec3 color;
            uniform sampler2D texture;

            varying vec2 vUv;
            varying vec3 vNormal;
            varying vec3 vPosition;

            void main() {
                // 基础颜色
                vec3 baseColor = color;

                // 添加纹理
                if (texture != null) {
                    baseColor *= texture2D(texture, vUv).rgb;
                }

                // 添加光照效果
                float light = dot(vNormal, normalize(vec3(1.0, 1.0, 1.0)));
                baseColor *= light;

                gl_FragColor = vec4(baseColor, 1.0);
            }
        `;
    }

    createMaterial() {
        return new THREE.ShaderMaterial({
            uniforms: this.uniforms,
            vertexShader: this.vertexShader,
            fragmentShader: this.fragmentShader,
            side: THREE.DoubleSide
        });
    }

    update(time) {
        this.uniforms.time.value = time;
    }
}
```

### 2. 高级着色器效果

```javascript
// 高级着色器效果
class AdvancedShaderEffects {
    constructor() {
        this.uniforms = {
            time: { value: 0 },
            resolution: { value: new THREE.Vector2() },
            mouse: { value: new THREE.Vector2() }
        };

        this.vertexShader = `
            varying vec2 vUv;
            varying vec3 vPosition;
            varying vec3 vNormal;

            void main() {
                vUv = uv;
                vPosition = position;
                vNormal = normal;

                // 添加扭曲效果
                vec3 newPosition = position;
                newPosition.x += sin(position.y * 5.0 + time) * 0.1;
                newPosition.y += cos(position.x * 5.0 + time) * 0.1;

                gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0);
            }
        `;

        this.fragmentShader = `
            uniform float time;
            uniform vec2 resolution;
            uniform vec2 mouse;

            varying vec2 vUv;
            varying vec3 vPosition;
            varying vec3 vNormal;

            // 噪声函数
            float noise(vec2 p) {
                return fract(sin(dot(p, vec2(12.9898, 78.233))) * 43758.5453);
            }

            void main() {
                // 创建动态纹理
                vec2 uv = vUv;
                uv.x += sin(uv.y * 10.0 + time) * 0.1;
                uv.y += cos(uv.x * 10.0 + time) * 0.1;

                // 添加噪声
                float n = noise(uv * 10.0 + time);

                // 创建颜色
                vec3 color = vec3(n, n * 0.5, 1.0);

                // 添加鼠标交互
                float dist = length(vPosition.xy - mouse);
                color += vec3(1.0 - smoothstep(0.0, 1.0, dist));

                gl_FragColor = vec4(color, 1.0);
            }
        `;
    }

    createMaterial() {
        return new THREE.ShaderMaterial({
            uniforms: this.uniforms,
            vertexShader: this.vertexShader,
            fragmentShader: this.fragmentShader,
            side: THREE.DoubleSide
        });
    }

    update(time, mouse) {
        this.uniforms.time.value = time;
        this.uniforms.mouse.value.copy(mouse);
    }
}
```

## 实战：创建着色器效果

让我们创建一个完整的着色器效果示例：

```javascript
// 创建场景
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.z = 5;

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 创建着色器效果
const shaderEffect = new AdvancedShaderEffects();
const material = shaderEffect.createMaterial();

// 创建几何体
const geometry = new THREE.PlaneGeometry(5, 5, 32, 32);
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

// 添加鼠标交互
const mouse = new THREE.Vector2();
window.addEventListener('mousemove', (event) => {
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
});

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新着色器
    shaderEffect.update(performance.now() * 0.001, mouse);

    // 渲染场景
    renderer.render(scene, camera);
}

animate();

// 窗口大小改变时更新
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
    shaderEffect.uniforms.resolution.value.set(window.innerWidth, window.innerHeight);
});
```

## 练习

1. 创建基础顶点着色器
2. 实现片元着色器效果
3. 添加动态纹理
4. 实现鼠标交互效果

## 下一步学习

在下一章中，我们将学习：
- 物理引擎集成
- 碰撞检测
- 物理模拟
- 粒子系统

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！