# Phase 2 — Three.js Core (Complete Guide)

## 1. Scene Graph & Object Hierarchy

Three.js dùng **scene graph** — tree of Objects3D. Parent transform ảnh hưởng children.

```javascript
const group = new THREE.Group()

const body = new THREE.Mesh(new THREE.BoxGeometry(1, 1, 1), mat)
const head = new THREE.Mesh(new THREE.SphereGeometry(0.4), mat)
head.position.y = 1.0  // relative to group

group.add(body)
group.add(head)
group.position.set(5, 0, 0)  // moves BOTH body + head
scene.add(group)

// Traverse all children
scene.traverse((object) => {
  if (object.isMesh) {
    object.castShadow = true
  }
})

// Get world position (after parent transforms applied)
const worldPos = new THREE.Vector3()
head.getWorldPosition(worldPos)

// Local vs World space
head.position         // local (relative to group)
head.getWorldPosition() // world (absolute in scene)
```

---

## 2. Geometry

### Built-in Geometries
```javascript
new THREE.BoxGeometry(w, h, d, wSegs, hSegs, dSegs)
new THREE.SphereGeometry(radius, widthSegs, heightSegs)
new THREE.PlaneGeometry(w, h, wSegs, hSegs)
new THREE.CylinderGeometry(radiusTop, radiusBottom, height, radialSegs)
new THREE.ConeGeometry(radius, height, radialSegs)
new THREE.TorusGeometry(radius, tube, radialSegs, tubularSegs)
new THREE.TorusKnotGeometry(radius, tube)
new THREE.CircleGeometry(radius, segments)
new THREE.RingGeometry(innerRadius, outerRadius, thetaSegs)
new THREE.TubeGeometry(path, tubularSegs, radius, radialSegs)
new THREE.LatheGeometry(points)       // rotation around Y axis
new THREE.ExtrudeGeometry(shape, options) // extrude 2D shape to 3D
new THREE.ShapeGeometry(shape)        // 2D filled shape
new THREE.EdgesGeometry(geometry)     // edges only
new THREE.WireframeGeometry(geometry) // wireframe
```

### Custom BufferGeometry
```javascript
const geometry = new THREE.BufferGeometry()

// Triangle vertices (counter-clockwise winding = front face)
const positions = new Float32Array([
  0,  1, 0,   // vertex 0 (top)
 -1, -1, 0,   // vertex 1 (bottom-left)
  1, -1, 0,   // vertex 2 (bottom-right)
])
geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

// Normals (cho lighting)
const normals = new Float32Array([
  0, 0, 1,
  0, 0, 1,
  0, 0, 1,
])
geometry.setAttribute('normal', new THREE.BufferAttribute(normals, 3))

// UV coordinates
const uvs = new Float32Array([
  0.5, 1.0,
  0.0, 0.0,
  1.0, 0.0,
])
geometry.setAttribute('uv', new THREE.BufferAttribute(uvs, 2))

// Index buffer (reuse vertices — memory efficient)
const indices = new Uint16Array([0, 1, 2])
geometry.setIndex(new THREE.BufferAttribute(indices, 1))

// Auto-compute normals if not providing them
geometry.computeVertexNormals()
```

### InstancedMesh (best performance for many identical objects)
```javascript
const count = 1000
const geometry = new THREE.BoxGeometry(0.1, 0.1, 0.1)
const material = new THREE.MeshStandardMaterial({ color: 0x4ecdc4 })
const mesh = new THREE.InstancedMesh(geometry, material, count)

const dummy = new THREE.Object3D()
const color = new THREE.Color()

for (let i = 0; i < count; i++) {
  dummy.position.set(
    (Math.random() - 0.5) * 20,
    (Math.random() - 0.5) * 20,
    (Math.random() - 0.5) * 20
  )
  dummy.updateMatrix()
  mesh.setMatrixAt(i, dummy.matrix)
  mesh.setColorAt(i, color.setHSL(i / count, 0.8, 0.5))
}

mesh.instanceMatrix.needsUpdate = true
mesh.instanceColor.needsUpdate = true
scene.add(mesh)

// Update single instance at runtime
mesh.getMatrixAt(5, dummy.matrix)  // read
dummy.position.y += 0.1
dummy.updateMatrix()
mesh.setMatrixAt(5, dummy.matrix)  // write
mesh.instanceMatrix.needsUpdate = true
```

