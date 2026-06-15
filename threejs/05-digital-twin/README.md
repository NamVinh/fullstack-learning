# Phase 5 — Digital Twin / Warehouse Simulation

## Architecture Overview

```
WebSocket / REST API (telemetry data)
         ↓
    Zustand Store (state management)
         ↓
    React Three Fiber (3D rendering)
         ↓
    InstancedMesh (100+ robots at 60fps)
```

## Project: Mini Warehouse Digital Twin

Target: tạo demo này để portfolio khi apply role 3D Web Engineer.

### Features
- 10x10 grid warehouse floor
- 20+ animated robots (InstancedMesh)
- Real-time telemetry panel (click robot → show data)
- WebSocket mock feed (hoặc setInterval simulation)
- Floor heatmap (custom shader)
- Camera: orbit + top-down toggle

### Folder structure
```
warehouse-twin/
  src/
    components/
      Scene.tsx          -- Canvas + lighting + controls
      WarehouseFloor.tsx -- Grid floor + heatmap shader
      RobotFleet.tsx     -- InstancedMesh for all robots
      TelemetryPanel.tsx -- HTML sidebar (react, not 3D)
    store/
      robotStore.ts      -- Zustand: robot positions, status
      telemetryStore.ts  -- Zustand: selected robot data
    hooks/
      useRobotSimulator.ts -- mock WebSocket feed
    shaders/
      heatmap.vert.glsl
      heatmap.frag.glsl
    App.tsx
```

## Key Concepts

### InstancedMesh cho robots

```tsx
// RobotFleet.tsx
import { useRef, useEffect } from 'react'
import { useFrame } from '@react-three/fiber'
import { InstancedMesh, Matrix4, Object3D } from 'three'
import { useRobotStore } from '../store/robotStore'

const dummy = new Object3D()

export function RobotFleet() {
  const meshRef = useRef<InstancedMesh>(null)
  const robots = useRobotStore(s => s.robots)

  useFrame(() => {
    if (!meshRef.current) return

    robots.forEach((robot, i) => {
      dummy.position.set(robot.x, 0.2, robot.z)
      dummy.rotation.y = robot.rotation
      dummy.updateMatrix()
      meshRef.current!.setMatrixAt(i, dummy.matrix)

      // Color based on status
      const color = robot.status === 'idle' ? idleColor : activeColor
      meshRef.current!.setColorAt(i, color)
    })

    meshRef.current.instanceMatrix.needsUpdate = true
    if (meshRef.current.instanceColor) {
      meshRef.current.instanceColor.needsUpdate = true
    }
  })

  return (
    <instancedMesh
      ref={meshRef}
      args={[undefined, undefined, robots.length]}
      castShadow
      onClick={(e) => {
        const robotId = robots[e.instanceId!]?.id
        if (robotId) useRobotStore.getState().selectRobot(robotId)
      }}
    >
      <boxGeometry args={[0.4, 0.3, 0.6]} />
      <meshStandardMaterial />
    </instancedMesh>
  )
}
```

### Mock WebSocket simulator

```typescript
// hooks/useRobotSimulator.ts
import { useEffect } from 'react'
import { useRobotStore } from '../store/robotStore'

export function useRobotSimulator() {
  const updateRobot = useRobotStore(s => s.updateRobot)

  useEffect(() => {
    const robots = Array.from({ length: 20 }, (_, i) => ({
      id: `robot-${i}`,
      x: (Math.random() - 0.5) * 18,
      z: (Math.random() - 0.5) * 18,
      targetX: 0,
      targetZ: 0,
      speed: 0.02 + Math.random() * 0.03,
      status: 'idle' as const,
      battery: 80 + Math.floor(Math.random() * 20),
      task: 'IDLE',
    }))

    // Initialize
    robots.forEach(r => updateRobot(r))

    // Simulate movement
    const interval = setInterval(() => {
      robots.forEach(robot => {
        // Random new target every few seconds
        if (Math.random() < 0.01) {
          robot.targetX = (Math.random() - 0.5) * 18
          robot.targetZ = (Math.random() - 0.5) * 18
          robot.task = ['PICK', 'DELIVER', 'CHARGE', 'IDLE'][Math.floor(Math.random() * 4)]
        }

        // Move toward target
        robot.x += (robot.targetX - robot.x) * robot.speed
        robot.z += (robot.targetZ - robot.z) * robot.speed
        robot.status = Math.abs(robot.targetX - robot.x) > 0.1 ? 'moving' : 'idle'
        robot.battery = Math.max(0, robot.battery - 0.001)

        updateRobot(robot)
      })
    }, 16) // ~60fps

    return () => clearInterval(interval)
  }, [])
}
```

### Heatmap Shader (floor activity visualization)

```glsl
// heatmap.frag.glsl
uniform sampler2D uHeatmapData; // texture encoding activity per grid cell
uniform float uIntensity;
varying vec2 vUv;

vec3 heatmapColor(float t) {
  // blue → cyan → green → yellow → red
  vec3 colors[5];
  colors[0] = vec3(0.0, 0.0, 1.0);   // blue
  colors[1] = vec3(0.0, 1.0, 1.0);   // cyan
  colors[2] = vec3(0.0, 1.0, 0.0);   // green
  colors[3] = vec3(1.0, 1.0, 0.0);   // yellow
  colors[4] = vec3(1.0, 0.0, 0.0);   // red

  t = clamp(t, 0.0, 1.0) * 4.0;
  int i = int(t);
  float f = fract(t);
  return mix(colors[i], colors[min(i + 1, 4)], f);
}

void main() {
  float activity = texture2D(uHeatmapData, vUv).r * uIntensity;
  vec3 floorColor = vec3(0.1, 0.1, 0.15);
  vec3 heat = heatmapColor(activity);
  vec3 color = mix(floorColor, heat, activity * 0.6);
  gl_FragColor = vec4(color, 1.0);
}
```

### Telemetry Sidebar (React HTML)

```tsx
// TelemetryPanel.tsx
export function TelemetryPanel() {
  const selectedId = useRobotStore(s => s.selectedRobotId)
  const robot = useRobotStore(s => s.robots.find(r => r.id === selectedId))

  if (!robot) return null

  return (
    <div style={{
      position: 'absolute',
      top: 0,
      right: 0,
      width: 260,
      height: '100%',
      background: 'rgba(10, 10, 20, 0.9)',
      color: 'white',
      padding: 20,
      fontFamily: 'monospace',
    }}>
      <h3>{robot.id}</h3>
      <div>Status: <span style={{ color: robot.status === 'moving' ? '#4ecdc4' : '#aaa' }}>{robot.status}</span></div>
      <div>Task: {robot.task}</div>
      <div>Battery: {robot.battery.toFixed(1)}%</div>
      <div>Position: ({robot.x.toFixed(2)}, {robot.z.toFixed(2)})</div>
      <div>Speed: {robot.speed.toFixed(3)} m/s</div>
    </div>
  )
}
```

## Performance Checklist

- [ ] InstancedMesh thay vì individual Mesh (1000+ → vẫn 60fps)
- [ ] `useFrame` thay vì `useState` cho animations (không re-render React)
- [ ] `useMemo` cho geometry/material (không re-create mỗi render)
- [ ] `dispose()` geometry/material khi unmount
- [ ] Frustum culling (Three.js tự làm)
- [ ] LOD cho model phức tạp nếu có
- [ ] `renderer.setPixelRatio(Math.min(devicePixelRatio, 2))` — cap ở 2x
