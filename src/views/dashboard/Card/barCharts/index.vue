<template>
  <div class="charts" ref="charts"></div>
</template>

<script>
//引入echarts
import * as echarts from 'echarts'
export default {
name:'barCharts',
mounted(){
    //初始化echarts实例
    let barCharts=echarts.init(this.$refs.charts)
    //配置数据
    barCharts.setOption({
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
               type:"bar",
               data:[47,29,33,20,48,19,29,70,44,36,40,28] ,
               color:'cyan',
               //拐点的样式设计
             }],
             grid:{
            left:0,
            top:0,
            right:0,
            bottom:0
        },
        tooltip:{}
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