---

## 3. Materials

### Material types
```javascript
// MeshBasicMaterial — no lighting, flat color/texture
new THREE.MeshBasicMaterial({ color: 0xff0000, wireframe: false })

// MeshStandardMaterial — PBR, physically based (DÙNG NHIỀU NHẤT)
new THREE.MeshStandardMaterial({
  color: 0xffffff,      // base color
  metalness: 0.0,       // 0 = non-metal, 1 = metal
  roughness: 0.5,       // 0 = mirror, 1 = matte
  map: colorTexture,    // albedo/color map
  normalMap: normalTex, // surface detail
  roughnessMap: roughTex,
  metalnessMap: metalTex,
  aoMap: aoTex,         // ambient occlusion
  emissive: 0x000000,   // self-illumination color
  emissiveMap: emissiveTex,
  emissiveIntensity: 1.0,
  transparent: false,
  opacity: 1.0,
  alphaMap: alphaTex,   // grayscale → alpha
  side: THREE.FrontSide, // FrontSide | BackSide | DoubleSide
  envMap: envTexture,   // reflection/IBL
  envMapIntensity: 1.0,
})

// MeshPhysicalMaterial — extends Standard, more realistic
new THREE.MeshPhysicalMaterial({
  ...standardOptions,
  clearcoat: 1.0,       // car paint top coat
  clearcoatRoughness: 0.0,
  transmission: 1.0,    // glass/transparency (physically correct)
  thickness: 0.5,
  ior: 1.5,             // index of refraction (glass=1.5, water=1.33)
  iridescence: 1.0,     // soap bubble effect
})

// MeshLambertMaterial — fast, non-PBR, diffuse only
// MeshPhongMaterial — fast, specular highlights (non-PBR)
// MeshToonMaterial — cel-shading / cartoon look
// PointsMaterial — for particle systems
// LineBasicMaterial / LineDashedMaterial — for lines
// SpriteMaterial — for sprites/billboards
```

### Material properties
```javascript
material.transparent = true
material.opacity = 0.5
material.depthWrite = false  // important for transparent objects
material.depthTest = true
material.blending = THREE.AdditiveBlending  // NormalBlending | AdditiveBlending | MultiplyBlending
material.side = THREE.DoubleSide
material.wireframe = true
material.fog = true          // affected by scene fog
material.flatShading = true  // low-poly look
material.vertexColors = true // use geometry color attribute

// Update material after changing properties
material.needsUpdate = true  // only needed for some properties

// Dispose when no longer needed (free GPU memory)
material.dispose()
geometry.dispose()
texture.dispose()
```

---

## 4. Textures

### Loading textures
```javascript
const loader = new THREE.TextureLoader()

// Async load
const texture = await loader.loadAsync('/textures/color.jpg')

// With LoadingManager (track all loads)
const manager = new THREE.LoadingManager(
  () => console.log('All loaded!'),           // onLoad
  (url, loaded, total) => console.log(`${loaded}/${total}`), // onProgress
  (url) => console.error(`Error: ${url}`)    // onError
)
const loaderManaged = new THREE.TextureLoader(manager)
```

### Texture settings
```javascript
texture.wrapS = THREE.RepeatWrapping    // horizontal: ClampToEdgeWrapping | RepeatWrapping | MirroredRepeatWrapping
texture.wrapT = THREE.RepeatWrapping    // vertical
texture.repeat.set(4, 4)               // tile 4x4
texture.offset.set(0.5, 0)             // shift UV
texture.rotation = Math.PI / 4         // rotate texture
texture.center.set(0.5, 0.5)           // rotation pivot

texture.minFilter = THREE.LinearMipmapLinearFilter // best quality downscaling
texture.magFilter = THREE.LinearFilter  // upscaling
texture.anisotropy = renderer.capabilities.getMaxAnisotropy() // sharp at angles

texture.colorSpace = THREE.SRGBColorSpace  // for color/albedo maps
// Normal maps and data textures: THREE.LinearSRGBColorSpace (default)
```

