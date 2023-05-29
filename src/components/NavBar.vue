<script setup>
import { ref } from 'vue';

const msg = ref('哈哈哈')
</script>

<template>
  <div class="navbar">
    <h1>NavBar: {{ msg }}</h1>
  </div>
</template>

<style scoped>
.navbar {
  color: red;
}
</style>