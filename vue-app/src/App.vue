<script setup lang="ts">
import ProductList from './components/ProductList.vue'
</script>

<template>
  <ProductList />
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body, html {
  margin: 0;
  padding: 0;
  overflow: hidden;
  height: 100vh;
  width: 100vw;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

#app {
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}
</style>

