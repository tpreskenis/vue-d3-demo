<template>
    <div>
        <div class="container" style="justify-content: center; display: flex;">
         <v-list-item-subtitle :class="yearClass">
             {{data_time}}:
         </v-list-item-subtitle>
         <div class="text-display">
         <v-list-item-title :class="data1TextClass">
             {{data_one_y}}
         </v-list-item-title>
         <p :class="labelClass">(Current)</p>
         </div>
            <v-divider
              vertical
            ></v-divider>
         <div class="text-display">
         <v-list-item-title :class="data2TextClass">
             {{data_two_y}}
         </v-list-item-title>
         <p :class="labelClass">(Projected)</p>
         </div>
        </div>
        <svg id="myLineChart" ref="myLineChart" width="342px" height="250px"></svg>
        <p :class="bottomLabelClass">{{focusedText}}</p>
        <div class="button-display">
            <v-btn
                fab
                small
                depressed
                @click="updateYear()"
                >
                    Y
            </v-btn>
            <v-btn
                fab
                small
                depressed
                @click="updateQuarter()"
                >
                    4Q
            </v-btn>
            <v-btn
                fab
                small
                depressed
                @click="updateMonth()"
                >
                    12M
            </v-btn>
        </div>
    </div>
</template>
<script>
import * as d3 from "d3";
export default {
  name: 'line-chart',
  props: {
    year1_: Array,
    year2_: Array,
    month1_: Array,
    month2_: Array,
    quarter1_: Array,
    quarter2_: Array,
  },
  data () {
    return {
        settings: {
          height: 250,
          width: 342,
        },
        margin: { top: 30, bottom: 20, left: 30, right: 30 },
        height: null,
        width: null,
        n: null,
        svg: {},
        xScale: {},
        yScale: {},
        xAxis: {},
        yAxis: {},

        // SVG
        tooltip: {},
        plot: {},
        gs: {},
        linepath: {},
        rectPadding: 5,
        dotted_line: {},

        // Information
        data_time: '2020',
        data_two_y: 1,
        data_one_y: 1,
        data1TextClass: 'green_text invisible',
        data2TextClass: 'dark_blue_text invisible',
        yearClass: 'year invisible',
        labelClass: 'label invisible',
        bottomLabelClass: 'focused invisible',
        focusedText: 'test',

        // Height
        maxHeight: null,

        // Months
        months_name: ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'],

        // Prediction 
        prediction: 0
    }
  },
  computed: {
    // MaxPredictions 
      yearPredictionMax: function() {
          var now = new Date('2021-04-19T03:24:00');
          var start = new Date(now.getFullYear(), 0, 0);
          var diff = now - start;
          var oneDay = 1000 * 60 * 60 * 24;
          var currentDay = Math.floor(diff / oneDay);
          if (currentDay > 31) {
              var lastyear1= this.every_year_set_1[this.every_year_set_1.length-1]
              var maxyear1 = Math.floor((lastyear1.y/currentDay)*365)
              var lastyear2= this.every_year_set_2[this.every_year_set_2.length-1]
              var maxyear2 = Math.floor((lastyear2.y/currentDay)*365)
              if (maxyear1 > maxyear2)
                return maxyear1
              else 
                return maxyear2
          }
          else 
            return null
      },
      monthPredictionMax: function() {
          var date = new Date('2021-04-19T03:24:00').getDate()
          var now = new Date('2021-04-19T03:24:00');
          var totalDaysInMonth = new Date(now.getFullYear(), now.getMonth()+1, 0).getDate();
          if (Math.floor(date/7) > 0) {
              var lastmonth1= this.months_entire_past_year_1[this.months_entire_past_year_1.length-1]
              var maxmonth1 =  Math.floor((lastmonth1.y/date)*totalDaysInMonth)
              var lastmonth2= this.months_entire_past_year_2[this.months_entire_past_year_2.length-1]
              var maxmonth2 =  Math.floor((lastmonth2.y/date)*totalDaysInMonth)
              if (maxmonth1 > maxmonth2)
                return maxmonth1
              else 
                return maxmonth2
          }
          else 
            return null
      },
      quarterPredictionMax: function() {
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
          var lastQuarter1= this.quarters_entire_past_year_1[this.quarters_entire_past_year_1.length-1]
          var maxquarter1 =  Math.floor((lastQuarter1.y/daysIntoQuarter)*daysInQuarter)
          var lastQuarter2= this.quarters_entire_past_year_2[this.quarters_entire_past_year_2.length-1]
          var maxquarter2 =  Math.floor((lastQuarter2.y/daysIntoQuarter)*daysInQuarter)
          if (maxquarter1 > maxquarter2)
            return maxquarter1
          else 
            return maxquarter2
        }
        else   
            return null
      },

      currentDay: function() {
          var now = new Date('2021-04-19T03:24:00');
          var start = new Date(now.getFullYear(), 0, 0);
          var diff = now - start;
          var oneDay = 1000 * 60 * 60 * 24;
          return Math.floor(diff / oneDay);
      },  
      daysInCurrentMonth: function() {
          var now = new Date('2021-04-19T03:24:00');
          return new Date(now.getFullYear(), now.getMonth()+1, 0).getDate();
      },  
      currentMonthDate: function() {
          return new Date('2021-04-19T03:24:00').getDate();
      },  
      currentMonth: function() {
          return new Date('2021-04-19T03:24:00').getMonth();
      },  
      currentYear: function() {
          return new Date('2021-04-19T03:24:00').getFullYear();
      },
      currentQuarter: function() {
        var d = new Date('2021-04-19T03:24:00');
        var q = Math.ceil((d.getMonth()+1) / 3) 
        var y = d.getFullYear() + (q == 1 ? 0 : 1)
        q = (q == 1 ? 4 : q-1) 
        return y.toString()+'Q'+q.toString()
      },
      start_year: function() {
        // Right now we are taking all time, but we can easily change this
        if (Number(this.year1_[0].year > this.year2_[0].year))
          return this.year2_[0].year 
        else return this.year1_[0].year 
      },
      every_year_set_1: function() {
         var start = new Date(Number(this.start_year)-1,1,1); 
         var end = new Date('2021-04-19T03:24:00');
         var time_dif = d3.timeYears(start, end); 
         var years = []
         time_dif.forEach(year => 
            years.push(year.getFullYear().toString())
         )
         var entire_years_ = []
         var current_year_dataset = this.year1_
         var count = 0;
           years.forEach(year => {
            var index_number = current_year_dataset.map(x=>x.year).indexOf(year)
            if (index_number >= 0)
                entire_years_.push({g: 1,tic: "'"+year.substring(2,4), d: year,y: current_year_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_years_.push({g: 1,tic: "'"+year.substring(2,4), d: year,y: 0, x: count})  
              count++         
            }
         )
        return entire_years_
      },
      every_year_set_2: function() {
         var start = new Date(Number(this.start_year)-1,1,1); 
         var end = new Date('2021-04-19T03:24:00');
         var time_dif = d3.timeYears(start, end); 
         var years = []
         time_dif.forEach(year => 
            years.push(year.getFullYear().toString())
         )
         var entire_years_ = []
         var current_year_dataset = this.year2_
         var count = 0;
           years.forEach(year => {
            var index_number = current_year_dataset.map(x=>x.year).indexOf(year)
            if (index_number >= 0)
                entire_years_.push({g: 2,tic: "'"+year.substring(2,4),d: year, y: current_year_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_years_.push({g: 2,tic: "'"+year.substring(2,4),d: year, y: 0, x: count})  
              count++         
            }
         )
        return entire_years_
      },
      months_entire_past_year_1: function() {
         var start = new Date('2021-04-19T03:24:00');
         start.setFullYear(start.getFullYear() -1)
         var end = new Date('2021-04-19T03:24:00');
         var time_dif = d3.timeMonths(start, end); 
         var months = [{year: start.getFullYear(),month: end.getMonth()}]
         time_dif.forEach(month => 
            months.push({year: month.getFullYear(),month: month.getMonth()})
         )
         var entire_months_ = []
         var current_month_dataset = this.month1_
         var count = 0;
         months.forEach(month => {
            var current_month = month.year +'-' +month.month
            var index_number = current_month_dataset.map(x=>x.month).indexOf(current_month)
            if (index_number >= 0)
                entire_months_.push({g:1, d: month.month, tic: this.months_name[month.month], y: current_month_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_months_.push({g:1, d: month.month, tic: this.months_name[month.month], y: 0, x: count})      
            count++    
                  
            }
         )
          return entire_months_
      },
      months_entire_past_year_2: function() {
         var start = new Date('2021-04-19T03:24:00');
         start.setFullYear(start.getFullYear() -1)
         var end = new Date('2021-04-19T03:24:00');
         var time_dif = d3.timeMonths(start, end); 
         var months = [{year: start.getFullYear(),month: end.getMonth()}]
         time_dif.forEach(month => 
            months.push({year: month.getFullYear(),month: month.getMonth()})
         )
         var entire_months_ = []
         var current_month_dataset = this.month2_
         var count = 0;
         months.forEach(month => {
            var current_month = month.year +'-' +month.month
            var index_number = current_month_dataset.map(x=>x.month).indexOf(current_month)
            if (index_number >= 0)
                entire_months_.push({g:2, d: month.month, tic: this.months_name[month.month], y: current_month_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_months_.push({g:2, d: month.month, tic: this.months_name[month.month], y: 0, x: count})      
            count++    
                  
            }
         )
          return entire_months_
      },
      quarters_entire_past_year_1: function() {          
         var start = new Date(Math.min.apply(Math, this.quarter1_.map(x=>x.year))-1,0,0);
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
         var current_quarters_dataset = this.quarter1_
         var count = 0;
         // Get Last 5 Quarters (We can go back and change if we want more than just the last 5)
         var last_year_quarters = quarters.slice((quarters.length-5),quarters.length)
         last_year_quarters.forEach(quarters => {
            var current_quarters = quarters.year + quarters.quarter
            var index_number = current_quarters_dataset.map(x=>x.quarter).indexOf(current_quarters)
            if (index_number >= 0)
                entire_quarters_.push({g: 1, tic: quarters.quarter, d: current_quarters,y: current_quarters_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_quarters_.push({g: 1, tic: quarters.quarter, d: current_quarters,y: 0, x: count})      
            count++               
            }
         )
          return entire_quarters_
      },
      quarters_entire_past_year_2: function() {          
         var start = new Date(Math.min.apply(Math, this.quarter2_.map(x=>x.year))-1,0,0);
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
         var current_quarters_dataset = this.quarter2_
         var count = 0;
         // Get Last 5 Quarters (We can go back and change if we want more than just the last 5)
         var last_year_quarters = quarters.slice((quarters.length-5),quarters.length)
         last_year_quarters.forEach(quarters => {
            var current_quarters = quarters.year + quarters.quarter
            var index_number = current_quarters_dataset.map(x=>x.quarter).indexOf(current_quarters)
            if (index_number >= 0)
                entire_quarters_.push({g: 2,tic: quarters.quarter, d: current_quarters,y: current_quarters_dataset.map(x=>x.amount)[index_number], x: count})
            else 
                entire_quarters_.push({g: 2,tic: quarters.quarter, d: current_quarters,y: 0, x: count})      
            count++               
            }
         )
          return entire_quarters_
      },
      quarter_legend: function() {
          var current_day = new Date('2021-04-19T03:24:00');
          return 'FY'+(current_day.getFullYear()-1).toString().substring(2,4)+' - FY'+current_day.getFullYear().toString().substring(2,4)
      },
      month_legend: function() {
          var current_day = new Date('2021-04-19T03:24:00');
          return this.months_name[current_day.getMonth()] + " '" + (current_day.getFullYear()-1).toString().substring(2,4) + ' - ' + this.months_name[current_day.getMonth()] + " '" + (current_day.getFullYear()).toString().substring(2,4)
      },
  },
  mounted() {
    this.drawChart();
  },
  methods: {
    drawSVG() {
      this.width = this.settings.width - this.margin.left - this.margin.right,
      this.height = this.settings.height - this.margin.top - this.margin.bottom;
      // Creating SVG
      this.svg = d3
        .select("#myLineChart")
        .append("svg")
        .attr("width", this.width + this.margin.left + this.margin.right)
        .attr("height", this.height + this.margin.top + this.margin.bottom)
        .append("g")
        .attr("transform", "translate(" + this.margin.left + "," + this.margin.top + ")");
    },
    setScales(data1,data2,ticks) {
      this.n = data1.length;
      
      if (Math.max(...(data1.map(a => a.y))) > Math.max(...data2.map(a => a.y)))
        this.maxHeight = Math.max(...data1.map(a => a.y))
      else 
        this.maxHeight = Math.max(...data2.map(a => a.y))

      // If the prediction is higher
      if(this.maxHeight < this.prediction)
        this.maxHeight = this.prediction

      this.xScale = d3
        .scaleLinear()
        .domain([0, this.n - 1]) // input
        .range([0, this.width]); // output

      this.yScale = d3
        .scaleLinear()
        .domain([0, this.maxHeight]) // input
        .range([this.height, 0]); // output

      var xScale = this.xScale

      var yScale = this.yScale

      this.ticks = ticks

      this.xAxis = this.svg
        .append("g")
        .attr("class", "x axis")
        .attr("transform", "translate(0," + this.height + ")")
        .call(d3.axisBottom(xScale).ticks(5).tickFormat((i) => { return this.ticks[i]; }));

      this.yAxis = this.svg
        .append("g")
        .attr("class", "y axis")
        .call(d3.axisLeft(yScale).ticks(3).tickValues([0,this.maxHeight/2,this.maxHeight]));
    },
    drawline(data,color,time) {
      var xScale = this.xScale

      var yScale = this.yScale

      this.line = d3
        .line()
        .x(function(d, i) {
          return xScale(i);
        })
        .y(function(d) {
          return yScale(d.y);
        });
      if (time == 'year') {
        var dataset = data.slice(0,data.length-1);
        var dataset_line = data.slice(0,data.length-1);
        dataset_line.push({d: data[data.length-1].d , x: data[data.length-1].x ,y: Math.floor((data[data.length-1].y / this.currentDay) * 365)})
      }
      else if (time == 'quarter') {
        dataset = data.slice(0,data.length-1);
        dataset_line = data.slice(0,data.length-1);
        dataset_line.push({d: data[data.length-1].d , x: data[data.length-1].x ,y: this.quarterPrediction(data)})   
      }
      else if (time == 'month') {
        dataset = data.slice(0,data.length-1);
        dataset_line = data.slice(0,data.length-1);
        dataset_line.push({d: data[data.length-1].d , x: data[data.length-1].x ,y: Math.floor((data[data.length-1].y / this.currentMonthDate) * this.daysInCurrentMonth)}) 
      }
      else {
        dataset = data
        dataset_line = data
      }
      var line = this.line;
      var dotted = this.dotted_line
      this.linepath = this.svg
        .append("path")
        .datum(dataset)
        .attr("class", color)
        .attr("d", line)
        .call(function(path) {
          path
            .transition()
            .duration(1000)     
            .attrTween("stroke-dasharray", (function() {
              var l = this.getTotalLength(),
                i = d3.interpolateString("0," + l, l + "," + l);
              return function(t) {
                    if (t == 1)
                        dotted.transition().duration(1000).style("opacity", '1')
                    return i(t);
              };
            }));
        });
      dotted = this.svg
        .append("path")
        .datum(dataset_line)
        .attr("class", color)
        .attr("d", line)
        .style("opacity", '0')
        .style("stroke-dasharray", "4,4");
    },

    drawDots(data, color, time) {
      var xScale = this.xScale

      var yScale = this.yScale

      var dataset = data.slice(0,data.length-1);
      dataset.push({g: dataset[1].g,d: data[data.length-1].d , x: data[data.length-1].x ,y: Math.floor((data[data.length-1].y / this.currentDay) * 365)})
      var data1 = this.every_year_set_1
      var data2 = this.every_year_set_2
      if (time == 'year') {
      data1 = this.every_year_set_1
      data2 = this.every_year_set_2
      }
      else if (time == 'month') {
      dataset = data.slice(0,data.length-1);
      dataset.push({g: dataset[1].g, d: data[data.length-1].d , x: data[data.length-1].x ,y: Math.floor((data[data.length-1].y / this.currentMonthDate) * this.daysInCurrentMonth)}) 
      data1 = this.months_entire_past_year_1
      data2 = this.months_entire_past_year_2    
      }
      else if (time == 'quarter') {
      dataset = data.slice(0,data.length-1);
      dataset.push({g: dataset[1].g, d: data[data.length-1].d , x: data[data.length-1].x ,y: this.quarterPrediction(data)}) 
      data1 = this.quarters_entire_past_year_1
      data2 = this.quarters_entire_past_year_2    
      }
      let nestedThis = this

      this.svg
        .selectAll("g.dot")
        .data(dataset)
        .enter()
        .append("circle")
        .attr("class", "dot")
        .attr("cx", function(d, i) {
          return xScale(i);
        })
        .attr("cy", function(d) {
          return yScale(d.y);
        })
        .attr("r", 3)
        .on("mouseover", function(d,i) {
          
          nestedThis.data_time = data1.find(o => o.x === i.x).d;

          if (time == 'month') {
            nestedThis.data_time = nestedThis.ticks[data1.find(o => o.x === i.x).x];
          }
          else if (time == 'quarter') {
            nestedThis.data_time = data1[data1.find(o => o.x === i.x).x].d;
          }

          if (nestedThis.data_time == nestedThis.currentYear && i.g == 1 && time == 'year') {
            nestedThis.data_one_y = data1.find(o => o.x === i.x).y;
            nestedThis.data_two_y = Math.floor((data1[data1.length-1].y / nestedThis.currentDay) * 365)
            nestedThis.data1TextClass = 'green_text'
            nestedThis.data2TextClass = 'green_text'
            nestedThis.labelClass = 'label'
          }
          else if (nestedThis.data_time == nestedThis.currentYear && i.g == undefined && time == 'year') {
            nestedThis.data_one_y = data2.find(o => o.x === i.x).y;
            nestedThis.data_two_y = Math.floor((data2[data2.length-1].y / nestedThis.currentDay) * 365)
            nestedThis.data1TextClass = 'match_text'
            nestedThis.data2TextClass = 'match_text'
            nestedThis.labelClass = 'label'
          }
          else if (nestedThis.data_time == nestedThis.currentYear && i.g == 2 && time == 'year') {
            nestedThis.data_one_y = data2.find(o => o.x === i.x).y;
            nestedThis.data_two_y = Math.floor((data2[data2.length-1].y / nestedThis.currentDay) * 365)
            nestedThis.data1TextClass = 'dark_blue_text'
            nestedThis.data2TextClass = 'dark_blue_text'
            nestedThis.labelClass = 'label'
          }
          else if (nestedThis.data_time == nestedThis.months_name[nestedThis.currentMonth] && i.g == undefined && time == 'month') {
            nestedThis.data_one_y = data1.find(o => o.x === i.x).y;
            nestedThis.data_two_y = Math.floor((data1[data1.length-1].y / nestedThis.currentMonthDate) * nestedThis.daysInCurrentMonth)
            nestedThis.data1TextClass = 'match_text'
            nestedThis.data2TextClass = 'match_text'
            nestedThis.labelClass = 'label'
          }
          else if (nestedThis.data_time == nestedThis.months_name[nestedThis.currentMonth] && i.g == 1 && time == 'month' && (data1.find(o => o.x === i.x).x == data.length-1)) {
            nestedThis.data_one_y = data1.find(o => o.x === i.x).y;
            nestedThis.data_two_y = Math.floor((data1[data1.length-1].y / nestedThis.currentMonthDate) * nestedThis.daysInCurrentMonth)
            nestedThis.data1TextClass = 'green_text'
            nestedThis.data2TextClass = 'green_text'
            nestedThis.labelClass = 'label'
          }
          else if (nestedThis.data_time == nestedThis.months_name[nestedThis.currentMonth] && i.g == 2 && time == 'month' && (data2.find(o => o.x === i.x).x == data.length-1)) {
            nestedThis.data_one_y = data2.find(o => o.x === i.x).y;
            nestedThis.data_two_y = Math.floor((data2[data2.length-1].y / nestedThis.currentMonthDate) * nestedThis.daysInCurrentMonth)
            nestedThis.data1TextClass = 'dark_blue_text'
            nestedThis.data2TextClass = 'dark_blue_text'
            nestedThis.labelClass = 'label'
          }
          else if (nestedThis.data_time == nestedThis.currentQuarter && i.g == undefined && time == 'quarter') {
            nestedThis.data_two_y = data1.find(o => o.x === i.x).y;
            nestedThis.data_one_y = nestedThis.quarterPrediction(data1)
            nestedThis.data1TextClass = 'match_text'
            nestedThis.data2TextClass = 'match_text'
            nestedThis.labelClass = 'label'
          }
          else if (nestedThis.data_time == nestedThis.currentQuarter && i.g == 1 && time == 'quarter') {
            nestedThis.data_two_y = data1.find(o => o.x === i.x).y;
            nestedThis.data_one_y = nestedThis.quarterPrediction(data1)
            nestedThis.data1TextClass = 'green_text'
            nestedThis.data2TextClass = 'green_text'
            nestedThis.labelClass = 'label'
          }
          else if (nestedThis.data_time == nestedThis.currentQuarter && i.g == 2 && time == 'quarter') {
            nestedThis.data_two_y = data2.find(o => o.x === i.x).y;
            nestedThis.data_one_y = nestedThis.quarterPrediction(data2)
            nestedThis.data1TextClass = 'dark_blue_text'
            nestedThis.data2TextClass = 'dark_blue_text'
            nestedThis.labelClass = 'label'
          }
          else {
            nestedThis.data_two_y = data2.find(o => o.x === i.x).y;
            nestedThis.data_one_y = data1.find(o => o.x === i.x).y;
            nestedThis.data1TextClass = 'green_text'
            nestedThis.data2TextClass = 'dark_blue_text'
            nestedThis.labelClass = 'label invisible'
          }
          nestedThis.yearClass = 'year'
          d3.select(this)
            .transition()
            .duration(200)
            .attr("r", 5.5)
            .style("fill", color);
        })                  
        .on("mouseout", function() {
          nestedThis.data1TextClass = 'green_text invisible'
          nestedThis.data2TextClass = 'dark_blue_text invisible'
          nestedThis.yearClass = 'year invisible'
          nestedThis.labelClass = 'label invisible'
          
          d3.select(this)
            .transition()
            .duration(200)
            .attr("r", 3.0)
            .style("fill", "#989a99");
        });
    },
    // Update Functions
    updateClear: function() {
        this.svg.selectAll('path').remove()
        this.svg.selectAll('.dot').remove()
        this.svg.selectAll('g').remove()
    },
    updateYear: function() {
        this.updateClear()
        this.bottomLabelClass = 'focused invisible'
        this.focusedText= '',
        this.prediction = this.yearPredictionMax

        this.setScales(this.every_year_set_1,this.every_year_set_2,this.every_year_set_1.map(x=>x.tic))
        this.drawline(this.every_year_set_1,"green",'year')
        this.drawline(this.every_year_set_2,"dark_blue",'year')
        this.drawDots(this.every_year_set_1, "#41B883",'year')
        this.drawDots(this.every_year_set_2, "#34495E",'year')
    },
    updateQuarter: function() {
        this.updateClear()
        this.bottomLabelClass = 'focused'
        this.focusedText= this.quarter_legend
        this.prediction = this.quarterPredictionMax

        this.setScales(this.quarters_entire_past_year_1,this.quarters_entire_past_year_2,this.quarters_entire_past_year_1.map(x=>x.tic))
        this.drawline(this.quarters_entire_past_year_1,"green",'quarter')
        this.drawline(this.quarters_entire_past_year_2,"dark_blue",'quarter')
        this.drawDots(this.quarters_entire_past_year_1, "#41B883",'quarter')
        this.drawDots(this.quarters_entire_past_year_2, "#34495E",'quarter')
    },
    updateMonth: function() {
        this.updateClear()
        this.bottomLabelClass = 'focused'
        this.focusedText= this.month_legend
        this.prediction = this.monthPredictionMax

        this.setScales(this.months_entire_past_year_1,this.months_entire_past_year_2,this.months_entire_past_year_1.map(x=>x.tic))
        this.drawline(this.months_entire_past_year_1,"green",'month')
        this.drawline(this.months_entire_past_year_2,"dark_blue",'month')
        this.drawDots(this.months_entire_past_year_1, "#41B883",'month')
        this.drawDots(this.months_entire_past_year_2, "#34495E",'month')
    },
    // Sets intial to a year!
    drawChart() {
      this.drawSVG()
      this.prediction = this.yearPredictionMax

      this.setScales(this.every_year_set_1,this.every_year_set_2,this.every_year_set_1.map(x=>x.tic))
      this.drawline(this.every_year_set_1,"green",'year')
      this.drawline(this.every_year_set_2,"dark_blue",'year')
      this.drawDots(this.every_year_set_1, "#41B883",'year')
      this.drawDots(this.every_year_set_2, "#34495E",'year')
    },
    // Prediction (Quarter)
      quarterPrediction: function(data) {
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
            var lastQuarter= data[data.length-1]
            return Math.floor((lastQuarter.y/daysIntoQuarter)*daysInQuarter)
        }
        else   
            return 0
      },

  }
};
</script>
<style>
.green {
  fill: none;
  stroke: #41B883;
  stroke-width: 1.2;
}
.dark_blue {
  fill: none;
  stroke: #34495E;
  stroke-width: 1.2;
}
.match_text {
    transition: all 0.3s ease;
    text-align: center;
    background: -webkit-linear-gradient(#34495E, #41B883);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    margin-bottom: 4px !important;
    font-size: 1.5rem !important;
    font-weight: 400;
    line-height: 2rem;
    letter-spacing: normal !important;
    font-family: "Roboto", sans-serif !important;
    opacity: 1;
}
.dark_blue_text {
    transition: all 0.3s ease;
    text-align: center;
    color: #34495E;
    margin-bottom: 4px !important;
    font-size: 1.5rem !important;
    font-weight: 400;
    line-height: 2rem;
    letter-spacing: normal !important;
    font-family: "Roboto", sans-serif !important;
    opacity: 1;
}
.green_text {
    transition: all 0.3s ease;
    text-align: center;
    color: #41B883;
    margin-bottom: 4px !important;
    font-size: 1.5rem !important;
    font-weight: 400;
    line-height: 2rem;
    letter-spacing: normal !important;
    font-family: "Roboto", sans-serif !important;
    opacity: 1;
}
.divider {
    margin-left: 10%;
    margin-right: 10%;
}
.year {
    transition: all 0.3s ease;
    text-align-last: center;
    font-style: italic;
    color: black;
    opacity: 1;
    font-weight: bold;
    font-size: 1.5rem !important;
    line-height: 2rem;
}
.label {
    transition: all 0.3s ease;
    font-size: x-small;
    font-style: italic;
    color: gray;
    margin-bottom: 0px !important;
    opacity: 1;
}
.invisible {
    opacity: 0 !important;
}
.text-display {
    margin-left: 25px;
    margin-right: 25px;
}
.dot {
  fill: #989a99;
  stroke: #fff;
}
.tick > line.currentColor {
  stroke: #fff;
}

.x.axis > .domain,
.y.axis > .domain {
  stroke: #fff;
}
.focused {
    transition: all 0.3s ease;
    text-align: center;
    font-size: x-small;
    margin-bottom: 8px;
    color: gray;
    font-style: italic;
    opacity: 1;
}
.button-display {
    display: flex;
    justify-content: space-evenly;
    padding-bottom: 15px;
}

</style>