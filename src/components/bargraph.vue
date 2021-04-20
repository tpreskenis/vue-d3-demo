<template>
  <div>
      <div class="container" style="display: flex; justify-content: space-around;">
        <div>
            <p :class="informationTextClass">{{timeNumber}}</p>
            <p :class="dataSentenceTextClass">{{timeframe}}</p>
        </div>

      </div>
    <div
        id="timeline">
    </div>
    <p :class="focusedYearClass">{{legend}}</p>
    <div class="button-display">
        <v-btn
            fab
            small
            depressed
            @click="years()"
            >
                Y
        </v-btn>
        <v-btn
            fab
            small
            depressed
            @click="quarters()"
            >
                4Q
        </v-btn>
        <v-btn
            fab
            small
            depressed
            @click="twelvemonths()"
            >
                12M
        </v-btn>
    </div>
  </div>
</template>
<script>
import * as d3 from 'd3'
export default {
  data () {
    return {
        settings: {
          height: 250,
          width: 342,
        },
        margin: { top: 30, bottom: 20, left: 30, right: 30 },
        dataset: null,
        months_name: ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'],
        // SVG
        svg: {},
        tooltip: {},
        plot: {},
        xScale: {},
        yScale: {},
        xAxis: {},
        yAxis: {},
        gs: {},
        ps: {},
        rectPadding: 5,

        // Information
        timeframe: 'Years',
        timeNumber: '1',
        informationTextClass: 'number_text invisible',
        dataSentenceTextClass: 'informationTextClass invisible',
        currentNumber: 0,
        focusedYearClass: 'focused-year invisible',

        // Focused Legend
        legend: '',

        // Prediction 
        prediction: 0

    }
  },
  props: {
    quarters_: Array,
    years_: Array,
    months_:Array,
  },
  computed: {
      // Prediction
      yearPrediction: function() {
          var now = new Date('2021-04-19T03:24:00');
          var start = new Date(now.getFullYear(), 0, 0);
          var diff = now - start;
          var oneDay = 1000 * 60 * 60 * 24;
          var currentDay = Math.floor(diff / oneDay);
          if (currentDay > 31) {
              var lastyear= this.dataset[this.dataset.length-1]
              return Math.floor((lastyear.amount/currentDay)*365)
          }
          else 
            return null
      },
      quarterPrediction: function() {
        var d = new Date('2021-04-19T03:24:00');
        var q = Math.ceil((d.getMonth()+1) / 3) 
        q = (q == 1 ? 4 : q-1) 
        // Current Day 
        var now = new Date('2021-04-19T03:24:00');
        var start = new Date(now.getFullYear(), 0, 0);
        var diff = now - start;
        var oneDay = 1000 * 60 * 60 * 24;
        var currentDay = Math.floor(diff / oneDay);
        var daysIntoQuarter = 0
        var daysInQuarter = 0
        if (q == 4) {
            daysIntoQuarter = currentDay
            daysInQuarter = 90
        }
        else if (q == 1) {
            daysIntoQuarter = currentDay - 90
            daysInQuarter = 91
        }
        else if (q == 2) {
            daysIntoQuarter = currentDay - 181
            daysInQuarter = 92
        }
        else if (q == 3) {
            daysIntoQuarter = currentDay - 273
            daysInQuarter = 92
        }
        if (Math.floor(daysIntoQuarter/14) > 0) {
            var lastQuarter= this.dataset[this.dataset.length-1]
            return Math.floor((lastQuarter.amount/daysIntoQuarter)*daysInQuarter)
        }
        else   
            return null
      },
      monthPrediction: function() {
          // Assumes the last month is the current month (I.E. I am making this in April, so it would be April)
          var date = new Date('2021-04-19T03:24:00').getDate()
          var now = new Date('2021-04-19T03:24:00');
          var totalDaysInMonth = new Date(now.getFullYear(), now.getMonth()+1, 0).getDate();
          if (Math.floor(date/7) > 0) {
              var lastmonth= this.dataset[this.dataset.length-1]
              return Math.floor((lastmonth.amount/date)*totalDaysInMonth)
          }
          else 
            return null
      },
      // Vs Average
      currentAverage: function() {
          let amountTotal = 0;
          let length = this.dataset.length;
          this.dataset.forEach(({amount}) => amountTotal += amount)
          return amountTotal/length
      },
      amountDifference: function() {
          return Math.trunc(this.currentNumber -this.currentAverage)
      },
      percentageDifference: function() {
          return Math.trunc(((Math.abs(this.amountDifference))/this.currentAverage)*100)
      },
      DataSentence: function() {
        if (Math.sign(this.amountDifference) == 1)
          return '+' +(Math.abs(this.amountDifference)) + ' 🠕 ' + this.percentageDifference +'%'
        else if (this.amountDifference == 0)
            return '0 - 0%'
        else 
          return '-' +(Math.abs(this.amountDifference)) + ' 🠗 ' + this.percentageDifference +'%'
      },
      years_entire: function() {
         var start = new Date(this.years_[0].year,1,1); 
         var end = new Date('2021-04-19T03:24:00');
         var time_dif = d3.timeYears(start, end); 
         var years = [this.years_[0].year]
         time_dif.forEach(year => 
            years.push(year.getFullYear().toString())
         )
         var entire_years_ = []
         var current_year_dataset = this.years_
         var count = 0;
         years.forEach(year => {
            var index_number = current_year_dataset.map(x=>x.year).indexOf(year)
            if (index_number >= 0)
                entire_years_.push({tic: "'"+year.substring(2,4), amount: current_year_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_years_.push({tic: "'"+year.substring(2,4), amount: 0, x: count})  
            count++          
            }
         )
          return entire_years_
      },
      quarters_entire_past_year: function() {          
         var start = new Date(Math.min.apply(Math, this.quarters_.map(x=>x.year))-1,0,0);
         var end = new Date('2021-04-19T03:24:00');
         var time_dif = d3.timeMonths(start, end); 
         var quarters = []
         time_dif.forEach(month => { 
             if (month.getMonth() == 0)
                quarters.push({year: month.getFullYear(),quarter: 'Q4'})
             else if (month.getMonth() == 3)
                quarters.push({year: month.getFullYear()+1,quarter: 'Q1'})
             else if (month.getMonth() == 6)
                quarters.push({year: month.getFullYear()+1,quarter: 'Q2'})
             else if (month.getMonth() == 9)
                quarters.push({year: month.getFullYear()+1,quarter: 'Q3'})
            }
         )
         var entire_quarters_ = []
         var current_quarters_dataset = this.quarters_
         var count = 0;
         // Get Last 5 Quarters (We can go back and change if we want more than just the last 5)
         var last_year_quarters = quarters.slice((quarters.length-5),quarters.length)
         last_year_quarters.forEach(quarters => {
            var current_quarters = quarters.year + quarters.quarter
            var index_number = current_quarters_dataset.map(x=>x.quarter).indexOf(current_quarters)
            if (index_number >= 0)
                entire_quarters_.push({tic: quarters.quarter, quarter: current_quarters,amount: current_quarters_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_quarters_.push({tic: quarters.quarter, quarter: current_quarters,amount: 0, x: count})      
            count++               
            }
         )
         console.log(entire_quarters_)
          return entire_quarters_
      },
      quarter_legend: function() {
          var current_day = new Date('2021-04-19T03:24:00');
          return 'FY'+(current_day.getFullYear()-1).toString().substring(2,4)+' - FY'+current_day.getFullYear().toString().substring(2,4)
      },
      months_entire_past_year: function() {
         var start = new Date('2021-04-19T03:24:00');
         start.setFullYear(start.getFullYear() -1)
         var end = new Date('2021-04-19T03:24:00');
         var time_dif = d3.timeMonths(start, end); 
         var months = [{year: start.getFullYear(),month: end.getMonth()}]
         time_dif.forEach(month => 
            months.push({year: month.getFullYear(),month: month.getMonth()})
         )
         var entire_months_ = []
         var current_month_dataset = this.months_
         var count = 0;
         months.forEach(month => {
            var current_month = month.year +'-' +month.month
            var index_number = current_month_dataset.map(x=>x.month).indexOf(current_month)
            if (index_number >= 0)
                entire_months_.push({tic: this.months_name[month.month], amount: current_month_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_months_.push({tic: this.months_name[month.month], amount: 0, x: count})      
            count++               
            }
         )
          return entire_months_
      },
      month_legend: function() {
          var current_day = new Date('2021-04-19T03:24:00');
          return this.months_name[current_day.getMonth()] + " '" + (current_day.getFullYear()-1).toString().substring(2,4) + ' - ' + this.months_name[current_day.getMonth()] + " '" + (current_day.getFullYear()).toString().substring(2,4)
      },

  },
  methods: {
      getAvg(grades) {
            const total = grades.reduce((acc, c) => acc + c, 0);
            return total / grades.length;
      },
      twelvemonths: function() {
          this.dataset = this.months_entire_past_year
          this.legend = this.month_legend
          this.prediction = this.monthPrediction
          this.updateAxis()
          this.updateData()
          this.focusedYearClass = 'focused-year'
          this.timeframe = 'Last 12 Months'
      },
      quarters: function() {
          this.dataset = this.quarters_entire_past_year
          this.legend = this.quarter_legend
          this.prediction = this.quarterPrediction
          this.updateAxis()
          this.updateData()
          this.focusedYearClass = 'focused-year'
          this.timeframe = 'Quarters'
      },
      years: function() {
          this.dataset = this.years_entire
          this.prediction = this.yearPrediction
          this.updateAxis()
          this.updateData()
          this.focusedYearClass = 'focused-year invisible'
          this.timeframe = 'Years'
      },      
      drawGraph: function() {
        this.$emit('timeframe','Years')
        // Define the svg element
        this.svg = d3
            .select("#timeline")
            .append("svg")
            .attr("width", this.settings.width)
            .attr("height", this.settings.height);

        // Define the div for the tooltip
        this.tooltip = d3
          .select("#timeline")
          .append("div")
          .attr("class", "timeline-tooltip")
          .style("opacity", 0);

        // Create Plot Areas
        this.plot = this.svg
          .append("g")
          .attr("class", "timeline")
          .attr(
            "transform",
            "translate(" +
              this.margin.left +
              "," +
              this.margin.top +
              ")"
          );
        this.prediction = this.yearPrediction
        this.AxisCreation()
        this.predictionCreation()
        this.dataCreation()
      },
      AxisCreation: function() {
        var maxArray = this.dataset.map(x=>x.amount)
        maxArray.push(this.prediction)
        this.xScale = d3.scaleBand()
            .domain(d3.range(this.dataset.length))
            .rangeRound([0, this.settings.width - this.margin.left - this.margin.right])
        let xAxis = d3.axisBottom(this.xScale).tickFormat((i) => { return this.dataset.map(x=>x.tic)[i]; });
        this.yScale = d3.scaleLinear()
            .domain([0, d3.max(maxArray)])
            .range([this.settings.height - this.margin.top - this.margin.bottom, 0])
        this.xAxis = this.plot.append('g')
            .attr('transform', 'translate(' + 0 + ',' + (this.settings.height - this.margin.top - this.margin.bottom) + ')')
            .call(xAxis)
        let yAxis = d3.axisLeft(this.yScale).ticks(3)
        this.yAxis = this.plot.append('g')
            .attr('transform', 'translate(0, 0)')
            .call(yAxis)
      },
      predictionCreation: function() {
          this.ps = this.plot.selectAll('.rect')
            .data(this.dataset)
            .enter()
            .append('g')
             
        let rectPadding = 5
        let xScale = this.xScale
        let yScale = this.yScale
        let height = this.settings.height
        let top = this.margin.top
        let bottom = this.margin.bottom
        let dataset = this.dataset
        let prediction = this.prediction
        
        this.ps.append('rect')
            .attr('x', function (d, i) {
                return xScale(i) + rectPadding / 2
            })
            .attr('y', function () {
            let min = yScale.domain()[0]
            return yScale(min)
            })
            .attr('width', function () {
            return xScale.step() - rectPadding
            })
            .attr('height', function () {
            return 0
            })
            .attr('fill', '#41B883')
            .attr('opacity',.4)
            .transition()
            .duration(500)
            .delay(function (d, i) {
            return i * 100
            })
            .attr('y', function (d) {
                if (d.x == (dataset.length-1)) {
                    return yScale(prediction)
                }
                else 
                    return yScale(d.amount)
            })
            .attr('height', function (d) {
               if (d.x == (dataset.length-1)) {
                    return height - top - bottom - yScale(prediction)
                }
                else 
                    return height - top - bottom - yScale(d.amount)
            })
            let nestedThis = this

            this.ps.on('mouseover', function(d,i){
                nestedThis.informationTextClass = 'number_text_pred'
                nestedThis.dataSentenceTextClass = 'information_text'
                // Number
                if (nestedThis.prediction>1) {
                    nestedThis.timeNumber = nestedThis.prediction 
                }
                else { 
                    nestedThis.timeNumber = nestedThis.prediction 
                }
                    var old_value = nestedThis.dataset[i.x-1].amount
                    var new_value = nestedThis.prediction

                    // Difference 
                    var amount_difference =  Math.trunc(new_value - old_value)
                    var difference_percent = Math.trunc(((Math.abs(new_value))/old_value)*100)-100
                    if (difference_percent == Infinity)
                        difference_percent = 100
                    if (Math.sign(amount_difference) == 1)
                        nestedThis.timeframe =  '+' +(Math.abs(amount_difference)) + ' 🠕 ' + difference_percent +'%' + ' (Prediction)'
                    else if (amount_difference == 0)
                        nestedThis.timeframe = '0 - 0%' + ' (Prediction)'
                    else 
                        nestedThis.timeframe = '-' +(Math.abs(amount_difference)) + ' 🠗 ' + difference_percent +'%' + ' (Prediction)'

                d3.select(this)
                    .select('rect')
                    .attr("fill", "#34495E")
                    .attr('opacity',.4);
            })
            .on('mouseout', function(){
                nestedThis.dataSentenceTextClass = 'information_text invisible'
                nestedThis.informationTextClass = 'number_text invisible'
                d3.select(this)
                    .select('rect')
                    .attr("fill", "#41B883")
                    .attr('opacity',.4);
            });
      },
      dataCreation: function() {
        this.gs = this.plot.selectAll('.rect')
            .data(this.dataset)
            .enter()
            .append('g')

             
        let rectPadding = 5
        let xScale = this.xScale
        let yScale = this.yScale
        let height = this.settings.height
        let top = this.margin.top
        let bottom = this.margin.bottom


        this.gs.append('rect')
            .attr('x', function (d, i) {
            return xScale(i) + rectPadding / 2
            })
            .attr('y', function () {
            let min = yScale.domain()[0]
            return yScale(min)
            })
            .attr('width', function () {
            return xScale.step() - rectPadding
            })
            .attr('height', function () {
            return 0
            })
            .attr('fill', '#41B883')
            .transition()
            .duration(500)
            .delay(function (d, i) {
            return i * 100
            })
            .attr('y', function (d) {
            return yScale(d.amount)
            })
            .attr('height', function (d) {
            return height - top - bottom - yScale(d.amount)
            })

            let nestedThis = this

            this.gs.on('mouseover', function(d,i){
                nestedThis.informationTextClass = 'number_text'
                nestedThis.dataSentenceTextClass = 'information_text'
                // Number
                if (i.amount>1) {
                    nestedThis.timeNumber = i.amount 
                }
                else { 
                    nestedThis.timeNumber = i.amount 
                }
                if (i.x == 0) {
                    // no month before so will do average!
                    nestedThis.currentNumber = i.amount
                    nestedThis.timeframe = nestedThis.DataSentence + ' VS. AVG'
                }
                else {
                    nestedThis.timeframe = i.amount
                    var old_value = nestedThis.dataset[i.x-1].amount
                    var new_value = i.amount

                    // Difference 
                    var amount_difference =  Math.trunc(new_value - old_value)
                    var difference_percent = Math.trunc(((Math.abs(new_value))/old_value)*100)-100
                    if (difference_percent == Infinity)
                        difference_percent = 100
                    if (Math.sign(amount_difference) == 1)
                        nestedThis.timeframe =  '+' +(Math.abs(amount_difference)) + ' 🠕 ' + difference_percent +'%'
                    else if (amount_difference == 0)
                        nestedThis.timeframe = '0 - 0%'
                    else 
                        nestedThis.timeframe = '-' +(Math.abs(amount_difference)) + ' 🠗 ' + difference_percent +'%'
                }

                // Month Over Month
                d3.select(this)
                    .select('rect')
                    .attr("fill", "#34495E");
            })
            .on('mouseout', function(){
                nestedThis.dataSentenceTextClass = 'information_text invisible'
                nestedThis.informationTextClass = 'number_text invisible'
                d3.select(this)
                    .select('rect')
                    .attr("fill", "#41B883");
            });
      },

      // Update
      updateAxis: function() {
        var maxArray = this.dataset.map(x=>x.amount)
        maxArray.push(this.prediction)
        this.xScale = d3.scaleBand()
            .domain(d3.range(this.dataset.length))
            .rangeRound([0, this.settings.width - this.margin.left - this.margin.right])
        this.yScale = d3.scaleLinear()
            .domain([0, d3.max(maxArray)])
            .range([this.settings.height - this.margin.top - this.margin.bottom, 0])
        let xAxis = d3.axisBottom(this.xScale).tickFormat((i) => { return this.dataset.map(x=>x.tic)[i]; });
        this.xAxis.call(xAxis)
        this.yAxis.call(g => g.selectAll(".tick").remove())
        this.yAxis.call(g => g.selectAll(".domain").remove())
        let yAxis = d3.axisLeft(this.yScale).ticks(3)
        this.yAxis.call(yAxis)
      },
      updateData: function() {
          this.gs.remove()
          this.ps.remove()
          this.predictionCreation()
          this.dataCreation()
      }
  },
  mounted () {
      this.dataset = this.years_entire
      this.drawGraph()
    }
}
</script>
<style scoped lang="scss">
.button-display {
    display: flex;
    justify-content: space-evenly;
    padding-bottom: 15px;
}
.information_text {
    transition: all 0.3s ease;
    margin-bottom: 0px !important;
    color: #34495E;
    font-size: small;
    font-weight: bold;
    text-align: -webkit-center;
    opacity: 1;
}
.number_text {
    transition: all 0.3s ease;
    margin-bottom: 0px;
    text-align-last: center;
    font-weight: bold;
    font-size: x-large;
    opacity: 1;
}
.number_text_pred {
    transition: all 0.3s ease;
    color: grey;
    margin-bottom: 0px;
    text-align-last: center;
    font-weight: bold;
    font-size: x-large;
    opacity: 1;
}
.invisible {
    opacity: 0 !important;
}
.focused-year {
    transition: all 0.3s ease;
    text-align: center;
    font-size: x-small;
    margin-bottom: 8px;
    color: gray;
    font-style: italic;
    opacity: 1;
}
.chart-button {
    transition: all 0.3s ease;
    place-self: center;
    opacity: 1;
}

</style>