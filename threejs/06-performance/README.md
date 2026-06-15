# Phase 6 — Performance Optimization

> Critical cho Digital Twin / Warehouse roles — 100+ robots phải chạy 60fps

## Core Principle: Minimize Draw Calls

Mỗi `renderer.render()` gửi draw calls đến GPU. Mỗi draw call = 1 object với 1 material.
**Target: dưới 100 draw calls cho scene 60fps.**

```javascript
// Check draw calls
console.log(renderer.info.render.calls)    // draw calls this frame
console.log(renderer.info.render.triangles)
console.log(renderer.info.memory.geometries)
console.log(renderer.info.memory.textures)
```

---

## 1. Instancing — Most Important Optimization

**1 draw call cho 1000+ objects.**

```javascript
// Vanilla Three.js InstancedMesh
const count = 1000
const mesh = new THREE.InstancedMesh(geometry, material, count)

const dummy = new THREE.Object3D()
for (let i = 0; i < count; i++) {
  dummy.position.set(Math.random()*20, 0, Math.random()*20)
  dummy.rotation.y = Math.random() * Math.PI * 2
  dummy.updateMatrix()
  mesh.setMatrixAt(i, dummy.matrix)
  mesh.setColorAt(i, new THREE.Color().setHSL(i/count, 0.8, 0.5))
}
mesh.instanceMatrix.needsUpdate = true
mesh.instanceColor.needsUpdate = true
scene.add(mesh)  // 1 draw call for 1000 objects!

// Update specific instance at runtime
const matrix = new THREE.Matrix4()
mesh.getMatrixAt(targetIndex, matrix)
// modify matrix...
mesh.setMatrixAt(targetIndex, matrix)
mesh.instanceMatrix.needsUpdate = true
```

---

## 2. Geometry Merging (Static Objects)

Combine static geometries → 1 draw call.

```javascript
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js'

// Merge floor tiles, wall segments, etc.
const geometries = []
for (let i = 0; i < 100; i++) {
  const tileGeom = new THREE.PlaneGeometry(1, 1)
  const matrix = new THREE.Matrix4().makeTranslation(i % 10, 0, Math.floor(i/10))
  tileGeom.applyMatrix4(matrix)  // bake transform into geometry
  geometries.push(tileGeom)
}

const merged = mergeGeometries(geometries)
const floor = new THREE.Mesh(merged, floorMaterial)  // 1 draw call for 100 tiles
scene.add(floor)

// IMPORTANT: merging = static, cannot move individual pieces after merge
```

---

## 3. LOD (Level of Detail)

Different detail levels based on distance from camera.

```javascript
import { LOD } from 'three'

const lod = new LOD()

// High detail (close up, < 5 units)
lod.addLevel(highPolyMesh, 0)

// Medium detail (5–20 units)
lod.addLevel(medPolyMesh, 5)

// Low detail (20–100 units)
lod.addLevel(lowPolyMesh, 20)

// Invisible (> 100 units)
lod.addLevel(new THREE.Object3D(), 100)

scene.add(lod)

// LOD updates automatically in render loop based on camera distance
// lod.update(camera)  — call manually if not using raycaster
```

---

## 4. Frustum Culling

Objects outside camera view are not rendered — Three.js does this automatically.

```javascript
// Ensure bounding sphere/box is computed for culling to work
geometry.computeBoundingSphere()
geometry.computeBoundingBox()

// Disable for objects that should always render (skybox, fullscreen quad)
mesh.frustumCulled = false

// Verify culling is working:
// renderer.info.render.calls should drop when camera looks away
```

---

## 5. Texture Optimization

```javascript
// 1. Use power-of-2 textures (256, 512, 1024, 2048)
// Non-power-of-2 disables mipmapping!

// 2. Use KTX2/Basis Universal — GPU-compressed textures (much faster loading)
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js'
// Convert PNG → KTX2: npm install -g @gltf-transform/cli
// npx gltf-transform etc optimize model.glb optimized.glb --texture-compress ktx2

// 3. Resize textures to what you actually need
// A 4096x4096 texture for a small box is wasteful

// 4. Share textures across materials
const sharedTexture = textureLoader.load('texture.jpg')
const mat1 = new THREE.MeshStandardMaterial({ map: sharedTexture })
const mat2 = new THREE.MeshStandardMaterial({ map: sharedTexture })  // same GPU memory

// 5. Anisotropy only where needed (floor, walls — not ceiling)
texture.anisotropy = renderer.capabilities.getMaxAnisotropy()  // costs GPU fill rate
```

---

## 6. Material Optimization