### Texture types in PBR workflow
```javascript
const material = new THREE.MeshStandardMaterial({
  // Color map — base color/albedo (sRGB)
  map: colorTexture,

  // Normal map — surface microdetails WITHOUT adding geometry
  // RGB encodes XYZ normal direction
  normalMap: normalTexture,
  normalScale: new THREE.Vector2(1, 1),  // strength

  // Roughness map — grayscale, white=rough, black=smooth
  roughnessMap: roughnessTexture,
  roughness: 1.0,  // multiplied with map

  // Metalness map — grayscale, white=metal, black=non-metal
  metalnessMap: metalnessTexture,
  metalness: 1.0,

  // Ambient Occlusion map — grayscale, darkens crevices
  // Requires second UV set (uv2)
  aoMap: aoTexture,
  aoMapIntensity: 1.0,

  // Emissive map — self-glowing parts (good for screens, LEDs)
  emissiveMap: emissiveTexture,
  emissive: new THREE.Color(0xffffff),
  emissiveIntensity: 2.0,

  // Displacement map — actually moves vertices (needs subdivided geometry)
  displacementMap: heightTexture,
  displacementScale: 0.1,

  // Alpha map — grayscale → transparency
  alphaMap: alphaTexture,
  transparent: true,
})

// For aoMap, need UV2
geometry.setAttribute('uv2', geometry.attributes.uv)
```

### Environment Map (IBL — Image Based Lighting)
```javascript
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js'
import { PMREMGenerator } from 'three'

const pmremGenerator = new THREE.PMREMGenerator(renderer)
pmremGenerator.compileEquirectangularShader()

const hdrLoader = new RGBELoader()
hdrLoader.load('/textures/studio.hdr', (texture) => {
  const envMap = pmremGenerator.fromEquirectangular(texture).texture

  scene.environment = envMap   // affects all MeshStandardMaterial with envMap
  scene.background = envMap    // visible background
  // OR: material.envMap = envMap (per-material)

  texture.dispose()
  pmremGenerator.dispose()
})
```

### CubeTextureLoader (skybox)
```javascript
const cubeLoader = new THREE.CubeTextureLoader()
const cubeTexture = cubeLoader.load([
  'px.jpg', 'nx.jpg',  // +X, -X
  'py.jpg', 'ny.jpg',  // +Y, -Y
  'pz.jpg', 'nz.jpg',  // +Z, -Z
])
scene.background = cubeTexture
```

---

## 5. Lighting

### Light types
```javascript
// AmbientLight — no direction, affects all faces equally
// Use for global fill light, low intensity (0.1–0.5)
const ambient = new THREE.AmbientLight(0xffffff, 0.3)
scene.add(ambient)

// DirectionalLight — parallel rays, like sun, has shadows
const dirLight = new THREE.DirectionalLight(0xffffff, 1.0)
dirLight.position.set(5, 10, 5)
dirLight.target.position.set(0, 0, 0)
scene.add(dirLight)
scene.add(dirLight.target)

// PointLight — radiates in all directions from a point
const pointLight = new THREE.PointLight(
  0xffffff,  // color
  1.0,       // intensity
  10,        // distance (0 = infinite)
  2          // decay (2 = physically accurate)
)
pointLight.position.set(2, 3, 2)
scene.add(pointLight)

// SpotLight — cone of light
const spotLight = new THREE.SpotLight(
  0xffffff, 1.0,
  20,   // distance
  Math.PI / 4,  // angle (max PI/2)
  0.1,  // penumbra (soft edge, 0–1)
  2     // decay
)
spotLight.position.set(0, 5, 0)
scene.add(spotLight)

// HemisphereLight — sky color from above, ground color from below
const hemiLight = new THREE.HemisphereLight(0x87ceeb, 0x8b7355, 0.5)
scene.add(hemiLight)

// RectAreaLight — like a softbox (works with MeshStandardMaterial/MeshPhysicalMaterial)
import { RectAreaLightHelper } from 'three/addons/helpers/RectAreaLightHelper.js'
import { RectAreaLightUniformsLib } from 'three/addons/lights/RectAreaLightUniformsLib.js'
RectAreaLightUniformsLib.init()
const rectLight = new THREE.RectAreaLight(0xffffff, 5, 2, 1) // color, intensity, width, height
rectLight.position.set(0, 3, 0)
rectLight.lookAt(0, 0, 0)
scene.add(rectLight)
```

