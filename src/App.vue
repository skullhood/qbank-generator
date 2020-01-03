<template>
  <div id="platform">
    <div id="app">
      <QView/>
    </div>
  </div>
</template>

<script>
import QView from './components/QView.vue'

export default {
  components: {
    QView
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
  padding-left: 1%;
  padding-right: 1%;
  padding-top: 5%;
  padding-bottom: 5%;
}

#platform { 
  border-radius: 10px; 
  box-shadow: 10px 10px 15px #AAAAAA;
}

button {
  margin: 5px;
  background-color: #AAAAAA; /* Green */
  border: none;
  color: white;
  padding: 7px 14px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  border-radius: 10px;
  -webkit-transition-duration: 0.4s; /* Safari */
  transition-duration: 0.4s;
}

button:hover{
  background-color: #4CAF50; /* Green */
  color: white;
  cursor: pointer;
}

</style>
