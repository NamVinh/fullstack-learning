# Phase 1 — 3D Math Basics

Không cần học như toán đại học — chỉ cần hiểu đủ để dùng Three.js.

## Coordinate System

Three.js dùng **right-hand coordinate system**:
- X → phải
- Y → lên
- Z → ra phía trước (về phía người xem)

```
     Y
     |
     |___X
    /
   Z
```

## Vectors

```javascript
import * as THREE from 'three'

const v1 = new THREE.Vector3(1, 0, 0) // hướng X
const v2 = new THREE.Vector3(0, 1, 0) // hướng Y

// Length (magnitude)
console.log(v1.length()) // 1

// Normalize (make length = 1)
const dir = new THREE.Vector3(3, 4, 0).normalize() // (0.6, 0.8, 0)

// Dot product — nếu = 0 thì 2 vector vuông góc
const dot = v1.dot(v2) // 0

// Cross product — tạo vector vuông góc với cả 2
const cross = v1.clone().cross(v2) // (0, 0, 1) — hướng Z
```

## Position, Rotation, Scale

```javascript
const mesh = new THREE.Mesh(geometry, material)

// Position
mesh.position.set(1, 2, 3)
mesh.position.x = 5

// Rotation (Euler angles, radians)
mesh.rotation.y = Math.PI / 4 // 45 degrees

// Scale
mesh.scale.set(2, 2, 2) // 2x bigger

// Matrix (computed automatically from position/rotation/scale)
mesh.updateMatrix()
console.log(mesh.matrix)
```

## Euler vs Quaternion

```javascript
// Euler — dễ đọc, nhưng bị "gimbal lock"
mesh.rotation.set(Math.PI/4, Math.PI/2, 0) // x, y, z in radians

// Quaternion — không bị gimbal lock, dùng cho smooth animation
const q = new THREE.Quaternion()
q.setFromEuler(new THREE.Euler(0, Math.PI/2, 0))
mesh.quaternion.copy(q)

// Slerp — interpolate between 2 rotations smoothly
const start = new THREE.Quaternion()
const end = new THREE.Quaternion().setFromAxisAngle(new THREE.Vector3(0,1,0), Math.PI)
start.slerp(end, 0.5) // 50% rotation
```

## Camera Projection

```javascript
// Perspective — như mắt người, vật xa nhỏ hơn
const camera = new THREE.PerspectiveCamera(
  75,                   // fov: field of view (degrees)
  window.innerWidth / window.innerHeight, // aspect ratio
  0.1,                  // near clipping plane
  1000                  // far clipping plane
)

// Orthographic — không có perspective, dùng cho 2D/UI/map views
const camera2 = new THREE.OrthographicCamera(
  -10, 10, // left, right
  10, -10, // top, bottom
  0.1, 1000
)
```

## UV Coordinates

UV = texture coordinates, từ (0,0) bottom-left đến (1,1) top-right

```javascript
// PlaneGeometry tự tạo UVs
const plane = new THREE.PlaneGeometry(1, 1)
// plane.attributes.uv chứa UV data

// Custom UV trên BufferGeometry
const uvs = new Float32Array([
  0, 0,   // vertex 0
  1, 0,   // vertex 1
  0, 1,   // vertex 2
  1, 1,   // vertex 3
])
geometry.setAttribute('uv', new THREE.BufferAttribute(uvs, 2))
```

## Quick Reference

| Concept | Three.js Class | Common use |
|---|---|---|
| Position/Direction | `Vector3` | movement, raycasting |
| Rotation | `Euler` / `Quaternion` | object rotation |
| Transform | `Matrix4` | combined transform |
| Color | `Color` | material colors |
| Camera space | `PerspectiveCamera` | scene rendering |
