<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
	<myCanvas :row="32" :col="32" :pen="color"></myCanvas>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import myCanvas from './components/myCanvas.vue'

export default {
	name: 'app',
	data(){
		return{color:'#000000'}
	},
	components: {
		myCanvas
	},
	methods:{
		change(res){
			this.color=res
			// this.$emit('changeColor',res)
		}
	}
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  /* margin-top: 60px; */
}
</style>
