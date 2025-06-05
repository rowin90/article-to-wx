# 前言
大家好，我是鲫小鱼。是一名`不写前端代码`的前端工程师，热衷于分享非前端的知识，带领切图仔逃离切图圈子，欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！

# 自定义几何体

在 Three.js 中，我们可以通过自定义几何体来创建独特的 3D 形状。本章将介绍如何创建和使用自定义几何体。

## 几何体基础

### 1. 顶点和面

![自定义几何体示例](https://threejs.org/examples/screenshots/webgl_geometry_shapes.jpg)

*图 18.1: 自定义几何体示例*

```javascript
// 基础几何体创建器
class GeometryCreator {
    constructor() {
        this.vertices = [];
        this.faces = [];
        this.uvs = [];
        this.normals = [];
    }

    addVertex(x, y, z) {
        this.vertices.push(x, y, z);
        return this.vertices.length / 3 - 1;
    }

    addFace(a, b, c) {
        this.faces.push(a, b, c);
    }

    addUV(u, v) {
        this.uvs.push(u, v);
    }

    addNormal(x, y, z) {
        this.normals.push(x, y, z);
    }

    createGeometry() {
        const geometry = new THREE.BufferGeometry();

        // 设置顶点
        geometry.setAttribute(
            'position',
            new THREE.Float32BufferAttribute(this.vertices, 3)
        );

        // 设置面
        geometry.setIndex(this.faces);

        // 设置UV
        if (this.uvs.length > 0) {
            geometry.setAttribute(
                'uv',
                new THREE.Float32BufferAttribute(this.uvs, 2)
            );
        }

        // 设置法线
        if (this.normals.length > 0) {
            geometry.setAttribute(
                'normal',
                new THREE.Float32BufferAttribute(this.normals, 3)
            );
        } else {
            geometry.computeVertexNormals();
        }

        return geometry;
    }
}
```

### 2. 参数化几何体

```javascript
// 参数化几何体生成器
class ParametricGeometryGenerator {
    constructor() {
        this.creator = new GeometryCreator();
    }

    createCylinder(radius, height, segments) {
        // 创建顶点
        for (let i = 0; i <= segments; i++) {
            const angle = (i / segments) * Math.PI * 2;
            const x = Math.cos(angle) * radius;
            const z = Math.sin(angle) * radius;

            // 底部顶点
            this.creator.addVertex(x, 0, z);
            this.creator.addUV(i / segments, 0);

            // 顶部顶点
            this.creator.addVertex(x, height, z);
            this.creator.addUV(i / segments, 1);
        }

        // 创建面
        for (let i = 0; i < segments; i++) {
            const a = i * 2;
            const b = i * 2 + 1;
            const c = (i + 1) * 2;
            const d = (i + 1) * 2 + 1;

            // 侧面
            this.creator.addFace(a, b, c);
            this.creator.addFace(b, d, c);

            // 底部
            this.creator.addFace(0, a, c);

            // 顶部
            this.creator.addFace(1, d, b);
        }

        return this.creator.createGeometry();
    }

    createTorus(radius, tube, radialSegments, tubularSegments) {
        // 创建顶点
        for (let i = 0; i <= radialSegments; i++) {
            const u = i / radialSegments * Math.PI * 2;

            for (let j = 0; j <= tubularSegments; j++) {
                const v = j / tubularSegments * Math.PI * 2;

                const x = (radius + tube * Math.cos(v)) * Math.cos(u);
                const y = (radius + tube * Math.cos(v)) * Math.sin(u);
                const z = tube * Math.sin(v);

                this.creator.addVertex(x, y, z);
                this.creator.addUV(i / radialSegments, j / tubularSegments);
            }
        }

        // 创建面
        for (let i = 0; i < radialSegments; i++) {
            for (let j = 0; j < tubularSegments; j++) {
                const a = i * (tubularSegments + 1) + j;
                const b = a + 1;
                const c = (i + 1) * (tubularSegments + 1) + j;
                const d = c + 1;

                this.creator.addFace(a, b, c);
                this.creator.addFace(b, d, c);
            }
        }

        return this.creator.createGeometry();
    }
}
```

## 高级几何体

### 1. 变形几何体

```javascript
// 变形几何体
class MorphGeometry {
    constructor(baseGeometry) {
        this.geometry = baseGeometry;
        this.morphTargets = [];
        this.morphTargetInfluences = [];
    }

    addMorphTarget(name, vertices) {
        const morphTarget = {
            name: name,
            vertices: vertices
        };

        this.morphTargets.push(morphTarget);
        this.morphTargetInfluences.push(0);

        return this.morphTargets.length - 1;
    }

    setMorphTargetInfluence(index, influence) {
        this.morphTargetInfluences[index] = influence;
    }

    updateGeometry() {
        const position = this.geometry.attributes.position;
        const baseVertices = position.array;

        // 重置顶点位置
        for (let i = 0; i < baseVertices.length; i++) {
            position.array[i] = baseVertices[i];
        }

        // 应用变形目标
        this.morphTargets.forEach((target, index) => {
            const influence = this.morphTargetInfluences[index];
            if (influence > 0) {
                const targetVertices = target.vertices;
                for (let i = 0; i < targetVertices.length; i++) {
                    position.array[i] += (targetVertices[i] - baseVertices[i]) * influence;
                }
            }
        });

        position.needsUpdate = true;
        this.geometry.computeVertexNormals();
    }
}
```

### 2. 程序化几何体

```javascript
// 程序化几何体生成器
class ProceduralGeometryGenerator {
    constructor() {
        this.creator = new GeometryCreator();
    }

    createTerrain(width, height, segments, heightMap) {
        // 创建顶点
        for (let i = 0; i <= segments; i++) {
            for (let j = 0; j <= segments; j++) {
                const x = (i / segments - 0.5) * width;
                const z = (j / segments - 0.5) * height;
                const y = heightMap(i / segments, j / segments);

                this.creator.addVertex(x, y, z);
                this.creator.addUV(i / segments, j / segments);
            }
        }

        // 创建面
        for (let i = 0; i < segments; i++) {
            for (let j = 0; j < segments; j++) {
                const a = i * (segments + 1) + j;
                const b = a + 1;
                const c = (i + 1) * (segments + 1) + j;
                const d = c + 1;

                this.creator.addFace(a, b, c);
                this.creator.addFace(b, d, c);
            }
        }

        return this.creator.createGeometry();
    }

    createFractalTree(iterations, angle, scale) {
        // 创建树干
        this.createBranch(0, 0, 0, 0, 1, 0, 1, iterations, angle, scale);
        return this.creator.createGeometry();
    }

    createBranch(x, y, z, dirX, dirY, dirZ, length, iterations, angle, scale) {
        if (iterations <= 0) return;

        // 创建当前分支
        const endX = x + dirX * length;
        const endY = y + dirY * length;
        const endZ = z + dirZ * length;

        // 添加顶点
        const v1 = this.creator.addVertex(x, y, z);
        const v2 = this.creator.addVertex(endX, endY, endZ);

        // 添加面
        this.creator.addFace(v1, v2, v1);

        // 创建子分支
        const newLength = length * scale;
        const newIterations = iterations - 1;

        // 左分支
        const leftDir = this.rotateVector(dirX, dirY, dirZ, angle);
        this.createBranch(endX, endY, endZ, leftDir.x, leftDir.y, leftDir.z, newLength, newIterations, angle, scale);

        // 右分支
        const rightDir = this.rotateVector(dirX, dirY, dirZ, -angle);
        this.createBranch(endX, endY, endZ, rightDir.x, rightDir.y, rightDir.z, newLength, newIterations, angle, scale);
    }

    rotateVector(x, y, z, angle) {
        const cos = Math.cos(angle);
        const sin = Math.sin(angle);

        return {
            x: x * cos - z * sin,
            y: y,
            z: x * sin + z * cos
        };
    }
}
```

## 实战：创建自定义几何体

让我们创建一个完整的自定义几何体示例：

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

// 创建几何体生成器
const geometryGenerator = new ProceduralGeometryGenerator();

// 创建地形
const terrainGeometry = geometryGenerator.createTerrain(
    20, 20, 32,
    (x, z) => {
        return Math.sin(x * Math.PI * 2) * Math.cos(z * Math.PI * 2) * 2;
    }
);

const terrainMaterial = new THREE.MeshStandardMaterial({
    color: 0x808080,
    wireframe: false
});

const terrain = new THREE.Mesh(terrainGeometry, terrainMaterial);
terrain.receiveShadow = true;
scene.add(terrain);

// 创建分形树
const treeGeometry = geometryGenerator.createFractalTree(5, Math.PI / 6, 0.7);
const treeMaterial = new THREE.MeshStandardMaterial({
    color: 0x00ff00,
    wireframe: false
});

const tree = new THREE.Mesh(treeGeometry, treeMaterial);
tree.castShadow = true;
tree.position.set(0, 2, 0);
scene.add(tree);

// 创建变形几何体
const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
const morphGeometry = new MorphGeometry(boxGeometry);

// 添加变形目标
morphGeometry.addMorphTarget('twist', new Float32Array([
    -0.5, -0.5, -0.5,
    0.5, -0.5, -0.5,
    0.5, 0.5, -0.5,
    -0.5, 0.5, -0.5,
    -0.5, -0.5, 0.5,
    0.5, -0.5, 0.5,
    0.5, 0.5, 0.5,
    -0.5, 0.5, 0.5
]));

const morphMaterial = new THREE.MeshStandardMaterial({
    color: 0xff0000,
    wireframe: false
});

const morphMesh = new THREE.Mesh(morphGeometry.geometry, morphMaterial);
morphMesh.castShadow = true;
morphMesh.position.set(3, 2, 0);
scene.add(morphMesh);

// 动画循环
function animate() {
    requestAnimationFrame(animate);

    // 更新变形几何体
    morphGeometry.setMorphTargetInfluence(0, Math.sin(Date.now() * 0.001) * 0.5 + 0.5);
    morphGeometry.updateGeometry();

    // 旋转树
    tree.rotation.y += 0.01;

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

1. 创建基础几何体
2. 实现参数化几何体
3. 添加变形效果
4. 创建程序化几何体

## 下一步学习

在下一章中，我们将学习：
- 高级动画系统
- 跨平台适配
- 性能优化
- 最佳实践

> 最后感谢阅读！欢迎关注我，微信公众号：`《鲫小鱼不正经》`。欢迎点赞、收藏、关注，一键三连！！！