<template>
  <v-app>
    <v-main class="app">
      <div>
        <d3container/>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import d3container from './components/d3Container.vue'

export default {
  name: 'App',
  components: {
    d3container
  }
}
</script>

<style lang="scss">
.app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background: linear-gradient(180deg, rgb(2, 122, 66) 0%, rgb(0, 255, 136) 27%, rgb(44, 252, 155) 67%, rgba(235,235,235,1) 100%);
  
}
</style>
<style lang="scss">
.v-main__wrap {
  place-self: center;
}
</style>