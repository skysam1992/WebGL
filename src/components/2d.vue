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
      const positionBuffer = gl.createBuffer()
      const positions = [
        1.0,  1.0,  0.0,
        -1.0, 1.0,  0.0,
        1.0,  -1.0, 0.0,
        -1.0, -1.0, 0.0
      ]
      gl.bindBuffer(WebGLRenderingContext.ARRAY_BUFFER, positionBuffer)
      gl.bufferData(WebGLRenderingContext.ARRAY_BUFFER, new Float32Array(positions), WebGLRenderingContext.STATIC_DRAW)

      const colorBuffer = gl.createBuffer()
      const colors = [
        1.0,  1.0,  1.0,  1.0,  // ???
        1.0,  0.0,  0.0,  1.0,  // ???
        0.0,  1.0,  0.0,  1.0,  // ???
        0.0,  0.0,  1.0,  1.0,  // ???
      ]
      gl.bindBuffer(WebGLRenderingContext.ARRAY_BUFFER, colorBuffer)
      gl.bufferData(WebGLRenderingContext.ARRAY_BUFFER, new Float32Array(colors), WebGLRenderingContext.STATIC_DRAW)

      return {
        position: positionBuffer,
        color: colorBuffer
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

      mat4.rotate(modelViewMatrix, modelViewMatrix, squareRotation, [0, 0, 1]);

      {
        const numComponents = 3
        const type = gl.FLOAT
        const normalize = false
        const stride = 0
        const offset = 0
        gl.bindBuffer(gl.ARRAY_BUFFER, buffers.position)
        gl.vertexAttribPointer(programInfo.attribLocations.vertexPosition, numComponents, type, normalize, stride, offset)
        gl.enableVertexAttribArray(programInfo.attribLocations.vertexPosition)

        gl.bindBuffer(gl.ARRAY_BUFFER, buffers.color)
        gl.vertexAttribPointer(programInfo.attribLocations.vertexColorAttribute, numComponents, type, normalize, stride, offset)
        gl.enableVertexAttribArray(programInfo.attribLocations.vertexColorAttribute)
      }
      
      gl.useProgram(programInfo.program)

      gl.uniformMatrix4fv(programInfo.uniformLocations.projectionMatrix, false, projectionMatrix)

      gl.uniformMatrix4fv(programInfo.uniformLocations.modelViewMatrix, false, modelViewMatrix)

      {
        const offset = 0;
        const vertexCount = 4;
        gl.drawArrays(gl.TRIANGLE_STRIP, offset, vertexCount);
      }
    }
  }
}
</script>