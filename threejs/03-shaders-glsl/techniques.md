# GLSL Shader Techniques

## 1. Noise Functions

Noise là foundation của nhiều shader effects (waves, fire, clouds, terrain).

### Classic Perlin Noise (copy-paste vào shader)
```glsl
// 2D Perlin noise — glsl-noise library style
vec4 permute(vec4 x) { return mod(((x*34.0)+1.0)*x, 289.0); }
vec2 fade(vec2 t) { return t*t*t*(t*(t*6.0-15.0)+10.0); }

float cnoise(vec2 P) {
  vec4 Pi = floor(P.xyxy) + vec4(0.0, 0.0, 1.0, 1.0);
  vec4 Pf = fract(P.xyxy) - vec4(0.0, 0.0, 1.0, 1.0);
  Pi = mod(Pi, 289.0);
  vec4 ix = Pi.xzxz;
  vec4 iy = Pi.yyww;
  vec4 fx = Pf.xzxz;
  vec4 fy = Pf.yyww;
  vec4 i = permute(permute(ix) + iy);
  vec4 gx = 2.0 * fract(i * 0.0243902439) - 1.0;
  vec4 gy = abs(gx) - 0.5;
  vec4 tx = floor(gx + 0.5);
  gx = gx - tx;
  vec2 g00 = vec2(gx.x, gy.x);
  vec2 g10 = vec2(gx.y, gy.y);
  vec2 g01 = vec2(gx.z, gy.z);
  vec2 g11 = vec2(gx.w, gy.w);
  vec4 norm = 1.79284291400159 - 0.85373472095314 * vec4(dot(g00,g00), dot(g01,g01), dot(g10,g10), dot(g11,g11));
  g00 *= norm.x; g01 *= norm.y; g10 *= norm.z; g11 *= norm.w;
  float n00 = dot(g00, vec2(fx.x, fy.x));
  float n10 = dot(g10, vec2(fx.y, fy.y));
  float n01 = dot(g01, vec2(fx.z, fy.z));
  float n11 = dot(g11, vec2(fx.w, fy.w));
  vec2 fade_xy = fade(Pf.xy);
  vec2 n_x = mix(vec2(n00, n01), vec2(n10, n11), fade_xy.x);
  return 2.3 * mix(n_x.x, n_x.y, fade_xy.y);
}

// Usage in fragment shader:
float noise = cnoise(vUv * 5.0 + uTime * 0.2); // animated noise
```

### FBM (Fractal Brownian Motion) — layered noise for organic look
```glsl
float fbm(vec2 p) {
  float value = 0.0;
  float amplitude = 0.5;
  float frequency = 1.0;

  for (int i = 0; i < 6; i++) {
    value += amplitude * cnoise(p * frequency);
    amplitude *= 0.5;    // each octave is half amplitude
    frequency *= 2.0;    // each octave is double frequency
    p = mat2(1.6, 1.2, -1.2, 1.6) * p;  // rotate domain
  }
  return value;
}
```

---

## 2. Common Visual Effects

### Fresnel Effect (rim lighting / glow on edges)
```glsl
// Vertex shader
varying vec3 vNormal;
varying vec3 vViewDirection;

void main() {
  vNormal = normalize(normalMatrix * normal);
  vec4 worldPos = modelMatrix * vec4(position, 1.0);
  vViewDirection = normalize(cameraPosition - worldPos.xyz);
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}

// Fragment shader
varying vec3 vNormal;
varying vec3 vViewDirection;
uniform vec3 uFresnelColor;

void main() {
  float fresnel = 1.0 - max(0.0, dot(vNormal, vViewDirection));
  fresnel = pow(fresnel, 3.0);  // sharpen edge
  gl_FragColor = vec4(uFresnelColor * fresnel, fresnel);
}
```

### Dissolve Effect (using noise + threshold)
```glsl
uniform float uProgress;  // 0 = fully visible, 1 = fully dissolved
uniform sampler2D uNoiseMap;

varying vec2 vUv;

void main() {
  float noise = texture2D(uNoiseMap, vUv).r;

  // Discard pixel if noise < progress
  if (noise < uProgress) discard;

  // Edge glow
  float edgeWidth = 0.05;
  float edge = smoothstep(uProgress, uProgress + edgeWidth, noise);
  vec3 edgeColor = mix(vec3(1.0, 0.3, 0.0), vec3(1.0, 1.0, 1.0), edge);

  vec3 baseColor = vec3(0.4, 0.6, 1.0);
  gl_FragColor = vec4(mix(edgeColor, baseColor, edge), 1.0);
}
```

