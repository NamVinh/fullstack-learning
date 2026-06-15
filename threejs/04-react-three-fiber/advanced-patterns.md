# R3F Advanced Patterns

## 1. useFrame — Performance Rules

```tsx
import { useRef } from 'react'
import { useFrame } from '@react-three/fiber'

// RULE: Never call React setState inside useFrame
// setState → re-render → slow (60x per second!)
// Instead: use refs to mutate directly

function AnimatedBox() {
  const meshRef = useRef()
  const materialRef = useRef()

  useFrame((state, delta) => {
    // ✅ Direct mutation via ref (no React re-render)
    meshRef.current.rotation.y += delta
    materialRef.current.color.setHSL(state.clock.elapsedTime * 0.1 % 1, 0.8, 0.5)

    // ❌ Never do this:
    // setRotation(rotation + delta)  // 60 state updates/second = death
  })

  return (
    <mesh ref={meshRef}>
      <boxGeometry />
      <meshStandardMaterial ref={materialRef} />
    </mesh>
  )
}

// Access camera, scene, gl (renderer) from useFrame state
useFrame(({ camera, scene, gl, clock, pointer, raycaster }) => {
  camera.position.lerp(targetPos, 0.05)
  camera.lookAt(0, 0, 0)
})
```

---

## 2. Context (pass data without props drilling)

```tsx
import { createContext, useContext, useRef } from 'react'
import { useFrame } from '@react-three/fiber'

const WarehouseContext = createContext(null)

export function WarehouseProvider({ children }) {
  const selectedRobotRef = useRef(null)

  return (
    <WarehouseContext.Provider value={{ selectedRobotRef }}>
      {children}
    </WarehouseContext.Provider>
  )
}

export const useWarehouse = () => useContext(WarehouseContext)

// Deep child can access without props drilling:
function RobotLabel() {
  const { selectedRobotRef } = useWarehouse()
  // ...
}
```

---

## 3. forwardRef with R3F

```tsx
import { forwardRef } from 'react'
import { Mesh } from 'three'

const Robot = forwardRef<Mesh, { position: [number, number, number] }>(
  ({ position }, ref) => {
    return (
      <mesh ref={ref} position={position}>
        <boxGeometry args={[0.5, 0.5, 0.5]} />
        <meshStandardMaterial color="teal" />
      </mesh>
    )
  }
)

// Parent can get direct access to the THREE.Mesh:
function Fleet() {
  const robot0Ref = useRef<Mesh>(null)

  useEffect(() => {
    robot0Ref.current?.position.set(5, 0, 5)
  }, [])

  return <Robot ref={robot0Ref} position={[0, 0, 0]} />
}
```

---

## 4. Portals — render into different scene

```tsx
import { createPortal } from '@react-three/fiber'

// Useful for: HUD elements, minimap, separate render targets
function HUD() {
  const [hudScene] = useState(() => new THREE.Scene())
  const hudCamera = useRef()

  useFrame(({ gl, camera }) => {
    gl.autoClear = false
    gl.clearDepth()
    gl.render(hudScene, hudCamera.current)
  })

  return createPortal(
    <>
      <orthographicCamera ref={hudCamera} args={[-1, 1, 1, -1, 0.1, 10]} position={[0, 0, 5]} />
      <mesh position={[0.8, 0.8, 0]}>
        <planeGeometry args={[0.3, 0.1]} />
        <meshBasicMaterial color="black" transparent opacity={0.7} />
      </mesh>
    </>,
    hudScene
  )
}
```

---

## 5. Teleport / Conditional Rendering

```tsx
// DON'T conditionally render 3D objects that frequently toggle
// (mount/unmount creates/destroys GPU resources)

// ✅ Visibility toggle (keeps GPU resources, just hides)
function Robot({ visible }) {
  return (
    <group visible={visible}>
      <mesh>...</mesh>
    </group>
  )
}

// ✅ Move off-screen when "hidden"
function Robot({ active }) {
  return (
    <mesh position={active ? [0, 0, 0] : [0, -1000, 0]}>
      ...
    </mesh>
  )
}

// ❌ Conditional mount (expensive for frequent toggles)
{isVisible && <Robot />}
```

---

## 6. Suspense for Asset Loading

```tsx
import { Suspense } from 'react'
import { useGLTF, useTexture } from '@react-three/drei'

function Model() {
  const { scene } = useGLTF('/robot.glb')  // suspends until loaded
  return <primitive object={scene} />
}

function Textured() {
  const [colorMap, normalMap] = useTexture(['/color.jpg', '/normal.jpg'])  // parallel load
  return (
    <mesh>
      <meshStandardMaterial map={colorMap} normalMap={normalMap} />
    </mesh>
  )
}

// Preload assets before render
useGLTF.preload('/robot.glb')

// Wrap in Suspense with fallback
<Suspense fallback={<LoadingScreen />}>
  <Model />
  <Textured />
</Suspense>
```

