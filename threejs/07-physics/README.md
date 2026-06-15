# Phase 7 — Physics (Rapier + Three.js)

> Rapier là physics engine tốt nhất hiện tại cho web — compiled từ Rust → WASM, nhanh hơn Cannon.js 5–10x

## Setup

```bash
npm install @dimforge/rapier3d-compat        # vanilla Three.js
npm install @react-three/rapier              # React Three Fiber wrapper
```

---

## Vanilla Three.js + Rapier

```javascript
import RAPIER from '@dimforge/rapier3d-compat'

async function initPhysics() {
  await RAPIER.init()  // load WASM

  // 1. Create physics world
  const world = new RAPIER.World({ x: 0, y: -9.81, z: 0 })  // gravity

  // 2. Create ground (static body)
  const groundBody = world.createRigidBody(
    RAPIER.RigidBodyDesc.fixed()
      .setTranslation(0, -0.5, 0)
  )
  world.createCollider(
    RAPIER.ColliderDesc.cuboid(10, 0.5, 10),  // half-extents
    groundBody
  )

  // 3. Create dynamic box
  const boxBody = world.createRigidBody(
    RAPIER.RigidBodyDesc.dynamic()
      .setTranslation(0, 5, 0)
  )
  world.createCollider(
    RAPIER.ColliderDesc.cuboid(0.5, 0.5, 0.5),
    boxBody
  )

  // 4. Sync physics → Three.js in animation loop
  const clock = new THREE.Clock()
  const bodies = [{ body: boxBody, mesh: boxMesh }]

  function animate() {
    requestAnimationFrame(animate)

    // Step physics (fixed timestep)
    world.step()

    // Sync mesh positions from physics
    for (const { body, mesh } of bodies) {
      const pos = body.translation()
      const rot = body.rotation()

      mesh.position.set(pos.x, pos.y, pos.z)
      mesh.quaternion.set(rot.x, rot.y, rot.z, rot.w)
    }

    renderer.render(scene, camera)
  }
  animate()

  // Apply force/impulse
  boxBody.applyImpulse({ x: 0, y: 5, z: 0 }, true)
  boxBody.applyTorqueImpulse({ x: 0, y: 1, z: 0 }, true)
}

initPhysics()
```

---

## React Three Fiber + @react-three/rapier

```tsx
import { Canvas } from '@react-three/fiber'
import { Physics, RigidBody, CuboidCollider } from '@react-three/rapier'

export default function App() {
  return (
    <Canvas>
      <Physics gravity={[0, -9.81, 0]} debug>
        {/* Static ground */}
        <RigidBody type="fixed">
          <mesh>
            <boxGeometry args={[20, 0.5, 20]} />
            <meshStandardMaterial color="gray" />
          </mesh>
        </RigidBody>

        {/* Dynamic falling boxes */}
        {Array.from({ length: 10 }, (_, i) => (
          <RigidBody key={i} position={[i-5, i*2+5, 0]}>
            <mesh>
              <boxGeometry args={[1, 1, 1]} />
              <meshStandardMaterial color="orange" />
            </mesh>
          </RigidBody>
        ))}
      </Physics>
    </Canvas>
  )
}
```

---

## RigidBody Types

```tsx
// Fixed — doesn't move (ground, walls, static obstacles)
<RigidBody type="fixed" />

// Dynamic — moved by physics (falling objects, robots with collisions)
<RigidBody type="dynamic" />

// KinematicPositionBased — you control position, physics reacts to it
// Good for: player character, elevator, controlled robot movement
<RigidBody type="kinematicPosition" />

// KinematicVelocityBased — you control velocity
<RigidBody type="kinematicVelocity" />
```

---

## Collider Shapes

```tsx
// Auto-detect from mesh geometry (Trimesh — accurate but slow)
<RigidBody colliders="trimesh">
  <mesh>...</mesh>
</RigidBody>

// Hull — convex hull approximation (faster)
<RigidBody colliders="hull">
  <mesh>...</mesh>
</RigidBody>

// Ball
<RigidBody colliders="ball">
  <mesh>...</mesh>
</RigidBody>

// Cuboid
<RigidBody colliders="cuboid">
  <mesh>...</mesh>
</RigidBody>

// Manual collider (doesn't follow mesh shape, just logical zone)
<RigidBody colliders={false}>
  <mesh>...</mesh>
  <CuboidCollider args={[0.5, 0.5, 0.5]} />
</RigidBody>

// Sensor — detects overlap but no physics response (trigger zones)
<RigidBody type="fixed">
  <CuboidCollider args={[2, 1, 2]} sensor onIntersectionEnter={handleEnter} />
</RigidBody>
```

---

## Useful Physics Interactions

```tsx
import { useRef } from 'react'
import { RigidBody } from '@react-three/rapier'

function PhysicsBox() {
  const bodyRef = useRef()

  const handleClick = () => {
    // Apply upward impulse on click
    bodyRef.current?.applyImpulse({ x: 0, y: 10, z: 0 })
    bodyRef.current?.applyTorqueImpulse({ x: 0, y: 2, z: 0 })
  }

  return (
    <RigidBody ref={bodyRef} position={[0, 5, 0]}>
      <mesh onClick={handleClick}>
        <boxGeometry />
        <meshStandardMaterial color="teal" />
      </mesh>
    </RigidBody>
  )
}

// Kinematic robot movement (controlled, but physically correct)
function Robot({ targetPosition }) {
  const bodyRef = useRef()

  useFrame(() => {
    if (!bodyRef.current) return

    const current = bodyRef.current.translation()
    const dx = targetPosition.x - current.x
    const dz = targetPosition.z - current.z

    // Move toward target at constant speed
    const speed = 3.0
    bodyRef.current.setNextKinematicTranslation({
      x: current.x + Math.sign(dx) * Math.min(Math.abs(dx), speed * delta),
      y: current.y,
      z: current.z + Math.sign(dz) * Math.min(Math.abs(dz), speed * delta),
    })
  })

  return (
    <RigidBody ref={bodyRef} type="kinematicPosition" colliders="cuboid">
      <mesh>
        <boxGeometry args={[0.5, 0.5, 0.5]} />
        <meshStandardMaterial color="blue" />
      </mesh>
    </RigidBody>
  )
}
```

---

## Collision Events

```tsx
<RigidBody
  onCollisionEnter={({ other }) => {
    console.log('Collision with:', other.rigidBodyObject?.name)
  }}
  onCollisionExit={() => {
    console.log('Collision ended')
  }}
  onIntersectionEnter={({ other }) => {
    // For sensors
    console.log('Entered sensor zone')
  }}
>
  <mesh>...</mesh>
</RigidBody>
```

---

## Physics Performance Tips

```
- Rapier is VERY fast (Rust/WASM), but still has cost
- Fewer colliders = better performance
- Use simple collider shapes (cuboid/ball) over trimesh
- Kinematic bodies for controlled objects (robots following paths)
- Fixed bodies for static world (no physics computation)
- Disable physics for very distant objects
- Fixed timestep (Rapier default) = deterministic simulation
```

---

## When to Use Physics vs Manual Animation

| Scenario | Use |
|---|---|
| Robot following predefined path | Manual animation (lerp/tween) |
| Robot avoiding obstacles dynamically | Physics (kinematic + raycasting) |
| Falling boxes, debris | Physics (dynamic) |
| Conveyor belts | Physics (kinematic) |
| Trigger zones (enter/leave area) | Physics sensor |
| Smooth camera movement | GSAP / lerp (no physics) |

For most warehouse digital twin robots: **kinematic physics** — you control the path, physics handles collision detection.
