<template>
  <div class="charts" ref="charts"></div>
</template>

<script>
//引入echarts
import * as echarts from 'echarts'
export default {
name:'lineCharts',
mounted(){
    //初始化echarts实例
    let lineCharts=echarts.init(this.$refs.charts)
    //配置数据
    lineCharts.setOption({
        xAxis:{
            //隐藏X轴
            show:false,
            type:"category"
        },
        yAxis:{
             //隐藏Y轴
            show:false
        },
        //系列
        series:[
            {
               type:"line",
               data:[47,29,33,20,48,19,29,70,44,36,40,28,19,45,62,39,36,42,45,25,45,20,40,60,35,32,40,60,50,52,42,60,40,33] ,
               //拐点的样式设计
               itemStyle:{
                   opacity:0
               },
               //线条的样式
               lineStyle:{
                   color:'purple'
               },
               //填充颜色的设置
               areaStyle:{
                   color:{
                       type:"linear",
                       x:0,
                       y:0,
                       x2:0,
                       y2:1,
                       colorStops:[
                           {
                               offset:0,
                               color:"purple",
                           },
                           {
                               offset:1,
                               color:"#fff",
                           },
                       ],
                       global:false,
                   }
               },
            }
        ],
        grid:{
            left:0,
            top:0,
            right:0,
            bottom:0
        }
    })
}
}
</script>

<style>
.charts{
    width: 100%;
    height: 100%;
}
</style>