<script setup>
</script>

<template>
  <div id="app">
    <header>
      <div class="wrapper">
        <nav>
          <router-link to="/">Home</router-link>
          <router-link to="/about">About</router-link>
        </nav>
      </div>
        <div class="image-editor-wrap">
          <router-view />
        </div>
    </header>
  </div>
</template>

<style scoped>
#app {
  width: 100%;
  height: 100%;
}
/* header {
  height: 100vh;
  width: 100vh;
} */

.wrapper {
  height: 60px;
}

.image-editor-wrap {
  height: 850px;
}
</style>
