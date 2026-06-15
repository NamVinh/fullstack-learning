# Three.js Learning Roadmap
> From React/TypeScript Developer → 3D Web Engineer

## Folder Structure

```
threejs/
  01-math-basics/
    README.md          ← Vectors, matrices, quaternions, UV coords, coordinate systems
  02-threejs-core/
    README.md          ← Scene graph, Geometry, Materials, Textures, Lighting, Camera,
                          Renderer, Post-processing, AnimationMixer, Model loading,
                          Particles, Lines, Sprites, Raycaster, Fog, Render targets,
                          Performance patterns, Render loop
    hello-world.html         ← Box + sphere + shadows + OrbitControls (run in browser)
    raycaster-click.html     ← Click/hover detection on 3D objects
    particles.html           ← 8000-particle system with custom shader
    instancing-demo.html     ← 1000 robots, 1 draw call — live FPS + draw call counter
  03-shaders-glsl/
    README.md          ← GLSL types, vertex/fragment shader, ShaderMaterial, uniforms
    techniques.md      ← Noise (Perlin, FBM), Fresnel, Dissolve, Hologram, Outline,
                          Vertex displacement, UV scrolling, Normal mapping, GPGPU,
                          SDF shapes, Cel shading, Debug patterns
    shader-demo.html         ← Wave effect + animated rings (run in browser)
  04-react-three-fiber/
    README.md          ← Setup, useFrame, events, Drei helpers, InstancedMesh, Zustand,
                          Post-processing, package versions
    advanced-patterns.md ← useFrame performance rules, Context, forwardRef, Portals,
                           Conditional rendering, Suspense, useThree, WebSocket
                           real-time data, Testing, Canvas props
  05-digital-twin/
    README.md          ← Architecture, InstancedMesh robots, Mock WebSocket simulator,
                          Heatmap shader, Telemetry sidebar, Performance checklist
  06-performance/
    README.md          ← Instancing, Geometry merging, LOD, Frustum culling, Texture
                          optimization, Material optimization, Worker threads, Memory
                          management, Renderer settings, Profiling tools, Draw call targets
  07-physics/
    README.md          ← Rapier setup, RigidBody types, Collider shapes, Sensor triggers,
                          Kinematic robot movement, Collision events, When to use physics
  08-debugging-tools/
    README.md          ← Stats panel, Leva GUI, Helper objects, renderer.info, Spector.js,
                          Chrome DevTools, Common bugs table, console.log patterns
```

---

## Tại sao bạn có lợi thế

- React + TypeScript → React Three Fiber (R3F) dùng React component model cho 3D
- Zustand (bạn đã biết) → state management cho robot fleet
- Performance mindset → draw calls, instancing
- AI tools (Claude Code, Cursor) → tăng tốc boilerplate code
- CI/CD + DevOps → deploy 3D app như web app thông thường

---

## Timeline (6 tháng để apply được role 3D Web Engineer)

| Tháng | Nội dung |
|---|---|
| 1 | Phase 1 (Math) + Phase 2 (Core basics) — chạy các `.html` demo |
| 2 | Phase 2 (advanced: textures, animation, particles) + Phase 3 (shaders basics) |
| 3 | Phase 3 (shader techniques) + Phase 4 (React Three Fiber) |
| 4 | Phase 4 (advanced R3F) + Phase 5 (Digital Twin patterns) |
| 5 | Phase 6 (Performance) + Phase 7 (Physics) + bắt đầu portfolio project |
| 6 | Build và polish portfolio project — Mini Warehouse Digital Twin |

---

## Course tốt nhất: Three.js Journey

- https://threejs-journey.com (Bruno Simon)
- ~$95, có lifetime access
- Covers phases 1–4 đầy đủ với 70+ bài lab
- Worth every dollar nếu serious về 3D Web

---

## Quick Start — Chạy Demo Ngay

Không cần install gì, mở trực tiếp trong browser:

1. [hello-world.html](02-threejs-core/hello-world.html) — basic scene
2. [raycaster-click.html](02-threejs-core/raycaster-click.html) — click detection
3. [particles.html](02-threejs-core/particles.html) — particle system
4. [instancing-demo.html](02-threejs-core/instancing-demo.html) — 1000 robots @ 60fps
5. [shader-demo.html](03-shaders-glsl/shader-demo.html) — custom GLSL shaders

---

## Portfolio Project (để apply role này)

> **Mini Warehouse Digital Twin**
> - Three.js + React Three Fiber + Zustand
> - 100x100m warehouse floor (merged geometry)
> - 50+ animated robots (InstancedMesh — 1 draw call)
> - Click robot → telemetry sidebar (speed, battery, task, path)
> - WebSocket mock feed hoặc interval simulation
> - Floor heatmap (custom GLSL shader showing traffic density)
> - Orbit controls + top-down camera toggle
> - Sensor zones that trigger alerts
> - Target: 60fps stable với 50+ robots
> - Deploy: Vercel/Netlify

---

## Tools

| Tool | Mục đích |
|---|---|
| https://threejs.org/editor/ | Test scene không cần code |
| https://shadertoy.com | Xem/viết GLSL shaders |
| https://thebookofshaders.com | Học GLSL từng bước |
| Blender (free) | Tạo/export 3D models (.glb) |
| Spector.js (Chrome ext) | Debug WebGL draw calls |
| Chrome DevTools → Performance | Profile render loop |
| Stats.js | FPS overlay |
| Leva | Debug GUI controls |
