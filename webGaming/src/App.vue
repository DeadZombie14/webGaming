<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <RouterView />
</template>

<style>
* {
  box-sizing: border-box;
}

html, body, #app {
  margin: 0;
  height: 100vh;
  overflow-x: hidden;
  font-size: 18px;
  background: black;
}

:root {
  --mainColorH: 251;
  --mainColorS: 64%;
  --mainColor: hsl(var(--mainColorH), var(--mainColorS), 38%);
}

.btn {
  border: none;
  background: var(--mainColor);
  border-radius: 0.2rem;
  padding: .5rem;
  color: #FFF;
  cursor: pointer;
}
.btn:hover {
  background: hsl(var(--mainColorH), var(--mainColorS), 48%);
}

</style>