### Shadows
```javascript
// Enable on renderer
renderer.shadowMap.enabled = true
renderer.shadowMap.type = THREE.PCFSoftShadowMap  // Soft shadows

// Enable on lights (only Directional, Point, Spot support shadows)
dirLight.castShadow = true
dirLight.shadow.mapSize.width = 2048   // shadow map resolution (power of 2)
dirLight.shadow.mapSize.height = 2048
dirLight.shadow.camera.near = 0.5     // shadow camera frustum
dirLight.shadow.camera.far = 50
dirLight.shadow.camera.left = -10     // for DirectionalLight
dirLight.shadow.camera.right = 10
dirLight.shadow.camera.top = 10
dirLight.shadow.camera.bottom = -10
dirLight.shadow.bias = -0.001         // fix shadow acne
dirLight.shadow.normalBias = 0.02

// Enable on objects
mesh.castShadow = true    // casts shadow onto others
mesh.receiveShadow = true // receives shadows from others

// Visualize shadow camera (debug)
const helper = new THREE.CameraHelper(dirLight.shadow.camera)
scene.add(helper)
```

### Light Helpers (Debug)
```javascript
scene.add(new THREE.DirectionalLightHelper(dirLight, 1))
scene.add(new THREE.PointLightHelper(pointLight, 0.5))
scene.add(new THREE.SpotLightHelper(spotLight))
scene.add(new THREE.HemisphereLightHelper(hemiLight, 1))
scene.add(new THREE.AxesHelper(3))  // X=red, Y=green, Z=blue
scene.add(new THREE.BoxHelper(mesh, 0xffff00))  // bounding box
scene.add(new THREE.GridHelper(10, 10))
```

---

## 6. Camera

```javascript
// PerspectiveCamera
const camera = new THREE.PerspectiveCamera(
  75,                     // fov (vertical, degrees)
  innerWidth / innerHeight, // aspect ratio
  0.1,                    // near (objects closer than this = invisible)
  1000                    // far (objects farther than this = invisible)
)
camera.position.set(0, 5, 10)
camera.lookAt(0, 0, 0)
// OR: camera.lookAt(new THREE.Vector3(0, 0, 0))

// Update aspect on resize
camera.aspect = newWidth / newHeight
camera.updateProjectionMatrix()  // REQUIRED after changing any camera property

// OrthographicCamera (no perspective, good for 2D/maps/UI)
const aspect = innerWidth / innerHeight
const frustumSize = 10
const camera = new THREE.OrthographicCamera(
  -frustumSize * aspect / 2, // left
   frustumSize * aspect / 2, // right
   frustumSize / 2,           // top
  -frustumSize / 2,           // bottom
  0.1, 1000
)

// Convert 3D world position to 2D screen position
const worldPos = new THREE.Vector3(1, 2, 3)
worldPos.project(camera)
// worldPos.x and .y are now NDC (-1 to 1)
const screenX = (worldPos.x + 1) / 2 * innerWidth
const screenY = -(worldPos.y - 1) / 2 * innerHeight
```

---

## 7. Renderer & Post-processing

### WebGLRenderer setup
```javascript
const renderer = new THREE.WebGLRenderer({
  canvas: document.querySelector('canvas'),
  antialias: true,           // MSAA anti-aliasing
  alpha: false,              // transparent background
  powerPreference: 'high-performance',
})
renderer.setSize(innerWidth, innerHeight)
renderer.setPixelRatio(Math.min(devicePixelRatio, 2))  // cap at 2x

// Color management (Three.js r152+)
renderer.outputColorSpace = THREE.SRGBColorSpace

// Tone mapping (HDR to LDR)
renderer.toneMapping = THREE.ACESFilmicToneMapping  // cinematic
renderer.toneMappingExposure = 1.0

// Shadows
renderer.shadowMap.enabled = true
renderer.shadowMap.type = THREE.PCFSoftShadowMap
```

### Post-processing (EffectComposer)
```javascript
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js'
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js'
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js'
import { SSAOPass } from 'three/addons/postprocessing/SSAOPass.js'
import { SMAAPass } from 'three/addons/postprocessing/SMAAPass.js'  // better AA
import { ShaderPass } from 'three/addons/postprocessing/ShaderPass.js'
import { FXAAShader } from 'three/addons/shaders/FXAAShader.js'

const composer = new EffectComposer(renderer)

// 1. Render scene first
composer.addPass(new RenderPass(scene, camera))

// 2. SSAO — ambient occlusion
const ssaoPass = new SSAOPass(scene, camera, innerWidth, innerHeight)
ssaoPass.kernelRadius = 16
composer.addPass(ssaoPass)

// 3. Bloom — glow effect
const bloomPass = new UnrealBloomPass(
  new THREE.Vector2(innerWidth, innerHeight),
  1.5,   // strength
  0.4,   // radius
  0.85   // threshold
)
composer.addPass(bloomPass)

// 4. FXAA — anti-aliasing (use instead of renderer antialias when using composer)
const fxaaPass = new ShaderPass(FXAAShader)
fxaaPass.uniforms.resolution.value.set(1/innerWidth, 1/innerHeight)
composer.addPass(fxaaPass)

// Render with composer instead of renderer
function animate() {
  requestAnimationFrame(animate)
  composer.render()  // instead of renderer.render(scene, camera)
}
```