```javascript
// 1. Share materials across meshes (same GPU program)
const sharedMat = new THREE.MeshStandardMaterial({ color: 0xff0000 })
// 1000 meshes sharing 1 material = 1 shader program

// 2. Use simpler materials for distant/small objects
// MeshBasicMaterial >> MeshStandardMaterial (no lighting calculation)

// 3. Avoid transparent materials when possible
// Transparent objects CANNOT be sorted with the depth buffer
// → always rendered last → expensive
// If you need transparency: use alphaTest instead of opacity when possible
material.alphaTest = 0.5  // discard pixels below threshold, no sorting needed

// 4. Avoid too many unique materials
// Each unique shader program = compile time + GPU state switch
```

---

## 7. Animation Loop Optimization

```javascript
// Bad: setInterval for animations (runs even when tab hidden)
// Good: requestAnimationFrame (pauses when tab hidden)

// Bad: creating objects inside render loop
function animate() {
  const color = new THREE.Color(0xff0000)  // allocates memory every frame!
}

// Good: reuse objects
const reusableColor = new THREE.Color()
function animate() {
  reusableColor.setHex(Math.random() * 0xffffff)
}

// Conditional rendering (only render when needed)
let needsRender = true
window.addEventListener('mousemove', () => { needsRender = true })

function animate() {
  requestAnimationFrame(animate)
  if (!needsRender) return
  renderer.render(scene, camera)
  needsRender = false
}
```

---

## 8. Worker Threads for Heavy Computation

```javascript
// Don't block main thread with heavy math
// Use Web Workers for pathfinding, physics preprocessing, etc.

// worker.js
self.onmessage = ({ data }) => {
  const { robotPositions } = data
  // Heavy A* pathfinding
  const paths = robotPositions.map(pos => findPath(pos, target))
  self.postMessage({ paths })
}

// main.js
const worker = new Worker('/worker.js')
worker.postMessage({ robotPositions: positions })
worker.onmessage = ({ data }) => {
  updateRobotPaths(data.paths)
}
```

---

## 9. Memory Management

```javascript
// Always dispose when removing objects
function removeFromScene(mesh) {
  scene.remove(mesh)
  mesh.geometry.dispose()

  if (Array.isArray(mesh.material)) {
    mesh.material.forEach(m => {
      m.map?.dispose()
      m.normalMap?.dispose()
      // ... dispose all textures
      m.dispose()
    })
  } else {
    mesh.material.map?.dispose()
    mesh.material.dispose()
  }
}

// Check for leaks
console.log(renderer.info.memory)
// { geometries: N, textures: N } — watch these don't grow over time

// Three.js caches geometries/materials internally
// Reset cache if building editor-style apps where many unique objects come and go
THREE.Cache.clear()
```

---

## 10. Renderer Settings

```javascript
// Cap pixel ratio at 2 (retina screens go up to 3, not worth it for 3D)
renderer.setPixelRatio(Math.min(devicePixelRatio, 2))

// Use lower resolution render target + upscale
renderer.setSize(innerWidth / 2, innerHeight / 2)
renderer.domElement.style.width = '100%'
renderer.domElement.style.height = '100%'

// Disable features you don't use
renderer.shadowMap.enabled = false  // if not using shadows
renderer.physicallyCorrectLights = false  // legacy, now default in r150+

// Power preference hint (for mobile)
const renderer = new THREE.WebGLRenderer({
  powerPreference: 'high-performance'  // use discrete GPU
})
```

---

## 11. Performance Profiling Tools

| Tool | How to use | What it shows |
|---|---|---|
| Stats.js | `npm i stats.js` + `stats.showPanel(0)` | FPS, MS, MB in overlay |
| Chrome DevTools Performance | Record + scroll | Main thread, GPU frames |
| Spector.js (Chrome ext) | Click extension icon | All draw calls, textures, shaders |
| `renderer.info` | `console.log(renderer.info)` | Draw calls, triangles, memory |
| Three.js Editor | threejs.org/editor | Visual scene inspection |

---

## 12. Benchmarks to Target

| Scenario | Target |
|---|---|
| Draw calls | < 100 per frame |
| Triangles | < 500k per frame |
| Texture memory | < 100MB |
| FPS | 60fps desktop, 30fps mobile |
| JS frame time | < 8ms (leaves 8ms for GPU) |

---

## 13. Warehouse Digital Twin Specific

```
100 robots × individual Mesh     = 100 draw calls   ❌
100 robots × InstancedMesh       = 1 draw call       ✅

Floor tiles × individual Mesh    = N draw calls      ❌
Floor tiles × mergeGeometries    = 1 draw call       ✅

Robot labels × Html (drei)       = DOM, not WebGL    ✅ (show only for selected)
Robot paths × Line2              = 1 per robot       ⚠ (show only nearby robots)
```

**Target architecture for 100-robot warehouse:**
- 1 InstancedMesh for robot bodies
- 1 InstancedMesh for robot sensor domes
- 1 merged geometry for warehouse floor
- 1 merged geometry for shelving
- Line paths: only render for selected robot
- HTML labels: only for hovered/selected robot
- Result: < 20 draw calls total → 60fps stable
