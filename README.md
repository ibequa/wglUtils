# wglUtils
WebGL helper library for boilerplate code.

### Initializing
Request WebGL context with depth and stencil buffers.

```javascript
var utils = wglUtils.glUtils(canvas),
    gl = utils.glContext;   // get raw [WebGLRenderingContext]
```

### Shaders
* *Loading from DOM*
```javascript 
var vs = utils.shader().fromDOM('shader-vs');
```
* *Loading from source string*
```javascript
var fs = utils.shader().fromSource(str, gl.FRAGMENT_SHADER);
```

### WebGLProgram
* *Creating a program*
```javascript
var mainProgram = utils.program([vs, fs]);
```
* *Setting uniforms*
```javascript
mainProgram.use();
mainProgram.uniforms
    .set('u_model', model)                // model is {Number[]}
     .set('u_projection', projection)     // projection is {Number[]}
     .set('u_view', view)                 // view is {Number[]}
      .set('u_sampler', 0);
      .set('u_sampler1', 1);
```

### Buffers
* *Creating a buffer*
```javascript
var vbo = utils.buffer(gl.ARRAY_BUFFER, vertices, Float32Array, gl.DYNAMIC_DRAW);
```
* *Assigning vertex attributes*
```javascript
vbo.attribPointer(mainProgram, 'aVertexPosition', 5 , 0)
    .attribPointer(mainProgram, 'aUV', 5, 3);
```

### Textures
* **Creating a texture**
```javascript
var tex = utils.texture(gl.TEXTURE_2D, false, gl.NEAREST, gl.NEAREST, gl.REPEAT);
```
* *Filling a texture #1*
```javascript
tex.imageFromRaw(gl.TEXTURE_2D, pixelsImageCanvasOrVideo, gl.RGB, gl.UNSIGNED_BYTE);
```
* *Filling a texture #2*
```javascript
tex.imageFromSrc(gl.TEXTURE_2D, 'mytexture.png', gl.RGBA, gl.UNSIGNED_BYTE, textureDownloadFinishedCallback);
```
* *Multitextuing*
```javascript
// render loop
tex.bind(mainProgram, 'u_sampler');
tex1.bind(mainProgram, 'u_sampler1');
```