---

## 8. Animation System

### AnimationMixer (for glTF animations)
```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js'

const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('/draco/')  // copy from three/examples/jsm/libs/draco/

const gltfLoader = new GLTFLoader()
gltfLoader.setDRACOLoader(dracoLoader)

let mixer

gltfLoader.load('/models/robot.glb', (gltf) => {
  const model = gltf.scene
  scene.add(model)

  mixer = new THREE.AnimationMixer(model)

  // Play all animations
  gltf.animations.forEach((clip) => {
    const action = mixer.clipAction(clip)
    action.play()
  })

  // Or play specific clip
  const walkClip = THREE.AnimationClip.findByName(gltf.animations, 'Walk')
  const walkAction = mixer.clipAction(walkClip)
  walkAction.play()
  walkAction.setLoop(THREE.LoopRepeat, Infinity)

  // Transition between animations
  const idleAction = mixer.clipAction(idleClip)
  idleAction.play()
  walkAction.crossFadeFrom(idleAction, 0.5, true)  // blend over 0.5s
})

// Update mixer in animation loop
const clock = new THREE.Clock()
function animate() {
  requestAnimationFrame(animate)
  const delta = clock.getDelta()  // seconds since last frame
  if (mixer) mixer.update(delta)
  renderer.render(scene, camera)
}
```

### AnimationAction properties
```javascript
action.play()
action.pause()
action.stop()
action.reset()
action.setLoop(THREE.LoopOnce | THREE.LoopRepeat | THREE.LoopPingPong, count)
action.clampWhenFinished = true  // hold last frame when LoopOnce finishes
action.timeScale = 2.0           // 2x speed (negative = reverse)
action.weight = 1.0              // blend weight (0–1)
action.time = 0                  // current time in clip

// Events
mixer.addEventListener('finished', (e) => {
  console.log('Animation finished:', e.action.getClip().name)
})
```

### Manual animation (no glTF)
```javascript
// Using GSAP for smooth transitions (install: npm i gsap)
import gsap from 'gsap'

gsap.to(mesh.position, {
  y: 3,
  duration: 1.5,
  ease: 'power2.inOut',
  repeat: -1,
  yoyo: true
})

gsap.to(mesh.rotation, {
  y: Math.PI * 2,
  duration: 2,
  ease: 'none',
  repeat: -1
})

// MorphTargets (shape keys — blend between shapes)
geometry.morphAttributes.position = [...]
mesh.morphTargetInfluences[0] = 0.5  // 50% morph
```

---

## 9. Model Loading

### GLTFLoader
```javascript
// GLTF/GLB is the standard 3D format for web
// .gltf = JSON + separate .bin + textures
// .glb = binary, self-contained (preferred)

import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'

const loader = new GLTFLoader()
loader.load(
  '/models/robot.glb',
  (gltf) => {
    const model = gltf.scene       // Object3D (scene graph)
    const animations = gltf.animations  // AnimationClip[]
    const cameras = gltf.cameras
    const scenes = gltf.scenes
    scene.add(model)
  },
  (progress) => console.log(`${progress.loaded / progress.total * 100}%`),
  (error) => console.error(error)
)

// Traverse model to configure shadows etc.
model.traverse((child) => {
  if (child.isMesh) {
    child.castShadow = true
    child.receiveShadow = true
    // Access material
    child.material.envMapIntensity = 1.0
  }
})
```

### Draco Compression
```bash
# Models can be compressed with Draco (up to 10x smaller)
# In Blender: export glTF with Draco compression enabled
```
```javascript
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js'

const dracoLoader = new DRACOLoader()
// Copy decoder from node_modules/three/examples/jsm/libs/draco/ to public/draco/
dracoLoader.setDecoderPath('/draco/')

const gltfLoader = new GLTFLoader()
gltfLoader.setDRACOLoader(dracoLoader)
```