---

## 7. Custom R3F Extension (extend)

```tsx
import { extend } from '@react-three/fiber'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

// Extend allows using non-standard Three.js objects as JSX
extend({ OrbitControls })

function Controls() {
  const { camera, gl } = useThree()
  return <orbitControls args={[camera, gl.domElement]} enableDamping />
}
// This is essentially what @react-three/drei does internally
```

---

## 8. Canvas Props & Configuration

```tsx
<Canvas
  // Camera
  camera={{ position: [0, 5, 10], fov: 75, near: 0.1, far: 1000 }}

  // Renderer
  gl={{
    antialias: true,
    toneMapping: THREE.ACESFilmicToneMapping,
    outputColorSpace: THREE.SRGBColorSpace,
    shadowMap: { enabled: true, type: THREE.PCFSoftShadowMap },
  }}

  // Shadow
  shadows  // shorthand for shadows={true}

  // Performance
  dpr={[1, 2]}     // min/max pixel ratio (responsive)
  frameloop="demand"  // only render when invalidated (saves power)
  // frameloop="always" (default — constant 60fps)
  // frameloop="never" — manual rendering

  // Events
  onCreated={({ gl, scene, camera }) => {
    // Access renderer after creation
  }}

  // Resize
  resize={{ scroll: true, debounce: { scroll: 50, resize: 0 } }}
>
```

---

## 9. useThree — Access R3F internals

```tsx
import { useThree } from '@react-three/fiber'

function MyComponent() {
  const {
    camera,         // the camera
    gl,             // renderer (THREE.WebGLRenderer)
    scene,          // the scene
    size,           // { width, height, top, left }
    viewport,       // { width, height, factor } in 3D units
    pointer,        // mouse position in NDC (-1 to 1)
    raycaster,      // shared raycaster
    clock,          // THREE.Clock
    performance,    // { current, min, max, debounce, regress() }
    set,            // modify state
    get,            // read state outside React
    invalidate,     // trigger re-render (for frameloop="demand")
    advance,        // manual render step
  } = useThree()

  // Viewport → 3D units (useful for UI positioning)
  const { width, height } = viewport
  // A mesh at position (width/2, 0, 0) is at the right edge of screen

  return null
}
```

---

## 10. R3F + WebSocket for Real-time Data

```tsx
// This is the core pattern for Digital Twin
import { useEffect, useRef } from 'react'
import { useFrame } from '@react-three/fiber'
import { useRobotStore } from '../store/robotStore'

function RealTimeRobots() {
  const updateRobot = useRobotStore(s => s.updateRobot)
  const robots = useRobotStore(s => s.robots)
  const meshRef = useRef()

  // Connect WebSocket
  useEffect(() => {
    const ws = new WebSocket('ws://warehouse-api/robots')

    ws.onmessage = (e) => {
      const data = JSON.parse(e.data)
      // data: { robotId, x, z, rotation, status, battery }
      updateRobot(data)
    }

    return () => ws.close()
  }, [])

  // Sync to InstancedMesh every frame
  const dummy = useRef(new THREE.Object3D())
  useFrame(() => {
    if (!meshRef.current) return
    robots.forEach((robot, i) => {
      dummy.current.position.set(robot.x, 0.25, robot.z)
      dummy.current.rotation.y = robot.rotation
      dummy.current.updateMatrix()
      meshRef.current.setMatrixAt(i, dummy.current.matrix)
    })
    meshRef.current.instanceMatrix.needsUpdate = true
  })

  return (
    <instancedMesh ref={meshRef} args={[null, null, robots.length]} castShadow>
      <boxGeometry args={[0.4, 0.5, 0.6]} />
      <meshStandardMaterial />
    </instancedMesh>
  )
}
```

---

## 11. Testing R3F Components

```tsx
// @testing-library/react + @react-three/test-renderer
import { act, create } from '@react-three/test-renderer'

test('robot renders at correct position', async () => {
  const renderer = await act(() =>
    create(
      <Canvas>
        <Robot position={[1, 0, 2]} />
      </Canvas>
    )
  )

  const mesh = renderer.scene.findByType('Mesh')
  expect(mesh.props.position).toEqual([1, 0, 2])
})
```

---

## Package Versions (as of 2025)

```json
{
  "three": "^0.170.0",
  "@react-three/fiber": "^8.17.0",
  "@react-three/drei": "^9.115.0",
  "@react-three/postprocessing": "^2.16.0",
  "@react-three/rapier": "^1.5.0",
  "leva": "^0.9.35",
  "gsap": "^3.12.5"
}
```
