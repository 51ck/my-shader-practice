<template>
  <canvas ref="canvas"></canvas>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue';
export default defineComponent({
  name: 'MyCanvas',

  setup() {
    const canvas = ref<HTMLCanvasElement | null>();

    const setCanvasSize = (w=window.innerWidth, h=window.innerHeight) => {
      if (!canvas.value) {
        return;
      }
      canvas.value.width = w;
      canvas.value.height = h;
    }

    onMounted(async () => {
      if (!canvas.value) {
        return;
      }

      setCanvasSize();
      const gl = canvas.value.getContext('webgl2');
      let buffer: WebGLBuffer;

      if (!gl) {
        return;
      }

      gl.viewport(0, 0, gl.drawingBufferWidth, gl.drawingBufferHeight);
      gl.clearColor(0.0, 0.0, 0.0, 1.0);
      gl.clear(gl.COLOR_BUFFER_BIT);



      // vertex shader
      const vertexShaderSource = (await import('./vertex.glsl')).default;
      const vertexShader = gl.createShader(gl.VERTEX_SHADER) as WebGLShader;
      gl.shaderSource(vertexShader, vertexShaderSource);
      gl.compileShader(vertexShader);

      // fragment shader
      const fragmentShaderSource = (await import('./fragment.glsl')).default;
      const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER) as WebGLShader;
      gl.shaderSource(fragmentShader, fragmentShaderSource);
      gl.compileShader(fragmentShader);



      let program: WebGLProgram = gl.createProgram() as WebGLProgram;
      gl.attachShader(program, vertexShader);
      gl.attachShader(program, fragmentShader);
      gl.linkProgram(program);
      gl.detachShader(program, vertexShader);
      gl.detachShader(program, fragmentShader);
      gl.deleteShader(vertexShader);
      gl.deleteShader(fragmentShader);

      if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
        const linkErrLog = gl.getProgramInfoLog(program);
        cleanup();
        console.error(`Shader program did not link successfully. Error log: ${linkErrLog}`);
        return;
      }

      initializeAttributes();

      gl.useProgram(program);
      gl.drawArrays(gl.POINTS, 0, 1);

      // cleanup();


      function initializeAttributes() {
        if (!gl) {
          console.warn('[initializeAttributes] gl is null')
          return;
        }
        gl.enableVertexAttribArray(0);
        buffer = gl.createBuffer() as WebGLBuffer;
        gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
        gl.vertexAttribPointer(0, 1, gl.FLOAT, false, 0, 0);
      }

      function cleanup() {
        if (!gl) {
          console.warn('[cleanup] gl is null')
          return;
        }
        gl.useProgram(null);
        if (buffer) {
          gl.deleteBuffer(buffer);
        }
        if (program) {
          gl.deleteProgram(program);
        }
      }
    });

    return {
      canvas,
    }
  },
});
</script>