### KTX2 Textures (GPU-compressed)
```javascript
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js'

const ktx2Loader = new KTX2Loader()
ktx2Loader.setTranscoderPath('/basis/')  // copy from three/examples/jsm/libs/basis/
ktx2Loader.detectSupport(renderer)

gltfLoader.setKTX2Loader(ktx2Loader)
// Now glTF models with KTX2 textures load correctly
```

---

## 10. Particles

```javascript
// Basic point cloud
const count = 10000
const positions = new Float32Array(count * 3)

for (let i = 0; i < count; i++) {
  positions[i * 3]     = (Math.random() - 0.5) * 20  // x
  positions[i * 3 + 1] = (Math.random() - 0.5) * 20  // y
  positions[i * 3 + 2] = (Math.random() - 0.5) * 20  // z
}

const geometry = new THREE.BufferGeometry()
geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

const material = new THREE.PointsMaterial({
  size: 0.05,                // point size in world units
  sizeAttenuation: true,     // smaller when farther away
  color: 0x4ecdc4,
  map: particleTexture,      // sprite texture (circle, star, etc.)
  alphaMap: particleTexture,
  transparent: true,
  depthWrite: false,         // IMPORTANT: prevents particles fighting each other
  blending: THREE.AdditiveBlending,  // additive = brighter where overlapping
  vertexColors: true,        // use colors array
})

const particles = new THREE.Points(geometry, material)
scene.add(particles)

// Animate particles
const positionArray = geometry.attributes.position.array
function animate() {
  const elapsed = clock.getElapsedTime()
  for (let i = 0; i < count; i++) {
    const y = i * 3 + 1
    positionArray[y] = Math.sin(elapsed + i * 0.1) * 0.3
  }
  geometry.attributes.position.needsUpdate = true  // REQUIRED
}
```

---

## 11. Lines

```javascript
// Simple Line
const points = [
  new THREE.Vector3(0, 0, 0),
  new THREE.Vector3(1, 0, 0),
  new THREE.Vector3(1, 1, 0),
]
const geometry = new THREE.BufferGeometry().setFromPoints(points)
const line = new THREE.Line(geometry, new THREE.LineBasicMaterial({ color: 0xff0000 }))
scene.add(line)

// LineSegments (pairs of points)
const segGeom = new THREE.BufferGeometry().setFromPoints([
  new THREE.Vector3(0,0,0), new THREE.Vector3(1,0,0),  // segment 1
  new THREE.Vector3(0,1,0), new THREE.Vector3(1,1,0),  // segment 2
])
const segments = new THREE.LineSegments(segGeom, new THREE.LineBasicMaterial())
scene.add(segments)

// Dashed line
const dashedLine = new THREE.Line(geometry, new THREE.LineDashedMaterial({
  color: 0x00ff00,
  dashSize: 0.1,
  gapSize: 0.05,
}))
dashedLine.computeLineDistances()  // REQUIRED for dashed to work
scene.add(dashedLine)

// Fat lines (Line2) — variable width, stays thick at any zoom
import { Line2 } from 'three/addons/lines/Line2.js'
import { LineMaterial } from 'three/addons/lines/LineMaterial.js'
import { LineGeometry } from 'three/addons/lines/LineGeometry.js'

const lineGeom = new LineGeometry()
lineGeom.setPositions([0,0,0, 1,0,0, 1,1,0])  // flat array

const lineMat = new LineMaterial({
  color: 0xff0000,
  linewidth: 5,  // pixels
  resolution: new THREE.Vector2(innerWidth, innerHeight),
})

const fatLine = new Line2(lineGeom, lineMat)
scene.add(fatLine)
```

---

## 12. Sprites & Billboard

```javascript
// Sprites always face the camera (billboard)
const spriteMat = new THREE.SpriteMaterial({
  map: new THREE.TextureLoader().load('/icons/marker.png'),
  color: 0xffffff,
  sizeAttenuation: false,  // same size regardless of distance
})
const sprite = new THREE.Sprite(spriteMat)
sprite.scale.set(0.1, 0.1, 1)
sprite.position.set(0, 2, 0)
scene.add(sprite)
```

---

## 13. Raycaster (Click/Hover Detection)

