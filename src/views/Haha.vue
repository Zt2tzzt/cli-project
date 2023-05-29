<script setup>
import { ref } from 'vue';

const msg = ref('哈哈哈')
</script>

<template>
  <div class="haha">
    <h1>NavBar: {{ msg }}</h1>
  </div>
</template>

<style scoped>
.haha {
  color: red;
}
</style>