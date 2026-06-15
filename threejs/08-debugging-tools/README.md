# Phase 8 — Debugging & Tooling

## 1. Stats Panel (FPS Monitor)

```javascript
// Vanilla Three.js
import Stats from 'three/addons/libs/stats.module.js'

const stats = new Stats()
stats.showPanel(0)  // 0=FPS, 1=MS per frame, 2=MB memory
document.body.appendChild(stats.dom)
stats.dom.style.position = 'absolute'
stats.dom.style.top = '0px'

function animate() {
  stats.begin()
  renderer.render(scene, camera)
  stats.end()
}
```

```tsx
// React Three Fiber — use drei
import { Stats } from '@react-three/drei'

// Inside <Canvas>:
<Stats />
```

---

## 2. Leva GUI (debug controls for R3F)

```tsx
import { useControls } from 'leva'

function Scene() {
  const { color, roughness, metalness, lightIntensity } = useControls('Material', {
    color: '#4ecdc4',
    roughness: { value: 0.5, min: 0, max: 1, step: 0.01 },
    metalness: { value: 0, min: 0, max: 1, step: 0.01 },
    lightIntensity: { value: 1, min: 0, max: 5, step: 0.1 },
  })

  return (
    <>
      <directionalLight intensity={lightIntensity} />
      <mesh>
        <meshStandardMaterial
          color={color}
          roughness={roughness}
          metalness={metalness}
        />
      </mesh>
    </>
  )
}
```

---

## 3. Helper Objects (Visual Debug)

```javascript
// Axes helper — shows X(red), Y(green), Z(blue) axes
scene.add(new THREE.AxesHelper(5))

// Grid
scene.add(new THREE.GridHelper(20, 20))

// Bounding box around object
scene.add(new THREE.BoxHelper(mesh, 0xffff00))

// Arrow showing direction/vector
const arrowHelper = new THREE.ArrowHelper(
  direction.normalize(),  // direction
  origin,                 // starting point
  length,
  0xff0000               // color
)
scene.add(arrowHelper)

// Light position/direction
scene.add(new THREE.DirectionalLightHelper(dirLight, 1))
scene.add(new THREE.PointLightHelper(pointLight, 0.3))

// Shadow camera frustum
const shadowHelper = new THREE.CameraHelper(dirLight.shadow.camera)
scene.add(shadowHelper)

// Wireframe overlay
mesh.material.wireframe = true
// OR add separate wireframe
const wireframe = new THREE.WireframeGeometry(geometry)
const lines = new THREE.LineSegments(wireframe, new THREE.LineBasicMaterial({ color: 0x00ff00 }))
scene.add(lines)
```

---

## 4. Renderer Info

```javascript
// Check every frame to find bottlenecks
function animate() {
  renderer.render(scene, camera)

  const info = renderer.info
  document.getElementById('debug').innerHTML = `
    Draw calls: ${info.render.calls}<br>
    Triangles: ${info.render.triangles}<br>
    Geometries: ${info.memory.geometries}<br>
    Textures: ${info.memory.textures}<br>
    Programs: ${info.programs?.length}
  `

  renderer.info.reset()  // reset per-frame counters
}
```

---

## 5. Spector.js (Chrome Extension)

- Install: Chrome Web Store → "Spector.js"
- Click the icon on any Three.js page
- "Capture" → shows ALL WebGL calls for one frame:
  - Every draw call with the exact geometry
  - Shader programs (vertex + fragment source)
  - All textures bound to GPU
  - Framebuffer contents
- Use to debug: missing textures, wrong draw order, shader errors

---

## 6. Three.js Editor

Open https://threejs.org/editor in browser:
- Import your .glb models to check they load correctly
- Inspect scene hierarchy
- Test materials/lights interactively
- Export scenes as JSON

---

## 7. Chrome DevTools for 3D

**Performance tab:**
1. Open DevTools → Performance
2. Click Record
3. Play your animation for ~5 seconds
4. Stop recording
5. Look at:
   - Main thread (JS): should be < 8ms per frame
   - GPU (if shown): should have consistent frame times
   - Memory: should be flat (no growing heap = no leaks)
   - `animate` function time in flame chart

**Memory tab:**
1. Take Heap Snapshot
2. Interact with app (create/destroy objects)
3. Take another snapshot
4. Compare → check for Three.js objects not getting GC'd
5. If `THREE.BufferGeometry` count keeps growing → memory leak

---

## 8. Common Bugs & Fixes

| Bug | Cause | Fix |
|---|---|---|
| Object invisible | Wrong position/scale/rotation | Add `AxesHelper`, check values |
| Black screen | Missing light + MeshStandardMaterial | Add AmbientLight |
| Flickering (z-fighting) | Two surfaces at same depth | Move apart, adjust near/far, use `polygonOffset` |
| Texture pixelated | Wrong filter or non-power-of-2 | Set `LinearMipmapLinearFilter`, use 512/1024px |
| Texture black | Wrong colorSpace | Set `texture.colorSpace = THREE.SRGBColorSpace` |
| Shadows not appearing | Not enabled or wrong shadow camera frustum | Enable renderer, light, objects; adjust shadow camera |
| Transparent objects look wrong | Render order issue | Set `depthWrite = false`, adjust `renderOrder` |
| Memory leak | Not disposing | Call `geometry.dispose()`, `material.dispose()`, `texture.dispose()` |
| Shader compile error | GLSL syntax error | Check browser console, add `renderer.debug.checkShaderErrors = true` |
| FPS drops with many objects | Too many draw calls | Use InstancedMesh or mergeGeometries |

---

## 9. Useful console.log patterns

```javascript
// Inspect object3D tree
function printScene(obj, indent = 0) {
  console.log(' '.repeat(indent) + `${obj.type}: ${obj.name}`)
  obj.children.forEach(child => printScene(child, indent + 2))
}
printScene(scene)

// Find all meshes
const meshes = []
scene.traverse(obj => { if (obj.isMesh) meshes.push(obj) })
console.log('Meshes:', meshes.length)

// Check bounding box size
const box = new THREE.Box3().setFromObject(mesh)
const size = new THREE.Vector3()
box.getSize(size)
console.log('Size:', size)

// Check world position
const worldPos = new THREE.Vector3()
mesh.getWorldPosition(worldPos)
console.log('World position:', worldPos)
```

---

## 10. Error Types

```
THREE.WebGLRenderer: Context Lost
→ GPU ran out of memory, or page was backgrounded
→ Fix: dispose unused objects, reduce texture sizes

THREE.WebGLProgram: shader error
→ GLSL compilation failed
→ Fix: check browser console for line number + error

THREE.WebGLTextures: Image has non-power-of-two size
→ Texture not 256/512/1024/2048
→ Fix: resize image or set repeat wrapping to ClampToEdgeWrapping

Cannot read property 'x' of undefined
→ Trying to access Vector3/Quaternion before object initialized
→ Fix: check initialization order, use optional chaining

Maximum update depth exceeded (React Three Fiber)
→ useFrame causing state updates every frame
→ Fix: don't call setState in useFrame, use refs instead
```
