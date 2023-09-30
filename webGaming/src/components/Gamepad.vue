<template>
    <div>       
    </div>
</template>

<style scoped>
/**/
</style>

<script lang="ts">
import { ref } from 'vue'
window.addEventListener("gamepadconnected", (e) => {
  console.log(
    "Gamepad connected at index %d: %s. %d buttons, %d axes.",
    e.gamepad.index,
    e.gamepad.id,
    e.gamepad.buttons.length,
    e.gamepad.axes.length,
  );
})
</script>