### Outline Effect (post-process style using normals)
```glsl
// Detect edges by sampling neighboring pixels
uniform sampler2D tDiffuse;
uniform vec2 uResolution;

varying vec2 vUv;

void main() {
  vec2 texel = 1.0 / uResolution;

  // Sobel operator for edge detection
  float n = texture2D(tDiffuse, vUv + vec2(0, texel.y)).r;
  float s = texture2D(tDiffuse, vUv - vec2(0, texel.y)).r;
  float e = texture2D(tDiffuse, vUv + vec2(texel.x, 0)).r;
  float w = texture2D(tDiffuse, vUv - vec2(texel.x, 0)).r;

  float edge = length(vec2(n - s, e - w));
  vec3 color = texture2D(tDiffuse, vUv).rgb;
  gl_FragColor = vec4(color * (1.0 - edge * 5.0), 1.0);
}
```

### Hologram Effect
```glsl
uniform float uTime;
varying vec3 vPosition;
varying vec2 vUv;
varying vec3 vNormal;
varying vec3 vViewDir;

void main() {
  // Scanlines
  float scanline = step(0.5, fract(vPosition.y * 20.0 - uTime * 2.0));

  // Fresnel for edge glow
  float fresnel = 1.0 - max(dot(vNormal, vViewDir), 0.0);
  fresnel = pow(fresnel, 2.0);

  // Flicker
  float flicker = 0.8 + 0.2 * sin(uTime * 40.0);

  vec3 holoColor = vec3(0.1, 0.9, 1.0);  // cyan
  float alpha = (scanline * 0.3 + fresnel * 0.7) * flicker;

  gl_FragColor = vec4(holoColor, alpha);
}
```

### Glow/Bloom via additive blending
```javascript
// No shader needed — use material blending
const glowMaterial = new THREE.MeshBasicMaterial({
  color: 0x00ffff,
  transparent: true,
  blending: THREE.AdditiveBlending,
  depthWrite: false,
})
// Or use UnrealBloomPass from EffectComposer (Phase 2 README)
```

---

## 3. Vertex Displacement Patterns

### Terrain / Heightmap
```glsl
uniform sampler2D uHeightMap;
uniform float uHeightScale;

varying float vHeight;

void main() {
  vec2 uv = uv;  // built-in
  float height = texture2D(uHeightMap, uv).r;
  vHeight = height;

  vec3 pos = position;
  pos.y += height * uHeightScale;

  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
}
```

### GPU Particles (vertex shader moves particles)
```glsl
uniform float uTime;
attribute float aRandom;  // per-particle random value

void main() {
  vec3 pos = position;

  // Circular orbit at random speeds
  float angle = uTime * (0.5 + aRandom * 1.5);
  float radius = 2.0 + aRandom * 3.0;
  pos.x = cos(angle) * radius;
  pos.z = sin(angle) * radius;
  pos.y = sin(uTime * 2.0 + aRandom * 10.0) * 0.5;

  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
  gl_PointSize = 3.0;
}
```

---

## 4. Texture Manipulation

### UV Scrolling (conveyor belt, waterfall)
```glsl
uniform float uTime;
varying vec2 vUv;

void main() {
  vec2 scrolledUV = vUv + vec2(0.0, uTime * 0.3);  // scroll upward
  vec4 color = texture2D(uTexture, scrolledUV);
  gl_FragColor = color;
}
```

### Texture Blending (terrain mix)
```glsl
uniform sampler2D uTextureGrass;
uniform sampler2D uTextureDirt;
uniform sampler2D uBlendMap;

varying vec2 vUv;

void main() {
  float blend = texture2D(uBlendMap, vUv).r;
  vec4 grass = texture2D(uTextureGrass, vUv * 10.0);
  vec4 dirt = texture2D(uTextureDirt, vUv * 10.0);
  gl_FragColor = mix(dirt, grass, blend);
}
```

