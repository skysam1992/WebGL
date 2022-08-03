<template>
  <canvas ref="canvas"></canvas>
</template>

<script>
import { mat4 } from 'gl-matrix'

export default {
  mounted() {
    this.main()
  },
  methods: {
    main() {
      const gl = this.$refs.canvas.getContext('webgl')
      if (!gl) {
        console.log('null')
        return
      }

      const vsSource = `
        attribute vec4 aVertexPosition;
        attribute vec4 aVertexColor;

        uniform mat4 uModelViewMatrix;
        uniform mat4 uProjectionMatrix;

        varying lowp vec4 vColor;

        void main() {
          gl_Position = uProjectionMatrix * uModelViewMatrix * aVertexPosition;
          vColor = aVertexColor;
        }
      `
      const fsSource = `
        varying lowp vec4 vColor;
        void main() {
          gl_FragColor = vColor;
        }
      `
      const shaderProgram = this.initShaderProgram(gl, vsSource, fsSource)

      const programInfo = {
        program: shaderProgram,
        attribLocations: {
          vertexPosition: gl.getAttribLocation(shaderProgram, 'aVertexPosition'),
          vertexColorAttribute: gl.getAttribLocation(shaderProgram, "aVertexColor")
        },
        uniformLocations: {
          projectionMatrix: gl.getUniformLocation(shaderProgram, 'uProjectionMatrix'),
          modelViewMatrix: gl.getUniformLocation(shaderProgram, 'uModelViewMatrix'),
        }
      }

      const buffers = this.initBuffers(gl)

      let squareRotation = 0.0
      // this.drawScene(gl, programInfo, buffers, squareRotation)
      this.startAnimation(gl, programInfo, buffers, squareRotation)

    },

    startAnimation(gl, programInfo, buffers, squareRotation) {
      setInterval(() => {
        this.drawScene(gl, programInfo, buffers, squareRotation)
        squareRotation += 0.01
      }, 10);
    },

    initShaderProgram(gl, vsSource, fsSource) {
      const vertexShader = this.loadShader(gl, gl.VERTEX_SHADER, vsSource)
      const fragmentShader = this.loadShader(gl, gl.FRAGMENT_SHADER, fsSource)
      
      const shaderProgram = gl.createProgram()
      gl.attachShader(shaderProgram, vertexShader)
      gl.attachShader(shaderProgram, fragmentShader)
      gl.linkProgram(shaderProgram)

      if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
        console.log('init error' + gl.getProgramInfoLog(shaderProgram))
        return null;
      }

      return shaderProgram
    },
    loadShader(gl, type, source) {
      const shader = gl.createShader(type)
      gl.shaderSource(shader, source)
      gl.compileShader(shader)
      if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        console.log('load error' + gl.getShaderInfoLog(shader))
        gl.deleteShader(shader)
        return null
      }

      return shader
    },
    initBuffers(gl) {
      const positions = [
        // Front face
        -1.0, -1.0,  1.0,
        1.0, -1.0,  1.0,
        1.0,  1.0,  1.0,
        -1.0,  1.0,  1.0,

        // Back face
        -1.0, -1.0, -1.0,
        -1.0,  1.0, -1.0,
        1.0,  1.0, -1.0,
        1.0, -1.0, -1.0,

        // Top face
        -1.0,  1.0, -1.0,
        -1.0,  1.0,  1.0,
        1.0,  1.0,  1.0,
        1.0,  1.0, -1.0,

        // Bottom face
        -1.0, -1.0, -1.0,
        1.0, -1.0, -1.0,
        1.0, -1.0,  1.0,
        -1.0, -1.0,  1.0,

        // Right face
        1.0, -1.0, -1.0,
        1.0,  1.0, -1.0,
        1.0,  1.0,  1.0,
        1.0, -1.0,  1.0,

        // Left face
        -1.0, -1.0, -1.0,
        -1.0, -1.0,  1.0,
        -1.0,  1.0,  1.0,
        -1.0,  1.0, -1.0
      ]
      const positionBuffer = gl.createBuffer()
      gl.bindBuffer(WebGLRenderingContext.ARRAY_BUFFER, positionBuffer)
      gl.bufferData(WebGLRenderingContext.ARRAY_BUFFER, new Float32Array(positions), WebGLRenderingContext.STATIC_DRAW)

      const faceColors = [
        [1.0,  1.0,  1.0,  1.0],    // 前白
        [1.0,  0.0,  0.0,  1.0],    // 后红
        [0.0,  1.0,  0.0,  1.0],    // 上绿
        [0.0,  0.0,  1.0,  1.0],    // 下蓝
        [1.0,  1.0,  0.0,  1.0],    // 右黄
        [1.0,  0.0,  1.0,  1.0]     // 左紫
      ]
      let colors = []
      for (var j = 0; j < faceColors.length; ++j) {
        const c = faceColors[j];
        colors = colors.concat(c, c, c, c);
      }
      const colorBuffer = gl.createBuffer()
      gl.bindBuffer(WebGLRenderingContext.ARRAY_BUFFER, colorBuffer)
      gl.bufferData(WebGLRenderingContext.ARRAY_BUFFER, new Float32Array(colors), WebGLRenderingContext.STATIC_DRAW)

      const cubeVertexIndices = [
        0,  1,  2,      0,  2,  3,    // 前
        4,  5,  6,      4,  6,  7,    // 后
        8,  9,  10,     8,  10, 11,   // 上
        12, 13, 14,     12, 14, 15,   // 下
        16, 17, 18,     16, 18, 19,   // 右
        20, 21, 22,     20, 22, 23    // 左
      ]
      const cubeVerticesIndexBuffer = gl.createBuffer()
      gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVerticesIndexBuffer)
      gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(cubeVertexIndices), gl.STATIC_DRAW)

      return {
        position: positionBuffer,
        color: colorBuffer,
        cube: cubeVerticesIndexBuffer
      }
    },
    drawScene(gl, programInfo, buffers, squareRotation) {
      gl.clearColor(0.0, 0.0, 0.0, 1.0)
      gl.clearDepth(1.0)
      gl.enable(gl.DEPTH_TEST)
      gl.depthFunc(gl.LEQUAL)

      gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT)

      const fieldOfView = 45 * Math.PI / 180
      const aspect = gl.canvas.clientWidth / gl.canvas.clientHeight
      const zNear = 0.1
      const zFar = 100.0
      const projectionMatrix = mat4.create()
      mat4.perspective(projectionMatrix, fieldOfView, aspect, zNear, zFar)

      const modelViewMatrix = mat4.create()
      mat4.translate(modelViewMatrix, modelViewMatrix, [-0.0, 0.0, -6.0])

      mat4.rotate(modelViewMatrix, modelViewMatrix, squareRotation, [0, 0, 1])
      mat4.rotate(modelViewMatrix, modelViewMatrix, squareRotation * 0.7, [0, 1, 0])
      mat4.rotate(modelViewMatrix, modelViewMatrix, squareRotation * 0.3, [1, 0, 0])

      {
        const numComponents = 3
        const type = gl.FLOAT
        const normalize = false
        const stride = 0
        const offset = 0
        gl.bindBuffer(gl.ARRAY_BUFFER, buffers.position)
        gl.vertexAttribPointer(programInfo.attribLocations.vertexPosition, numComponents, type, normalize, stride, offset)
        gl.enableVertexAttribArray(programInfo.attribLocations.vertexPosition)
      }

      {
        const numComponents = 4;
        const type = gl.FLOAT;
        const normalize = false;
        const stride = 0;
        const offset = 0;
        gl.bindBuffer(gl.ARRAY_BUFFER, buffers.color);
        gl.vertexAttribPointer(programInfo.attribLocations.vertexColorAttribute, numComponents, type, normalize, stride, offset);
        gl.enableVertexAttribArray(programInfo.attribLocations.vertexColorAttribute);
      }

      {
        gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, buffers.cube);
      }
      
      gl.useProgram(programInfo.program)

      gl.uniformMatrix4fv(programInfo.uniformLocations.projectionMatrix, false, projectionMatrix)
      gl.uniformMatrix4fv(programInfo.uniformLocations.modelViewMatrix, false, modelViewMatrix)

      {
        const vertexCount = 36;
        const type = gl.UNSIGNED_SHORT;
        const offset = 0;
        gl.drawElements(gl.TRIANGLES, vertexCount, type, offset);
      }
    }
  }
}
</script>