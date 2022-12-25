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
  width: 30px !important;
  height: 30px !important;
  padding: 5px !important;
  opacity: 1;
  background-color: red;
  border-radius: 30%;
}
.custom-div-icon:hover {
  filter: contrast(2);
}
.interactive-polyline:hover {
  stroke-opacity: 0.5 !important;
}
</style>
