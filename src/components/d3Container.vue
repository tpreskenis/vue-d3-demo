<template>
  <div>
  <v-card
    class="mx-auto rounded-xl"
    max-width="344" 
    min-height=345   
  >
  <div style="text-align: -webkit-center;">
    <v-img  
      max-width="250" 
      :src="require('../assets/Vue-d3-image.jpg')">
    </v-img>
    <p class="info-text">A demo of D3's interactive data visualizations inside of a Vue componet.</p>
  </div>
  <switchd3 @changeD3="changeD3"/>
    <bargraph
    v-if="!line"
    :years_="year"
    :quarters_="quarter"
    :months_="month"/>
    <linegraph
    v-if="line"
    :year1_="year1"
    :year2_="year2"
    :month1_="month1"
    :month2_="month2"
    :quarter1_="quarter1"
    :quarter2_="quarter2"/>
  </v-card>
  </div>
</template>

<script>
import switchd3 from './switch.vue';
import bargraph from './bargraph.vue';
import linegraph from './linegraph.vue';

export default {
  components: {
    switchd3, 
    bargraph,
    linegraph
  },
  name: 'Container',
    data: function () {
        return {
            line: false
        }
    },
  methods: {
    changeD3(changeD3) {
      this.line = changeD3
    }
  },
  computed: {
    // bar data
    year: function() {
        const whole = require('../data/bardata.json'); 
        return whole.year
      },
    quarter: function() {
        const whole = require('../data/bardata.json'); 
        return whole.quarter
      },
    month: function() {
        const whole = require('../data/bardata.json'); 
        return whole.month
      },
    // line data
    year1: function() {
        const whole = require('../data/linedata.json'); 
        return whole.year1
      },
    year2: function() {
        const whole = require('../data/linedata.json'); 
        return whole.year2
      },
    quarter1: function() {
        const whole = require('../data/linedata.json'); 
        return whole.quarter1
      },
    quarter2: function() {
        const whole = require('../data/linedata.json'); 
        return whole.quarter2
      },
    month1: function() {
        const whole = require('../data/linedata.json'); 
        return whole.month1
      },
    month2: function() {
        const whole = require('../data/linedata.json'); 
        return whole.month2
      },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.info-text {
  color: gray;
  font-style: italic;
}

</style>
