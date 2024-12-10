<template>
    <div class="home">
      <h1>Welcome to the Item Manager PWA</h1>
      <ItemManager />
    </div>
  </template>
  
  <script>
  import ItemManager from "../components/ItemManager.vue";
  
  export default {
    components: {
      ItemManager,
    },
  };
  </script>
  
  <style scoped>
  .home {
    text-align: center;
    padding: 20px;
  }
  </style>
  