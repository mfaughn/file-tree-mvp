<!-- App.vue -->
<template>
  <div id="app">
    <h1>File Tree Drag & Drop Demo</h1>
    <file-tree></file-tree>
  </div>
</template>

<script>
import FileTree from './components/FileTree.vue';

export default {
  name: 'App',
  components: {
    FileTree
  }
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  padding: 0 20px;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
}

/* Global reset */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

ul {
  list-style-type: none;
}
</style>