```javascript
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()

// Set raycaster precision for Points
raycaster.params.Points.threshold = 0.1

window.addEventListener('pointermove', (e) => {
  // NDC coordinates (-1 to +1)
  mouse.x = (e.clientX / innerWidth) * 2 - 1
  mouse.y = -(e.clientY / innerHeight) * 2 + 1
})

function checkIntersections() {
  raycaster.setFromCamera(mouse, camera)

  // Check against specific objects
  const intersects = raycaster.intersectObjects(clickableObjects, true)  // true = recursive (checks children)

  if (intersects.length > 0) {
    const hit = intersects[0]
    console.log(hit.object)      // intersected mesh
    console.log(hit.point)       // 3D point of intersection
    console.log(hit.distance)    // distance from camera
    console.log(hit.face)        // triangle face
    console.log(hit.uv)          // UV at intersection point
    console.log(hit.instanceId)  // for InstancedMesh
  }
}

// Call in animate loop or on mousemove
```

---

## 14. Fog

```javascript
// Linear fog — linear interpolation from near to far
scene.fog = new THREE.Fog(0x1a1a2e, 5, 50)  // color, near, far

// Exponential fog — more realistic
scene.fog = new THREE.FogExp2(0x1a1a2e, 0.05)  // color, density

// Make sky match fog color
scene.background = new THREE.Color(0x1a1a2e)
```

---

## 15. Render Targets (Off-screen Rendering)

```javascript
// Render to texture (for reflections, portals, shadow maps, post-processing)
const renderTarget = new THREE.WebGLRenderTarget(1024, 1024, {
  minFilter: THREE.LinearFilter,
  magFilter: THREE.LinearFilter,
  format: THREE.RGBAFormat,
})

// Render scene B into render target
renderer.setRenderTarget(renderTarget)
renderer.render(scenB, cameraB)
renderer.setRenderTarget(null)  // back to screen

// Use render target texture on a mesh
material.map = renderTarget.texture

// Resize on window resize
renderTarget.setSize(newWidth, newHeight)

// Dispose when done
renderTarget.dispose()
```

---

## 16. Performance Patterns

```javascript
// 1. Reuse geometries and materials (don't create inside loops)
const sharedGeometry = new THREE.BoxGeometry()
const sharedMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 })

for (let i = 0; i < 100; i++) {
  const mesh = new THREE.Mesh(sharedGeometry, sharedMaterial)
  // multiple meshes share geometry/material — only ONE uploaded to GPU
  scene.add(mesh)
}

// 2. Merge static geometries into one draw call
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js'

const merged = mergeGeometries([geomA, geomB, geomC])
const mergedMesh = new THREE.Mesh(merged, material)
// 3 meshes → 1 draw call

// 3. Frustum culling — automatic but needs correct bounding sphere
geometry.computeBoundingSphere()
geometry.computeBoundingBox()
mesh.frustumCulled = true  // default

// 4. LOD (Level of Detail)
const lod = new THREE.LOD()
lod.addLevel(highDetailMesh, 0)   // distance < this → use this mesh
lod.addLevel(medDetailMesh, 10)
lod.addLevel(lowDetailMesh, 50)
scene.add(lod)

// 5. Stats panel
import Stats from 'three/addons/libs/stats.module.js'
const stats = new Stats()
document.body.appendChild(stats.dom)
function animate() {
  stats.begin()
  renderer.render(scene, camera)
  stats.end()
}

// 6. Dispose properly to avoid memory leaks
function disposeMesh(mesh) {
  mesh.geometry.dispose()
  if (Array.isArray(mesh.material)) {
    mesh.material.forEach(m => m.dispose())
  } else {
    mesh.material.dispose()
  }
  scene.remove(mesh)
}
```

---

## 17. Complete Render Loop Pattern

```javascript
const clock = new THREE.Clock()
let animationId

function animate() {
  animationId = requestAnimationFrame(animate)

  const delta = clock.getDelta()     // seconds since last frame (typically ~0.016 for 60fps)
  const elapsed = clock.getElapsedTime() // total elapsed seconds

  // Delta-based movement (frame-rate independent)
  mesh.rotation.y += delta * Math.PI  // PI radians per second regardless of FPS

  controls.update()     // if using OrbitControls with enableDamping
  if (mixer) mixer.update(delta)
  composer ? composer.render() : renderer.render(scene, camera)
}

animate()

// Stop loop
function stop() {
  cancelAnimationFrame(animationId)
}
```
