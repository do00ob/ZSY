<template>
  <div>
    <div id="main"></div>
    <div class="averageRating"></div>
    <el-input v-model="storeUrl" placeholder="请输入店铺网址"></el-input>
    <el-button type="primary" @click="pa">捕获</el-button>
    <el-checkbox v-model="checked">备选项</el-checkbox>
  </div>

</template>

<script>
import axios from 'axios';
import * as echarts from 'echarts';
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      storeUrl: "",
      checked: true,
      storeData: [],
      rootStoreData: []
    }
  },
  methods: {
    processString(str) {
      if (str.includes('K sold')) {
        const numberPart = str.split('K sold')[0];
        return parseFloat(numberPart) * 1000;
      } else {
        return parseInt(str.split(' sold')[0]);
      }
    },
    transposeArray(arr) {
      const rows = arr.length;
      const cols = arr[0].length;
      const newArray = new Array(cols).fill().map(() => new Array(rows));
      for (let i = 0; i < rows; i++) {
        for (let j = 0; j < cols; j++) {
          newArray[j][i] = arr[i][j];
        }
      }
      return newArray;
    },
    /*该方法可以把多个数组组合成一个对象数据，并且可以对对象内的属性进行命名 */
    combineArraysToObjectArray(arrays,...propertyNames) {
      const result = [];
      const length = arrays[0].length;
      for (let i = 0; i < length; i++) {
        const obj = {};
        for (let j = 0; j < arrays.length; j++) {
          obj[propertyNames[j]] = arrays[j][i];
        }
        result.push(obj);
      }
      return result;
    },
    pa() {
      axios.post('http://127.0.0.1:5000/api/pa', { url: this.storeUrl })
        .then(response => {
          console.log(response.data);
          this.rootStoreData =this.transposeArray(response.data.message);/*行列调换 */
          console.log(this.rootStoreData)
          response.data.message.forEach(item => {
            this.storeData = [...this.storeData, { name: item[0], value: item[7] }];
          });
          console.log(this.storeData);

          var myChart = echarts.init(document.getElementById('main'));
          var option = {
            tooltip: {
              trigger: 'item'
            },
            legend: {
              top: '5%',
              left: 'center'
            },
            series: [
              {
                name: 'Access From',
                type: 'pie',
                radius: ['40%', '70%'],
                avoidLabelOverlap: false,
                itemStyle: {
                  borderRadius: 10,
                  borderColor: '#fff',
                  borderWidth: 2
                },
                label: {
                  show: false,
                  position: 'center'
                },
                emphasis: {
                  label: {
                    show: true,
                    fontSize: 40,
                    fontWeight: 'bold'
                  }
                },
                labelLine: {
                  show: false
                },
                data: this.storeData
              }
            ]
          };
          // 绘制图表
          myChart.setOption(option);
          /*上面是第一个图*/
          var mychart_2 = echarts.init(document.getElementById('averageRating'));
          var optinon_2 = {
            tooltip: {
              trigger: 'axis',
              axisPointer: {
                // Use axis to trigger tooltip
                type: 'shadow' // 'shadow' as default; can also be 'line' or 'shadow'
              }
            },
            legend: {},
            grid: {
              left: '3%',
              right: '4%',
              bottom: '3%',
              containLabel: true
            },
            xAxis: {
              type: 'value'
            },
            yAxis: {
              type: 'category',
              data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
            },
            series: [
              {
                name: 'Direct',
                type: 'bar',
                stack: 'total',
                label: {
                  show: true
                },
                emphasis: {
                  focus: 'series'
                },
                data: [320, 302, 301, 334, 390, 330, 320]
              },
              {
                name: 'Mail Ad',
                type: 'bar',
                stack: 'total',
                label: {
                  show: true
                },
                emphasis: {
                  focus: 'series'
                },
                data: [120, 132, 101, 134, 90, 230, 210]
              },
              {
                name: 'Affiliate Ad',
                type: 'bar',
                stack: 'total',
                label: {
                  show: true
                },
                emphasis: {
                  focus: 'series'
                },
                data: [220, 182, 191, 234, 290, 330, 310]
              },
              {
                name: 'Video Ad',
                type: 'bar',
                stack: 'total',
                label: {
                  show: true
                },
                emphasis: {
                  focus: 'series'
                },
                data: [150, 212, 201, 154, 190, 330, 410]
              },
              {
                name: 'Search Engine',
                type: 'bar',
                stack: 'total',
                label: {
                  show: true
                },
                emphasis: {
                  focus: 'series'
                },
                data: [820, 832, 901, 934, 1290, 1330, 1320]
              }
            ]
          };
          mychart_2.setOption(optinon_2)
        })
        .catch(error => {
          console.error(error);
        });
    }
  },
  mounted() {
  },

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

div {
  width: 400px;
  height: 400px;
}

.averageRating {
  width: 400px;
  height: 400px;
}
</style>
