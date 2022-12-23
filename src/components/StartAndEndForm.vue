<template>
  <div
    style="
      width: 300px;
      position: absolute;
      z-index: 800;
      top: 2vh;
      right: 2vw;

      background-color: white;
      padding: 20px;
      border-radius: 10px;
    "
  >
    <slot />
  </div>
</template>

<style scoped>
:slotted(div) {
  margin-bottom: 1vh;
}
</style>
