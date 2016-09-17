/**
 * Created by ilya on 11/08/16.
 */

var wglUtils = wglUtils || {};

wglUtils.glUtils = function (canvas) {
    var gl = canvas.getContext('webgl', {stencil: true}) || canvas.getContext('experimental-webgl');

    if (!gl) {
        alert('error initialising webgl');
        return null;
    }

    gl.viewport(0, 0, canvas.width, canvas.height);

    function wglTextureManager(gl, target, mipmap, min_filter, mag_filter, wrap) {
        var texture = gl.createTexture();

        gl.bindTexture(target, texture);

        if (mipmap === 'undefined') mipmap = true;
        min_filter = min_filter || gl.LINEAR;
        mag_filter = mag_filter || gl.LINEAR;
        wrap = wrap || gl.REPEAT;

        gl.texParameteri(target, gl.TEXTURE_MIN_FILTER, min_filter);
        gl.texParameteri(target, gl.TEXTURE_MAG_FILTER, mag_filter);
        gl.texParameteri(target, gl.TEXTURE_WRAP_S, wrap);
        gl.texParameteri(target, gl.TEXTURE_WRAP_T, wrap);

        return {
            glTexture: texture,
            /**
             * @param imageTarget
             * @param src
             * @param internalFormat
             * @param type
             * @param callback
             */
            imageFromSrc: function (imageTarget, src, internalFormat, type, callback) {
                var img = new Image();

                img.onload = function () {
                    gl.bindTexture(target, texture);
                    gl.texImage2D(imageTarget, 0, internalFormat, internalFormat, type, img);

                    if (mipmap) {
                        gl.generateMipmap(target);
                        gl.texParameteri(target, gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_LINEAR);
                    }
                    callback();
                };
                img.src = src;
                return this;
            },
            /**
             * @param target
             * @param {ImageData|HTMLImageElement|HTMLCanvasElement|HTMLVideoElement} pixelsImageCanvasOrVideo
             * @param internalFormat
             * @param type
             */
            imageFromRaw: function (target, pixelsImageCanvasOrVideo, internalFormat, type) {
                gl.bindTexture(target, texture);
                gl.texImage2D(target, 0, internalFormat, internalFormat, type, pixelsImageCanvasOrVideo);

                if (mipmap) {
                    gl.generateMipmap(target);
                    gl.texParameteri(target, gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_LINEAR);
                }
                return this;
            },
            /**
             * @param program
             * @param uniformName
             */
            bind: function (program, uniformName) {
                var u = program.uniforms.get(uniformName);

                var bindPoint = u.cachedSamplerValue,
                    textureUnit = gl.TEXTURE0 + bindPoint;

                gl.activeTexture(textureUnit);
                gl.bindTexture(target, texture);
                return this;
            }
        }
    }

    function wglShaderManager(gl) {
        var shader;

        return {
            glShader: shader,

            /**
             * @param source
             * @param type
             */
            fromSource: function (source, type) {
                shader = gl.createShader(type);
                gl.shaderSource(shader, source);
                gl.compileShader(shader);

                if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
                    console.log('error compiling the shader:' + gl.getShaderInfoLog(shader));
                    gl.deleteShader(shader);
                    return null;
                }
                return shader;
            },
            /**
             * @param domID
             * @param type
             */
            fromDOM: function (domID, type) {
                var theSource, shaderDom;

                shaderDom = document.getElementById(domID);
                if (!shaderDom) {
                    console.log('cannot find document element with id ' + domID);
                    return null;
                }
                // Get shader source
                theSource = shaderDom.textContent;

                // Get shader type
                if (!type) {
                    if (shaderDom.type === 'x-shader/x-vertex') {
                        type = gl.VERTEX_SHADER;
                    }
                    else if (shaderDom.type === 'x-shader/x-fragment') {
                        type = gl.FRAGMENT_SHADER;
                    }
                    else {
                        console.log('invalid shader type ' + shaderDom.type);
                        return null;
                    }
                }
                return this.fromSource(theSource, type);
            }
        }
    }

    function wglProgramManager(gl, shaders) {
        var program = gl.createProgram();

        var uniforms = {},
            attributes = {};

        for (var i = 0; i < shaders.length; i++)
            gl.attachShader(program, shaders[i]);

        gl.linkProgram(program);
        if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
            console.log('failed at program linking: ' + gl.getProgramInfoLog(program));
            gl.deleteProgram(program);
            return null;
        }

        // Collect active uniforms
        var count = gl.getProgramParameter(program, gl.ACTIVE_UNIFORMS),
            activeInfo = null,
            name = '',
            location;
        for (i = 0; i < count; i++) {
            activeInfo = gl.getActiveUniform(program, i);

            name = activeInfo.name;
            location = gl.getUniformLocation(program, name);

            uniforms[name] = {
                location: location,
                type: activeInfo.type
            };
        }

        // Collect active attributes
        var attribSize = 0,
            type;
        count = gl.getProgramParameter(program, gl.ACTIVE_ATTRIBUTES);
        for (i = 0; i < count; i++) {
            activeInfo = gl.getActiveAttrib(program, i);

            name = activeInfo.name;
            location = gl.getAttribLocation(program, name);
            type = activeInfo.type;
            attribSize = getAttribSize(gl, type);

            attributes[name] = {
                location: location,
                type: type,
                size: attribSize
            }
        }

        return {
            glProgram: program,
            uniforms: {
                /**
                 * @param uniformName
                 */
                get: function (uniformName) {
                    var u = uniforms[uniformName];
                    if (!u) console.log('uniform ' + uniformName + ' does not active or exist');
                    return u;
                },
                /**
                 * @param uniformName
                 * @param value
                 */
                set: function (uniformName, value) {
                    var u = uniforms[uniformName];
                    if (u) {
                        setUniform(gl, u, value);
                    } else {
                        console.log('uniform ' + uniformName + ' does not active or exist');
                    }
                    return this;
                }
            },
            attributes: {
                /**
                 * @param attribName
                 */
                get: function (attribName) {
                    var attr = attributes[attribName];
                    if (!attr) console.log('attribute ' + attribName + ' does not active or exist');
                    return attr;
                }
            },
            use: function () {
                gl.useProgram(program);
            }
        }
    }

    function wglBufferManager(gl, target, data, TypedArray, draw) {
        var buffer = gl.createBuffer(),
            componentByteSize = TypedArray.BYTES_PER_ELEMENT;
        gl.bindBuffer(target, buffer);
        gl.bufferData(target, new TypedArray(data), draw);

        return {
            glBuffer: buffer,
            bytesPerElement: componentByteSize,
            length: data.length,
            /**
             * @param offset
             * @param data
             */
            update: function (offset, data) {
                gl.bindBuffer(target, buffer);
                gl.bufferSubData(target, offset, new TypedArray(data));
                return this;
            },
            /**
             * @param program
             * @param name
             * @param stride
             * @param offset
             * @param type
             */
            attribPointer: function (program, name, stride, offset, type) {
                var attr = program.attributes.get(name);
                if (!attr) return this;

                stride = stride || attr.size;
                offset = offset || 0;

                stride *= this.bytesPerElement;
                offset *= this.bytesPerElement;

                type = type || gl.FLOAT;
                gl.bindBuffer(target, buffer);
                gl.enableVertexAttribArray(attr.location);
                gl.vertexAttribPointer(attr.location, attr.size, type, gl.FALSE, stride, offset);
                return this;
            },
            bind: function () {
                gl.bindBuffer(target, buffer);
                return this;
            }
        }
    }

    function setUniform(gl, u, value) {
        var type = u.type,
            location = u.location;
        switch (type) {
            case gl.FLOAT_MAT4:
                gl.uniformMatrix4fv(location, false, value);
                break;
            case gl.FLOAT_VEC3:
                gl.uniform3f(location, value[0], value[1], value[2]);
                break;
            case gl.FLOAT_VEC2:
                gl.uniform2f(location, value[0], value[1]);
                break;
            case gl.FLOAT_VEC4:
                gl.uniform4f(location, value[0], value[1], value[2], value[3]);
                break;
            case gl.FLOAT:
                gl.uniform1f(location, value);
                break;
            case gl.INT:
                gl.uniform1i(location, value);
                break;
            case gl.SAMPLER_2D:
            case gl.SAMPLER_CUBE:
                gl.uniform1i(location, value);
                // Cache sampler value for later texture binding
                u.cachedSamplerValue = value;
                break;
            case gl.INT_VEC3:
                gl.uniform3i(location, value[0], value[1], value[2]);
                break;
            case gl.INT_VEC4:
                gl.uniform3i(location, value[0], value[1], value[2]);
                break;
            case gl.FLOAT_MAT3:
                gl.uniformMatrix3fv(location, false, value);
                break;
            case gl.FLOAT_MAT2:
                gl.uniformMatrix2fv(location, false, value);
                break;
            case gl.INT_VEC2:
                gl.uniform2i(location, value[0], value[1]);
                break;
            default:
                console.log('unknown uniform type ' + type);
        }
    }

    function getAttribSize(gl, type) {
        if (type === gl.FLOAT || type === gl.INT) return 1;
        else if (type === gl.FLOAT_VEC2 || type === gl.INT_VEC2) return 2;
        else if (type === gl.FLOAT_VEC3 || type === gl.INT_VEC3) return 3;
        else return 4;
    }

    return {
        glContext: gl,

        /**
         * @param {Number} target
         * @param {Number[]} data
         * @param {ArrayBuffer} TypedArray
         * @param {Number} draw
         */
        buffer: function (target, data, TypedArray, draw) {
            return wglBufferManager(gl, target, data, TypedArray, draw);
        },

        shader: function () {
            return wglShaderManager(gl);
        },

        /**
         * @param {Object[]} shaders
         */
        program: function (shaders) {
            return wglProgramManager(gl, shaders);
        },

        /**
         * @param {Number} target
         * @param {Boolean=1} mipmap
         * @param {Number=} min_filter
         * @param {Number=} mag_filter
         * @param {Number=} wrap
         */
        texture: function (target, mipmap, min_filter, mag_filter, wrap) {
            return wglTextureManager(gl, target, mipmap, min_filter, mag_filter, wrap);
        }
    }
};
