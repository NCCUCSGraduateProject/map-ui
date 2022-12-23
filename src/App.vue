<script setup></script>

<template>
  <router-view />
</template>

<style>
@import "@/assets/base.css";

body {
}

#app {
  width: 100%;
  margin: 0 auto;

  font-weight: normal;
}
.custom-div-icon {
  /* background-color: white; */
  /* border-radius: 50%; */
  opacity: 0.8;
  background-color: white;
  border-radius: 30%;
}
.custom-div-icon:hover {
  opacity: 1;
}
.interactive-polyline:hover {
  stroke-opacity: 0.5;
}
</style>
