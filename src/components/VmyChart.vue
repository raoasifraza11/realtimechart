<template>
  <div>
    <h1>{{title}}</h1>
    <div class="hello" id="chartdiv"></div>
  </div>
</template>

<script>


import * as am5 from '@amcharts/amcharts5';
import * as am5xy from '@amcharts/amcharts5/xy';
import am5themes_Animated from '@amcharts/amcharts5/themes/Animated';


export default {
  name: 'MyChart',
  props: {
    title: String,
  },
  data() {
    return {
    }
  },

  methods: {
   displayVChart(){
     let root = am5.Root.new("chartdiv");


// Set themes
// https://www.amcharts.com/docs/v5/concepts/themes/
     root.setThemes([
       am5themes_Animated.new(root)
     ]);


// Generate random data
     let value = 100;

     function generateChartData() {
       let chartData = [];
       let firstDate = new Date();
       firstDate.setDate(firstDate.getDate() - 1000);
       firstDate.setHours(0, 0, 0, 0);

       for (var i = 0; i < 50; i++) {
         let newDate = new Date(firstDate);
         newDate.setSeconds(newDate.getSeconds() + i);

         value += (Math.random() < 0.5 ? 1 : -1) * Math.random() * 10;

         chartData.push({
           date: newDate.getTime(),
           value: value
         });
       }
       return chartData;
     }

     let data = generateChartData();


// Create chart
// https://www.amcharts.com/docs/v5/charts/xy-chart/
     let chart = root.container.children.push(am5xy.XYChart.new(root, {
       focusable: true,
       panX: true,
       panY: true,
       wheelX: "panX",
       wheelY: "zoomX",
       pinchZoomX:true
     }));

     let easing = am5.ease.linear;


// Create axes
// https://www.amcharts.com/docs/v5/charts/xy-chart/axes/
     let xAxis = chart.xAxes.push(am5xy.DateAxis.new(root, {
       maxDeviation: 0.5,
       groupData: false,
       extraMax:0.1, // this adds some space in front
       extraMin:-0.1,  // this removes some space form th beginning so that the line would not be cut off
       baseInterval: {
         timeUnit: "second",
         count: 1
       },
       renderer: am5xy.AxisRendererX.new(root, {
         minGridDistance: 50
       }),
       tooltip: am5.Tooltip.new(root, {})
     }));

     let yAxis = chart.yAxes.push(am5xy.ValueAxis.new(root, {
       renderer: am5xy.AxisRendererY.new(root, {})
     }));


// Add series
// https://www.amcharts.com/docs/v5/charts/xy-chart/series/
     let series = chart.series.push(am5xy.LineSeries.new(root, {
       name: "Series 1",
       xAxis: xAxis,
       yAxis: yAxis,
       valueYField: "value",
       valueXField: "date",
       tooltip: am5.Tooltip.new(root, {
         pointerOrientation: "horizontal",
         labelText: "{valueY}"
       })
     }));

// tell that the last data item must create bullet
     data[data.length - 1].bullet = true;
     series.data.setAll(data);


// Create animating bullet by adding two circles in a bullet container and
// animating radius and opacity of one of them.
     series.bullets.push(function(root, series, dataItem) {
       // only create sprite if bullet == true in data context
       if (dataItem.dataContext.bullet) {
         let container = am5.Container.new(root, {});
         // let circle0 = container.children.push(am5.Circle.new(root, {
         //   radius: 5,
         //   fill: am5.color(0xff0000)
         // }));
         let circle1 = container.children.push(am5.Circle.new(root, {
           radius: 5,
           fill: am5.color(0xff0000)
         }));

         circle1.animate({
           key: "radius",
           to: 20,
           duration: 1000,
           easing: am5.ease.out(am5.ease.cubic),
           loops: Infinity
         });
         circle1.animate({
           key: "opacity",
           to: 0,
           from: 1,
           duration: 1000,
           easing: am5.ease.out(am5.ease.cubic),
           loops: Infinity
         });

         return am5.Bullet.new(root, {
           locationX:undefined,
           sprite: container
         })
       }
     })


// Add cursor
// https://www.amcharts.com/docs/v5/charts/xy-chart/cursor/
     let cursor = chart.set("cursor", am5xy.XYCursor.new(root, {
       xAxis: xAxis
     }));
     cursor.lineY.set("visible", false);


// Update data every second
     setInterval(function () {
       addData();
     }, 1000)


     function addData() {
       let lastDataItem = series.dataItems[series.dataItems.length - 1];

       let lastValue = lastDataItem.get("valueY");
       let newValue = Math.round((Math.random() * 10 - 5) + value);
       let lastDate = new Date(lastDataItem.get("valueX"));
       let time = am5.time.add(new Date(lastDate), "day", 1).getTime();
       series.data.removeIndex(0);
       series.data.push({
         date: time,
         value: newValue
       })

       let newDataItem = series.dataItems[series.dataItems.length - 1];
       newDataItem.animate({
         key: "valueYWorking",
         to: newValue,
         from: lastValue,
         duration: 600,
         easing: easing
       });

       // use the bullet of last data item so that a new sprite is not created
       newDataItem.bullets = [];
       newDataItem.bullets[0] = lastDataItem.bullets[0];
       newDataItem.bullets[0].get("sprite").dataItem = newDataItem;
       // reset bullets
       lastDataItem.dataContext.bullet = false;
       lastDataItem.bullets = [];


       let animation = newDataItem.animate({
         key: "locationX",
         to: 0.5,
         from: -0.5,
         duration: 600
       });
       if (animation) {
         let tooltip = xAxis.get("tooltip");
         if (tooltip && !tooltip.isHidden()) {
           animation.events.on("stopped", function () {
             xAxis.updateTooltip();
           })
         }
       }
     }


// Make stuff animate on load
// https://www.amcharts.com/docs/v5/concepts/animations/
     chart.appear(1000, 100);
   }
  },
  mounted() {
    this.displayVChart();
  },

  beforeDestroy() {
  }
}
</script>

<style scoped>
.hello {
  height: 500px;
  width: 100%;
}
</style>