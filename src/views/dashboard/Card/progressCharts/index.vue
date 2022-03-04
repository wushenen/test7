<template>
  <div class="charts" ref="charts"></div>
</template>

<script>
//引入echarts
import * as echarts from "echarts";
export default {
  name: "progressCharts",
  mounted() {
    //初始化echarts实例
    let progressCharts = echarts.init(this.$refs.charts);
    //配置数据
    progressCharts.setOption({
      xAxis: {
        //隐藏X轴
        show: false,
        //最小值与最大值的设置
        min: 0,
        max: 100,
      },
      yAxis: {
        //隐藏Y轴
        show: false,
        type: "category",
      },
      //系列
      series: [
        {
          type: "bar",
          data: [78],
          color: "yellowgreen",
          //柱状图的宽度
          barWidth: 20,
          //背景颜色
          showBackground: true,
          backgroundStyle: {
            color: "#eee",
          },
          //文本
          label: {
            show: true,
            formatter:'|',
            position:'right'
          },
        },
      ],
      grid: {
        left: 0,
        top: 0,
        right: 0,
        bottom: 0,
      },
      tooltip: {},
    });
  },
};
</script>

<style>
.charts {
  width: 100%;
  height: 100%;
}
</style>