<template>
  <header class="fixed z-10 pointer-events-none">
    <h1 class="font-mono text-2xl">my shader excercise</h1>
  </header>
  <RouterView />

  <MyCanvas class="fixed w-full h-full inset-0"></MyCanvas>
</template>

<script lang="ts">
import { RouterView } from 'vue-router'
import { defineComponent } from 'vue';
import MyCanvas from './components/MyCanvas/MyCanvas.vue';

export default defineComponent({
  name: 'App',
  components: {
    RouterView,
    MyCanvas,
  },
});
</script>
