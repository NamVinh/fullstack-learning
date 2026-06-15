# Phase 4 — React Three Fiber

## Setup

```bash
npm create vite@latest my-3d-app -- --template react-ts
cd my-3d-app
npm install three @react-three/fiber @react-three/drei @react-three/postprocessing
npm install -D @types/three
```

## Basic R3F App

```tsx
// App.tsx
import { Canvas } from '@react-three/fiber'
import { OrbitControls } from '@react-three/drei'
import { Box } from './Box'

export default function App() {
  return (
    <Canvas
      camera={{ position: [3, 3, 5], fov: 75 }}
      shadows
    >
      <ambientLight intensity={0.3} />
      <directionalLight position={[5, 10, 5]} intensity={1} castShadow />
      <Box position={[-1.5, 0.5, 0]} />
      <OrbitControls enableDamping />
    </Canvas>
  )
}
```

## useFrame — Animation Loop

```tsx
import { useRef } from 'react'
import { useFrame } from '@react-three/fiber'
import { Mesh } from 'three'

export function Box({ position }: { position: [number, number, number] }) {
  const meshRef = useRef<Mesh>(null)

  useFrame((state, delta) => {
    if (!meshRef.current) return
    meshRef.current.rotation.y += delta * 0.5  // delta = time since last frame (seconds)
    // state.clock.elapsedTime — total time
    // state.camera — access camera
    // state.scene — access scene
  })

  return (
    <mesh ref={meshRef} position={position} castShadow>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="orange" metalness={0.3} roughness={0.4} />
    </mesh>
  )
}
```

## Events

```tsx
function ClickableBox() {
  const [active, setActive] = useState(false)
  const [hovered, setHovered] = useState(false)

  return (
    <mesh
      onClick={() => setActive(!active)}
      onPointerOver={() => setHovered(true)}
      onPointerOut={() => setHovered(false)}
      scale={active ? 1.5 : 1}
    >
      <boxGeometry />
      <meshStandardMaterial color={hovered ? 'hotpink' : 'orange'} />
    </mesh>
  )
}
```

## Drei Helpers

```tsx
import {
  OrbitControls,   // camera controls
  PerspectiveCamera, // custom camera
  Text,            // 3D text
  Html,            // HTML in 3D space (great for labels/tooltips)
  useGLTF,         // load .glb models
  Environment,     // HDR environment maps
  Sky,             // sky background
  Plane,           // plane with auto-normals
  Box,             // pre-built box
  Sphere,          // pre-built sphere
  Grid,            // grid helper
  Stats,           // performance stats (fps)
  useHelper,       // visualize light/camera helpers
  Instances,       // instanced rendering
  Instance,        // single instance
} from '@react-three/drei'

// HTML overlay on 3D object (perfect for robot labels)
function Robot({ position, name }) {
  return (
    <group position={position}>
      <mesh>
        <boxGeometry args={[0.5, 0.5, 0.5]} />
        <meshStandardMaterial color="teal" />
      </mesh>
      <Html
        position={[0, 0.5, 0]}
        center
        style={{ background: 'rgba(0,0,0,0.8)', color: 'white', padding: '4px 8px', borderRadius: 4 }}
      >
        {name}
      </Html>
    </group>
  )
}

// Load glTF model
function Model() {
  const { scene } = useGLTF('/robot.glb')
  return <primitive object={scene} />
}
```

## Instanced Mesh — 1000+ Objects

```tsx
import { useRef, useMemo } from 'react'
import { useFrame } from '@react-three/fiber'
import { Instances, Instance } from '@react-three/drei'

const ROBOT_COUNT = 100

function Robots() {
  const positions = useMemo(() =>
    Array.from({ length: ROBOT_COUNT }, (_, i) => ({
      x: (i % 10) * 2 - 9,
      z: Math.floor(i / 10) * 2 - 9,
    })),
  [])

  return (
    <Instances limit={ROBOT_COUNT}>
      <boxGeometry args={[0.4, 0.4, 0.4]} />
      <meshStandardMaterial color="teal" />
      {positions.map((pos, i) => (
        <Instance key={i} position={[pos.x, 0.2, pos.z]} />
      ))}
    </Instances>
  )
}
```

## State Management với Zustand

```tsx
// store.ts
import { create } from 'zustand'

interface RobotState {
  robots: { id: string; x: number; z: number; status: string }[]
  selectedRobotId: string | null
  selectRobot: (id: string) => void
}

export const useRobotStore = create<RobotState>((set) => ({
  robots: [],
  selectedRobotId: null,
  selectRobot: (id) => set({ selectedRobotId: id }),
}))

// 3D component
function Robot({ robot }) {
  const selectRobot = useRobotStore(s => s.selectRobot)
  const isSelected = useRobotStore(s => s.selectedRobotId === robot.id)

  return (
    <mesh
      position={[robot.x, 0.2, robot.z]}
      onClick={() => selectRobot(robot.id)}
    >
      <boxGeometry args={[0.4, 0.4, 0.4]} />
      <meshStandardMaterial color={isSelected ? 'yellow' : 'teal'} />
    </mesh>
  )
}
```

## Post-processing

```tsx
import { EffectComposer, Bloom, DepthOfField, Vignette } from '@react-three/postprocessing'

// Inside <Canvas>:
<EffectComposer>
  <Bloom luminanceThreshold={0.8} intensity={1.5} />
  <Vignette darkness={0.5} />
</EffectComposer>
```
