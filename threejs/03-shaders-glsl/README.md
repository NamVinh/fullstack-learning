# Phase 3 — GLSL Shaders

## Tại sao cần học Shaders?

- Performance: shader chạy trên GPU, xử lý millions of pixels song song
- Custom visual effects không thể làm với standard materials
- Differentiating skill — không phải frontend dev nào cũng biết

## WebGL Pipeline

```
CPU (JavaScript)
    ↓
Vertex Shader (GPU) — chạy 1 lần cho mỗi VERTEX
    ↓
Rasterization — GPU tự làm, convert triangles → fragments (pixels)
    ↓
Fragment Shader (GPU) — chạy 1 lần cho mỗi PIXEL
    ↓
Screen
```

## GLSL Data Types

```glsl
float x = 1.0;        // PHẢI có .0 — không phải integer
int i = 5;
bool flag = true;

vec2 uv = vec2(0.5, 0.5);     // 2D vector (x,y) hoặc (r,g)
vec3 pos = vec3(1.0, 2.0, 3.0); // 3D vector (x,y,z) hoặc (r,g,b)
vec4 color = vec4(1.0, 0.0, 0.0, 1.0); // (r,g,b,a)

mat4 transform; // 4x4 matrix

// Swizzling — truy cập components linh hoạt
vec3 v = vec3(1.0, 2.0, 3.0);
float x = v.x;     // 1.0
vec2 xy = v.xy;    // vec2(1.0, 2.0)
vec3 rgb = v.rgb;  // same as xyz
vec3 rev = v.zyx;  // vec3(3.0, 2.0, 1.0)
```

## Vertex Shader (Three.js built-ins)

```glsl
// Uniforms tự động inject bởi Three.js:
uniform mat4 modelMatrix;       // object world transform
uniform mat4 viewMatrix;        // camera transform
uniform mat4 projectionMatrix;  // perspective/ortho projection
uniform mat4 modelViewMatrix;   // modelMatrix * viewMatrix (shortcut)

// Attributes từ BufferGeometry:
attribute vec3 position;  // vertex position
attribute vec3 normal;    // vertex normal
attribute vec2 uv;        // texture coordinates

// Varying — pass data sang fragment shader:
varying vec2 vUv;
varying vec3 vNormal;

void main() {
  vUv = uv;
  vNormal = normal;

  // Transform vertex to clip space
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}
```

## Fragment Shader

```glsl
precision mediump float;  // precision hint

varying vec2 vUv;  // nhận từ vertex shader

void main() {
  // gl_FragColor = final pixel color (r, g, b, a)
  gl_FragColor = vec4(vUv.x, vUv.y, 0.0, 1.0); // color based on UV
}
```

## ShaderMaterial trong Three.js

```javascript
const material = new THREE.ShaderMaterial({
  uniforms: {
    uTime: { value: 0.0 },
    uColor: { value: new THREE.Color(0xff6b6b) },
  },
  vertexShader: `
    varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    uniform float uTime;
    uniform vec3 uColor;
    varying vec2 vUv;
    void main() {
      float pulse = sin(uTime * 3.0) * 0.5 + 0.5;
      gl_FragColor = vec4(uColor * pulse, 1.0);
    }
  `,
})

// Update uniform trong animation loop
function animate() {
  material.uniforms.uTime.value = clock.getElapsedTime()
}
```

## Useful GLSL Functions

```glsl
// Interpolate between two values
float result = mix(0.0, 1.0, 0.5); // 0.5

// Clamp to range
float clamped = clamp(value, 0.0, 1.0);

// Smooth step (S-curve interpolation)
float smooth = smoothstep(0.3, 0.7, uv.x);

// Trig
float wave = sin(uTime + position.x);

// Fractional part (fract) — tạo repeating pattern
float rep = fract(uv.x * 10.0); // 0→1 repeated 10 times

// Length — distance from origin
float dist = length(uv - vec2(0.5)); // distance from center
```

## Common Patterns

### Wave effect
```glsl
void main() {
  vec3 pos = position;
  pos.y += sin(pos.x * 5.0 + uTime * 2.0) * 0.2;
  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
}
```

### Gradient based on UV
```glsl
void main() {
  vec3 colorA = vec3(0.31, 0.80, 0.78); // teal
  vec3 colorB = vec3(1.0, 0.42, 0.42);  // red
  vec3 color = mix(colorA, colorB, vUv.x);
  gl_FragColor = vec4(color, 1.0);
}
```

### Circle mask
```glsl
void main() {
  vec2 center = vec2(0.5, 0.5);
  float dist = length(vUv - center);
  float circle = 1.0 - smoothstep(0.3, 0.32, dist);
  gl_FragColor = vec4(vec3(circle), 1.0);
}
```