### Normal Mapping from scratch
```glsl
// Fragment shader
uniform sampler2D uNormalMap;
uniform vec3 uLightDir;

varying vec2 vUv;
varying mat3 vTBN;  // tangent-bitangent-normal matrix (from vertex shader)

void main() {
  // Sample normal map (0–1) → convert to -1 to 1
  vec3 normalSample = texture2D(uNormalMap, vUv).rgb * 2.0 - 1.0;

  // Transform to world space using TBN matrix
  vec3 normal = normalize(vTBN * normalSample);

  // Diffuse lighting
  float diffuse = max(dot(normal, uLightDir), 0.0);

  gl_FragColor = vec4(vec3(diffuse), 1.0);
}
```

---

## 5. GPGPU (GPU General Purpose Computing)

Dùng GPU để simulate physics, particle systems với 1M+ particles.

```javascript
import { GPUComputationRenderer } from 'three/addons/misc/GPUComputationRenderer.js'

// Create computation renderer (width x height = number of particles)
const WIDTH = 64  // 64x64 = 4096 particles
const gpuCompute = new GPUComputationRenderer(WIDTH, WIDTH, renderer)

// Create position texture (stores x,y,z,w for each particle)
const dtPosition = gpuCompute.createTexture()
// Fill with initial positions
const posArray = dtPosition.image.data
for (let i = 0; i < posArray.length; i += 4) {
  posArray[i]     = (Math.random() - 0.5) * 10  // x
  posArray[i + 1] = (Math.random() - 0.5) * 10  // y
  posArray[i + 2] = (Math.random() - 0.5) * 10  // z
  posArray[i + 3] = 1.0                          // w
}

// Position update shader (runs every frame on GPU)
const positionVariable = gpuCompute.addVariable('texturePosition', `
  uniform float uTime;

  void main() {
    vec2 uv = gl_FragCoord.xy / resolution.xy;
    vec4 pos = texture2D(texturePosition, uv);

    // Simple gravity
    pos.y -= 0.001;

    // Wrap at floor
    if (pos.y < -5.0) pos.y = 5.0;

    gl_FragColor = pos;
  }
`, dtPosition)

positionVariable.material.uniforms.uTime = { value: 0 }
gpuCompute.setVariableDependencies(positionVariable, [positionVariable])
gpuCompute.init()

// In animate loop:
function animate() {
  positionVariable.material.uniforms.uTime.value = clock.getElapsedTime()
  gpuCompute.compute()
  // Use result texture in particle material:
  particleMaterial.uniforms.uParticlePositions.value = gpuCompute.getCurrentRenderTarget(positionVariable).texture
}
```

---

## 6. Useful Shader Recipes

### Polar coordinates
```glsl
// Convert UV to polar (angle + radius from center)
vec2 center = vec2(0.5, 0.5);
vec2 dir = vUv - center;
float angle = atan(dir.y, dir.x);  // -PI to PI
float radius = length(dir);        // 0 to ~0.7

// Star pattern
float starMask = step(0.5 + 0.3 * cos(angle * 5.0), radius * 2.0);
```

### SDF (Signed Distance Field) shapes
```glsl
// SDF circle
float sdCircle(vec2 p, float r) {
  return length(p) - r;
}

// SDF box
float sdBox(vec2 p, vec2 b) {
  vec2 d = abs(p) - b;
  return length(max(d, 0.0)) + min(max(d.x, d.y), 0.0);
}

// Usage: create crisp shapes without texture
float dist = sdCircle(vUv - 0.5, 0.3);
float mask = 1.0 - smoothstep(-0.01, 0.01, dist);
gl_FragColor = vec4(vec3(1.0), mask);
```

### Cel shading (toon shading)
```glsl
uniform vec3 uLightDir;
varying vec3 vNormal;

void main() {
  float diffuse = max(dot(vNormal, uLightDir), 0.0);

  // Quantize to discrete steps
  float toon = floor(diffuse * 4.0) / 4.0;

  gl_FragColor = vec4(vec3(0.3, 0.7, 0.9) * toon, 1.0);
}
```

---

## 7. Debugging Shaders

```glsl
// Visualize UV
gl_FragColor = vec4(vUv, 0.0, 1.0);  // red=U, green=V

// Visualize normals (world space)
gl_FragColor = vec4(vNormal * 0.5 + 0.5, 1.0);  // map -1..1 → 0..1

// Visualize a float value
float value = someCalculation();
gl_FragColor = vec4(value, value, value, 1.0);  // grayscale

// Checkerboard pattern (verify UV tiling)
float checker = mod(floor(vUv.x * 10.0) + floor(vUv.y * 10.0), 2.0);
gl_FragColor = vec4(vec3(checker), 1.0);
```
