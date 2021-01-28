<template>
  <div id="app">
    <keep-alive exclude="Detail">
      <router-view/>
    </keep-alive>
    <main-tar-bar/>
  </div>
</template>

<script>
  import MainTarBar from 'components/contents/maintarbar/MainTarBar'
  export default {
    name: 'app',
    components: {
      MainTarBar
    }
  }
</script>

<style>
  /*css 引入方式*/
  @import "assets/css/base.css";
</style>
