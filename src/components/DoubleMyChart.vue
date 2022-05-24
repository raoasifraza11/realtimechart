<template>
  <div>
    <h1>{{title}}</h1>
    <div class="hello" id="dmchartdiv"></div>
  </div>
</template>

<script>


import * as am5 from '@amcharts/amcharts5';
import * as am5xy from '@amcharts/amcharts5/xy';
import am5themes_Animated from '@amcharts/amcharts5/themes/Animated';


export default {
  name: 'DoubleMyChart',
  props: {
    title: String,
  },
  data() {
    return {
      myTitle: 'Hello',
    }
  },

  methods: {

    displayChart(){
      let root = am5.Root.new("dmchartdiv");


// Set themes
// https://www.amcharts.com/docs/v5/concepts/themes/
      root.setThemes([
        am5themes_Animated.new(root)
      ]);


// Create chart
// https://www.amcharts.com/docs/v5/charts/xy-chart/
      let chart = root.container.children.push(am5xy.XYChart.new(root, {
        panX: true,
        panY: true,
        wheelX: "panX",
        wheelY: "zoomX",
        pinchZoomX:true,
        focusable: true,
      }));

      let easing = am5.ease.linear;


// Add cursor
// https://www.amcharts.com/docs/v5/charts/xy-chart/cursor/
      let cursor = chart.set("cursor", am5xy.XYCursor.new(root, {}));
      //cursor.lineX.set("forceHidden", true);
      cursor.lineY.set("visible", false);

// Generate random data
      let date = new Date();
      date.setHours(0, 0, 0, 0);
      let value = 100;

      function generateData() {
        value = Math.round((Math.random() * 10 - 5) + value);
        am5.time.add(date, "day", 1);
        return {
          date: date.getTime(),
          value: value
        };
      }


      function generateDatas(count) {
        let data = [];
        for (var i = 0; i < count; ++i) {
          data.push(generateData());
        }
        return data;
      }



// Create axes
// https://www.amcharts.com/docs/v5/charts/xy-chart/axes/
      let xAxis = chart.xAxes.push(am5xy.DateAxis.new(root, {
        maxDeviation: 0.5,
        groupData: false,
        extraMax: 0.1,
        extraMin:-0.1,
        baseInterval: {
          timeUnit: "day",
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
        name: "Series",
        xAxis: xAxis,
        yAxis: yAxis,
        valueYField: "value",
        valueXField: "date",
        tooltip: am5.Tooltip.new(root, {
          pointerOrientation: "horizontal",
          labelText: "{valueY}"
        })
      }));

      series.fills.template.setAll({
        fillOpacity: 0.2,
        visible: true
      });


// Add scrollbar // todo: hide above scroll bar
// https://www.amcharts.com/docs/v5/charts/xy-chart/scrollbars/
//       chart.set("scrollbarX", am5.Scrollbar.new(root, {
//         orientation: "horizontal"
//       }));


// Set data
      let data = generateDatas(1200);
      // set bullet in last
      data[data.length - 1].bullet = true;
      series.data.setAll(data);

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


      function addData() {
        let lastDataItem = series.dataItems[series.dataItems.length - 1];

        let lastValue = lastDataItem.get("valueY");
        let newValue = Math.round((Math.random() * 10 - 5) + value);
        let lastDate = new Date(lastDataItem.get("valueX"));
        // five days increment for testing
        let time = am5.time.add(new Date(lastDate), "day", 5).getTime();
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

      setInterval(function () {
        addData();
      }, 1000)




      let rangeDate = new Date();
      am5.time.add(rangeDate, "day", Math.round(series.dataItems.length / 1.5));
      let rangeTime = rangeDate.getTime();

// add series range
      let seriesRangeDataItem = xAxis.makeDataItem({});
      let seriesRange = series.createAxisRange(seriesRangeDataItem);
      seriesRange.fills.template.setAll({
        visible: true,
        opacity: 0.3
      });

      // Set the fill pattern
      seriesRange.fills.template.set("fillPattern", am5.CirclePattern.new(root, {
        color: am5.color(0xff0000),
        rotation: 45,
        strokeWidth: 2,
        width: 2000,
        height: 2000,
        fill:am5.color(0xffffff)
      }));

      seriesRange.strokes.template.set("stroke", am5.color(0xff0000));

      xAxis.onPrivate("max", function (value) {
        seriesRangeDataItem.set("endValue", value);
        seriesRangeDataItem.set("value", rangeTime);
      });

// add axis range
      let range = xAxis.createAxisRange(xAxis.makeDataItem({}));
      let color = root.interfaceColors.get("primaryButton");

      range.set("value", rangeDate.getTime());
      range.get("grid").setAll({
        strokeOpacity: 1,
        stroke: color
      });

      let resizeButton = am5.Button.new(root, {
        themeTags: ["resize", "horizontal"],
        icon: am5.Graphics.new(root, {
          themeTags: ["icon"]
        })
      });

// restrict from being dragged vertically
      resizeButton.adapters.add("y", function () {
        return 0;
      });

// restrict from being dragged outside of plot
      resizeButton.adapters.add("x", function (x) {
        return Math.max(0, Math.min(chart.plotContainer.width(), x));
      });

// change range when x changes
      resizeButton.events.on("dragged", function () {
        let x = resizeButton.x();
        let position = xAxis.toAxisPosition(x / chart.plotContainer.width());

        let value = xAxis.positionToValue(position);

        range.set("value", value);

        seriesRangeDataItem.set("value", value);
        seriesRangeDataItem.set("endValue", xAxis.getPrivate("max"));
      });

// set bullet for the range // todo: stop the vertical scroll of middle range selector
//       range.set("bullet", am5xy.AxisBullet.new(root, {
//         sprite: resizeButton
//       }));


// Make stuff animate on load
// https://www.amcharts.com/docs/v5/concepts/animations/
      //series.appear(1000);
      chart.appear(1000, 100);
    }

  },
  mounted() {
    this.displayChart();
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