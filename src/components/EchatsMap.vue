<template>
  <div id="mychart" ref="myChart"></div>
</template>

<script>
import * as echarts from 'echarts';

import chinaJSON from "/public/mapJSON/中华人民共和国.json"
export default {
  data() {
    return {
      myChart: null,
      option: {
        title: {
        },
        // tooltip: {
        //   trigger: 'item',
        //   showDelay: 0,
        //   transitionDuration: 0.2
        // },
        visualMap: {
          left: 'right',
          show: false,
          min: 500000,
          max: 38000000,
          inRange: {
            color: [
              '#10c8a4',
              // '#313695',
              // '#4575b4',
              // '#74add1',
              // '#abd9e9',
              // '#e0f3f8',
              // '#ffffbf',
              // '#fee090',
              // '#fdae61',
              // '#f46d43',
              // '#d73027',
              // '#a50026'
            ]
          },
          text: ['High', 'Low'],
          calculable: true
        },
        // toolbox: {
        //   show: true,
        //   //orient: 'vertical',
        //   left: 'left',
        //   top: 'top',
        //   feature: {
        //     dataView: { readOnly: false },
        //     restore: {},
        //     saveAsImage: {}
        //   }
        // },
        series: [
          {
            name: 'China',
            type: 'map',
            roam: true,
            map: 'China',
            // 默认样式
            itemStyle: {
              areaColor: "rgba(255,255,255, .3)",
              borderWidth: 0
            },
            // 高亮状态样式
            emphasis: {
              label: {
                show: false,
              },
              itemStyle: {
                areaColor: "#10c8a4",
              }
            },
            // 选中状态样式
            select: {
              label: {
                show: false,
              },
              itemStyle: {
                areaColor: "#10c8a4",
              }
            },
            selectedMode: false,
            selected: { "青海省": true },
            nameMap: {
            },
            data: [
              { name: "内蒙古自治区", value: "3000" }
            ]
          }
        ]
      }
    }
  },
  mounted() {
    this.myChart = echarts.init(this.$refs.myChart);

    this.myChart.showLoading();

    this.myChart.hideLoading();
    echarts.registerMap('China', chinaJSON, {});
    this.myChart.setOption(this.option);


    this.option && this.myChart.setOption(this.option);
    this.myChart.on('click', (e) => {
      console.log("echarts click", e);
      this.option.series[0].data = [{ name: e.name, value: 0 }];
      this.myChart.setOption(this.option);
      this.$bus.$emit("echartsMapClick", e);
    });
  }
}
</script>

<style>
#mychart {
  position: absolute;
  bottom: 20px;
  left: 20px;
  width: 400px;
  height: 400px;
  background-color: #16324f88;
}
</style>