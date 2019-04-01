<template>
  <div id="app" class="has-background-light">
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: 'jacketer'
  }
</script>

<style lang="scss">
  @import "node_modules/bulma/bulma.sass";
  html {
    height: 100%;
  }
  body {
    height: 100%;
  }
  div#app {
    height: 100%;
  }
</style>
