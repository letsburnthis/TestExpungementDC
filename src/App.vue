<!--Mainn app, most of the actual code is in components folder-->

<template>
  <div id="app">
    
    <!--this places the component in the HTML-->
    <HelloWorld msg="Test"/>
    <crime msg=""/>
    
    
    
    
  </div>

</template>

<script>
//this is where compenents are made accessable to App.vue
import HelloWorld from './components/HelloWorld.vue';
import crime from './components/crime.vue'



export default {
  name: 'app',
  //this is where the components are listedd to make them accessabe to index.html???
  components: {
    HelloWorld,
    crime,
    
   
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
