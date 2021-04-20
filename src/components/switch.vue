<template>
      <div class="switch-wrapper">
        <div class="option_one" v-on:click="switch_('bar')"/>
            <p :class="text_class_one">Bargraph<p/>
        <div class="option_two" v-on:click="switch_('line')"/>
            <div :class="text_class_two">Linegraph</div>
        <input v-model="checkbox" type="checkbox" class="checkbox" id="checkbox"/>
        <label for="checkbox" class="switch"></label>
    </div>
</template>
<script>
export default {
    name: "switch-sports",
    data: function () {
        return {
            checkbox: false
        }
    },
  // define methods under the `methods` object
  methods: {
    switch_: function (sport) {
        if (sport == 'line') {
            if (!this.checkbox) {
                this.checkbox = true
                this.$emit('changeD3', true)
            }
        }
        else if (sport == 'bar') {
            if (this.checkbox) {
                this.checkbox = false
                this.$emit('changeD3', false)
            }
        }
    }
    },
    computed: {
      // assuming this is the text on the left
      text_class_one: function () {
            if (!this.checkbox)
                return "toggle_text_one active_green"
            else   
                return "toggle_text_one not_active"
      },
      // assuming this is the text on the right
      text_class_two: function () {
            if (this.checkbox)
                return "toggle_text_two active_blue"
            else  
                return "toggle_text_two not_active"
      }
    }
  }
</script>
<style lang="scss">
.switch-wrapper {
  position: relative;
  display: block;
  width: 330px;
  margin: 3rem auto 0;

  .option_one{
    width: 165px;
    height: 50px;
    position: absolute;
    z-index: 2;
    cursor: pointer;
  }
  
  .toggle_text_one {
      font-family: 'Nunito', sans-serif;
      position: absolute;
      top: 0;
      right: 0;
      left: -50%;
      z-index: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100%;
      margin: auto;
      font-size: 20px;
      font-weight: bold;
      text-align: center;
      pointer-events: none;
      transition: all 0.3s;
  }


  .option_two{
    width: 165px;
    left: 165px;
    height: 50px;
    position: absolute;
    z-index: 2;
    cursor: pointer;
  }

  .toggle_text_two {
      font-family: 'Nunito', sans-serif;
      position: absolute;
      top: 0;
      right: 0;
      left: 50%;
      z-index: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100%;
      margin: auto;
      font-size: 20px;
      font-weight: bold;
      text-align: center;
      pointer-events: none;
      transition: all 0.3s;
  }

  .active_green {
      color: #41B883
  }
  
  .active_blue {
      color: #34495E
  }

  .not_active {
      color: #878787
  }

  .switch {
    position: relative;
    display: block;
    width: 100%;
    height: 50px;
    margin: 0;
    border: 3px solid #F0F0F0;
    border-radius: 20px;
    transition: all 0.3s;
    background: #F0F0F0;

    &::before {
      position: absolute;
      top: 0;
      left: 0;
      width: 165px;
      height: 100%;
      background-color: rgb(255, 255, 255);
      border-radius: 18px;
      content: "";
      transition: all 0.3s;
    }
  }

  .checkbox {
    display: none;

    &:checked + .switch {
      background-color: #F0F0F0;

      &::before {
        left : 159px;
      }
    }
  }
}
</style>