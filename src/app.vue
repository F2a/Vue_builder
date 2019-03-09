<template>
  <div id="app">
    <div class="chartBox">
      <v-chart class="charts" :options="pie"/>
    </div>
    <div class="chartBox">
      <v-chart class="charts" :options="line"/>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import ECharts from 'vue-echarts'
import 'echarts/lib/chart/pie'
import 'echarts/lib/chart/bar'
import 'echarts/lib/chart/scatter'
import 'echarts/lib/chart/effectScatter'
import 'echarts/lib/component/tooltip'
import 'echarts/lib/component/axis'
import 'echarts/lib/component/dataZoom'

require('echarts');
require('echarts/extension/bmap/bmap');

import changshaData from './assets/data.json'

export default {
  name: 'Home',
  components: {
    'v-chart': ECharts
  },
  data() {
    const salaryType = ['面议', '1-5k', '5-8k', '8-10k', '10-13k', '13-15k', '15-20k', '20-25k', '25-30k', '30k+']
    return {
      changshaData,
      salaryType,
    };
  },
  computed: {
    average: function (){
      return this.changshaData.data.map((val, i) => {
        const re = /^[0-9]+.?[0-9]*/
        if (re.test(val[5])&&re.test(val[6])){
         return parseFloat((val[5] + val[6])/2).toFixed(1)
        }else{
          // 面议
          return 0 
        }
      })
    },
    pie: function() {
      const data = Array(10).fill(0).map((val, i) => {
              return { value: val, name: this.salaryType[i] }
            })
      this.average.map((val, i) => {
        if(val<1) data[0].value++
        if(val>=1&&val<5) data[1].value++
        if(val>=5&&val<8) data[2].value++
        if(val>=8&&val<10) data[3].value++
        if(val>=10&&val<13) data[4].value++
        if(val>=13&&val<15) data[5].value++
        if(val>=15&&val<20) data[6].value++
        if(val>=20&&val<25) data[7].value++
        if(val>=25&&val<30) data[8].value++
        if(val>=30) data[9].value++
      })
      return {   
        title : {
          text: '分布',
          subtext: '纯属虚构',
          x:'center'
        },
        tooltip : {
          trigger: 'item',
          formatter: "{b} : {c} ({d}%)"
        },
        legend: {
          orient: 'vertical',
          left: 'left',
          data: this.salaryType
        },
        series : [
          {
            name: '访问来源',
            type: 'pie',
            radius : '55%',
            center: ['50%', '60%'],
            data: data,
            itemStyle: {
              emphasis: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }
        ]
      }
    },
    line: function () {
      const sortedData = this.changshaData.data.sort(this.sortData(true));
      const lineData = this.average.sort(this.sortData(false));
      return {
        color: '#61a0a8',        
        tooltip : {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow',
            label: {
              show: true
            }
          },
          formatter: function (params){
            const i = params[0].dataIndex
            let result =`
              ${sortedData[i][0]} <br />
              ${sortedData[i][1]} <br />
              ${sortedData[i][4]} <br />
            `
            if(sortedData[i][5]){
              result += `${sortedData[i][5]} - ${sortedData[i][6]}K`
            }else{
              result += '面议'
            }
            return result
          }
        },
        toolbox: {
          show : true,
          feature : {
            mark : {show: true},
            dataView : {show: true, readOnly: false},
            magicType: {show: true, type: ['line', 'bar']},
            restore : {show: true},
            saveAsImage : {show: true}
          }
        },
        calculable : true,
        legend: {
          itemGap: 5
        },
        grid: {
          top: '12%',
          left: '1%',
          right: '10%',
          containLabel: true
        },
        xAxis: [
          {
            type : 'category',
            data : sortedData.map((val, i) => {
              return val[0]
            })
          }
        ],
        yAxis: [
          {
            type : 'value',
            name : '薪资',
            axisLabel: {

            }
          }
        ],
        dataZoom: [
          {
            show: true,
            start: 94,
            end: 100
          },
          {
            type: 'inside',
            start: 94,
            end: 100
          },
        ],
        series : [
          {
            name: '柱状图排序',
            type: 'bar',
            data: lineData
          },
        ]
      }
    },
    scatter: function() {
      let data = [
        {name: '海门', value: 9},
        {name: '鄂尔多斯', value: 12},
        {name: '招远', value: 12},
      ]
      let geoCoordMap = {
        '海门':[121.15,31.89],
        '鄂尔多斯':[109.781327,39.608266],
        '招远':[120.38,37.35],
      }
      let convertData = function (data) {
        var res = [];
        for (var i = 0; i < data.length; i++) {
            var geoCoord = geoCoordMap[data[i].name];
            if (geoCoord) {
                res.push({
                    name: data[i].name,
                    value: geoCoord.concat(data[i].value)
                });
            }
        }
        return res;
      };
      return {
        title: {
          text: '薪资地域分布图',
          subtext: 'data from zhaopin.com',
          left: 'center'
        },
        tooltip : {
          trigger: 'item'
        },
    bmap: {
        // 百度地图中心经纬度
        center: [120.13066322374, 30.240018034923],
        // 百度地图缩放
        zoom: 14,
        // 是否开启拖拽缩放，可以只设置 'scale' 或者 'move'
        roam: true,
        // 百度地图的自定义样式，见 http://developer.baidu.com/map/jsdevelop-11.htm
        mapStyle: {}
    },
    series: [{
        type: 'scatter',
        // 使用百度地图坐标系
        coordinateSystem: 'bmap',
        // 数据格式跟在 geo 坐标系上一样，每一项都是 [经度，纬度，数值大小，其它维度...]
        data: [ [120, 30, 1] ]
    }]
      }
    }
  },
  methods: {
    sortData: function(isObj) {
      return function(A, B) {
        if(isObj){
          let averageA = 0, averageB = 0;
          const re = /^[0-9]+.?[0-9]*/
          if (re.test(A[5])&&re.test(A[6])) averageA = parseFloat((A[5] + A[6])/2).toFixed(1)
          if (re.test(B[5])&&re.test(B[6])) averageB = parseFloat((B[5] + B[6])/2).toFixed(1)
          return averageA - averageB
        }
        return A - B
      }
    }
  },
  beforeCreate() {
    console.log('在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用');
  },
  created() {
    console.log('在实例创建完成后被立即调用');
  },
  beforeMount() {
    console.log('在挂载开始之前被调用');
  },
  mounted() {
    console.log('挂载完成');
  },
  beforeUpdate() {
    console.log('数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前');
  },
  updated() {
    console.log('发生在虚拟 DOM 重新渲染和打补丁之后');
    // 当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。然而在大多数情况下，你应该避免在此期间更改状态。
    this.$nextTick(function () {
      // 所有的子组件也都一起被挂载可以用 vm.$nextTick
    })
  },
  activated() {
    console.log('keep-alive 组件激活时调用');
  },
  deactivated() {
    console.log('keep-alive 组件停用时调用');
  },
  beforeDestroy() {
    console.log('Vue 实例销毁之前调用');
  },
  destroyed() {
    console.log('Vue 实例销毁后调用');
  },
  errorCaptured(err, vm, info) {
    console.log('当捕获一个来自子孙组件的错误时被调用');
  }
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.chartBox{
  width: 100%;
}
.charts{
  width: 100%;
}
</style>
