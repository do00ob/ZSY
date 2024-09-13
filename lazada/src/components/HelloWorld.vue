<template>
  <div>
    <!--<el-backtop target=".page-component__scroll .el-scrollbar__wrap"></el-backtop>-->
    <div id="main"></div>
    <div id="averageRating"></div>
    <el-input placeholder="请输入店铺名称" v-model="storeUrl" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="pc"></el-button>
    </el-input>
    <!--<el-input v-model="storeUrl" placeholder="请输入店铺网址" maxlength="5"></el-input>
    <el-row :gutter="20">
      <el-col :span="6" :offset="18">
        <el-button type="primary" @click="pa">搜索</el-button>
        <el-button type="primary" @click="pb">搜索</el-button>
      </el-col>
    </el-row>-->
    <!--<el-checkbox v-model="checked">备选项</el-checkbox> -->
    <div class="container" v-show="showCards" v-loading="loading">
      <el-button type="primary" class="buttonAnalyse" @click="analyse">数据分析<i
          class="el-icon-s-marketing"></i></el-button>
      <el-row class="main">
        <el-col :span="10" v-for="(o, index) in products" :key="o" :offset="index % 2 > 0 ? 2 : 0">
          <el-card shadow="hover" :body-style="{ padding: '0px' }" class="cardMode">
            <img :src="o.imgUrl" class="image">
            <div style="padding: 14px;">
              <div class="nameFence">{{ o.name }}</div>
              <div class="bottom clearfix">
                <time class="time">{{ o.price }}</time>
                <el-button type="primary" class="button" icon="el-icon-search"
                  @click="() => paiLiTao(o.imgUrl)">搜同款</el-button>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>

    <el-dialog title="数据分析" :visible.sync="dialogVisible" width="80%" :before-close="handleClose">
      <div class="graphContainer">
        <div id="showgraph1" class="showgraph"></div>
        <div id="showgraph2" class="showgraph"></div>
        <div id="showgraph3" class="showgraph"></div>
        <div id="showgraph4" class="showgraph"></div>

        <div id="showgraph5" class="showgraph_next"></div>
      </div>
      
      <span slot="footer" class="dialog-footer" style="position: relative;">
        <el-button type="primary" @click="nextPage" class="next" v-show="isNext">下一页</el-button>
        <el-button @click="reset">取 消</el-button>
        <el-button type="primary" @click="pb">确 定</el-button>
      </span>
    </el-dialog>

    <!--<el-button type="primary" style="margin-left: 16px" @click="change">
      打开
    </el-button>-->

    <el-drawer :visible.sync="drawer" title="I am the title" :with-header="false">
      <el-row>
        <el-col :span="8" v-for="(o, index) in products" :key="o" :offset="index > 0 ? 2 : 0">
          <el-card :body-style="{ padding: '10px' }">
            <img :src="o.imgUrl" class="image">
            <div style="padding: 14px;">
              <span>{{ o.name }}</span>
              <div class="bottom clearfix">
                <time class="time">{{ o.price }}</time>
                <el-button type="primary" class="button" icon="el-icon-search"
                  @click="() => paiLiTao(o.imgUrl)">搜同款</el-button>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </el-drawer>

  </div>

</template>


<script>
import axios from 'axios';
import * as echarts from 'echarts';
import 'echarts-wordcloud';
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
      rootStoreData: [],
      currentDate: new Date(),
      products: [],
      findSameProducts: [],
      drawer: false,
      showCards: false,
      loading:false,
      dialogVisible: false,
      isNext : false,
      keyWords:[]
    }
  },
  methods: {
    
    change() {
      
      
      console.log(this.drawer);

    },
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
    combineArraysToObjectArray(arrays, ...propertyNames) {
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
    combineArraysToObjectArray2(selName, arrays, ...propertyNames) {
      const result = [];
      const length = arrays[0].length;
      //console.log(selName)
      for (let i = 0; i < length; i++) {
        const obj = {};
        if (this.rootStoreData[9][i].indexOf(String(selName)) !== -1) {
          for (let j = 0; j < arrays.length; j++) {
            obj[propertyNames[j]] = arrays[j][i]
          }
          result.push(obj);
        }
      }
      return result;
    },
    /*可以将文字里面的数字提取出来，销量的计算很需要这个 */
    convertArrayToNumbers(arr) {
      return arr.map(item => {
        if (typeof item === 'string' && item.includes('K')) {
          const numberPart = parseFloat(item.split('K')[0]);
          return numberPart * 1000;
        } else {
          return parseInt(item);
        }
      });
    },
    pc(){
      var updateOrAddObject = function(array, inputName) {
      // 查找是否存在具有相同 name 的对象
      const obj = array.find(item => item.name === inputName);

      if (obj) {
        // 如果对象已存在，增加其 value
        obj.value += 1;
      } else {
        // 如果对象不存在，新增一个对象，value 为 1
        array.push({ name: inputName, value: 1 });
      }
    }
      if(this.storeUrl == ''){
        this.$message({
          showClose: true,
          message: '请输入关键字',
          type: 'warning'
        });
        return;
      }
      updateOrAddObject(this.keyWords,this.storeUrl)
      this.showCards = true
      this.products = this.combineArraysToObjectArray2(this.storeUrl, [this.rootStoreData[2], this.rootStoreData[0], this.rootStoreData[5],this.rootStoreData[6],this.rootStoreData[7],this.rootStoreData[12]], "imgUrl", "name", "price", "score","comments","sales")
      this.$message({
          showClose: true,
          message: '搜索出'+this.products.length+'条相关条目',
          type: 'success'
        });
      //console.log(this.products)
    },
    pb() {
      this.products=[];
      var ObjectArrayTocombineArrays = function (attribute, initialArray) {
        var array = [];
        for (let i = 0; i < initialArray.length; i++) {
          var motoNum = initialArray[i][attribute].replace(/[a-zA-Z]/g, '')
          array.push(Math.round(motoNum*100)/100)
        }
        return array
      }
      //console.log(response.data);
      this.isNext = true
      this.rootStoreData[12] = this.convertArrayToNumbers(this.rootStoreData[12])
      //console.log("销量", this.rootStoreData[12]);
      var ar_1 = ObjectArrayTocombineArrays("comments", this.products)
      var ar_2 = ObjectArrayTocombineArrays("sales", this.products)
      var ar_3 = ObjectArrayTocombineArrays("price", this.products)
      var ar_4 = ObjectArrayTocombineArrays("score", this.products)
      //console.log(this.products)
      //console.log(ar_1)
      /*this.rootStoreData = this.transposeArray(response.data.message);/*行列调换 */
      //console.log(this.rootStoreData)
      //var myChart = echarts.init(document.getElementById('showgraph'));
      // eslint-disable-next-line
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
            data: this.combineArraysToObjectArray([this.rootStoreData[0], this.rootStoreData[7]], "name", "value")
          }
        ]
      };
    // 绘制图表
    //myChart.setOption(option);
    /*上面是第一个图*/
    //var mychart_2 = echarts.init(document.getElementById('showgraph'));
          //-----------------------------------------------
          var chart_1 = echarts.init(document.getElementById('showgraph1'));
          var chart_2 = echarts.init(document.getElementById('showgraph2'));
          var chart_3 = echarts.init(document.getElementById('showgraph3'));
          var chart_4 = echarts.init(document.getElementById('showgraph4'));
          //-----------------------------------------------
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
              axisLabel: {
                formatter: function (value) {
                  // 根据实际情况对标签进行处理，比如只显示前几个字符等
                  return value.substring(0, 5) + '...';
                }
              },
              type: 'category',
              data: this.rootStoreData[0]
            },
            series: [
              {
                name: '评论数',
                type: 'bar',
                stack: 'total',
                itemStyle: {
                  color: '#dede01'
                },
                label: {
                  show: true,
                  precision: 1,
                  position: 'right',
                  valueAnimation: true,
                  fontFamily: 'monospace'
                },
                emphasis: {
                  focus: 'series'
                },
                data: ar_1
              },
              /*{
                name: '销量',
                type: 'bar',
                stack: 'total',
                label: {
                  show: true
                },
                emphasis: {
                  focus: 'series'
                },
                data: ar_1
              },*/
            ]

          };

          var optinon_3 = {
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
              axisLabel: {
                formatter: function (value) {
                  // 根据实际情况对标签进行处理，比如只显示前几个字符等
                  return value.substring(0, 5) + '...';
                }
              },
              type: 'category',
              data: this.rootStoreData[0]
            },
            series: [
              {
                name: '销量',
                type: 'bar',
                stack: 'total',
                itemStyle: {
                  color: '#ee1122'
                },
                label: {
                  show: true,
                  precision: 1,
                  position: 'right',
                  valueAnimation: true,
                  fontFamily: 'monospace'
                },
                emphasis: {
                  focus: 'series'
                },
                data: ar_2
              }
            ],
            graphic: {
              elements: [
                {
                  type: 'text',
                  right: 20,
                  bottom: 60,
                  style: {
                    text: "单位:K",
                    font: 'bolder 80px monospace',
                    fill: 'rgba(100, 100, 100, 0.25)'
                  },
                  z: 100,
                }
              ]
            }

          };
          var optinon_4 = {
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
              axisLabel: {
                formatter: function (value) {
                  // 根据实际情况对标签进行处理，比如只显示前几个字符等
                  return value.substring(0, 5) + '...';
                }
              },
              type: 'category',
              data: this.rootStoreData[0]
            },
            series: [
              {
                name: '价格',
                type: 'bar',
                stack: 'total',
                emphasis: {
                  focus: 'series'
                },
                data: ar_3,
                itemStyle: {
                  color: '#11de21'
                },
                label: {
                  show: true,
                  precision: 1,
                  position: 'right',
                  valueAnimation: true,
                  fontFamily: 'monospace'
                }
              },
              
            ],
            graphic: {
              elements: [
                {
                  type: 'text',
                  right: 20,
                  bottom: 60,
                  style: {
                    text: "单位:RMB",
                    font: 'bolder 80px monospace',
                    fill: 'rgba(100, 100, 100, 0.25)'
                  },
                  z: 100,
                }
              ]
            }

          };
          var optinon_5 = {
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
              type: 'value',
              max:5,
              min:4.5
            },
            yAxis: {
              axisLabel: {
                formatter: function (value) {
                  // 根据实际情况对标签进行处理，比如只显示前几个字符等
                  return value.substring(0, 5) + '...';
                }
              },
              type: 'category',
              data: this.rootStoreData[0]
            },
            series: [
              {
                name: '评分',
                type: 'bar',
                stack: 'total',
                itemStyle: {
                  color: '#1212df'
                },
                label: {
                  show: true,
                  precision: 1,
                  position: 'right',
                  valueAnimation: true,
                  fontFamily: 'monospace'
                },
                emphasis: {
                  focus: 'series'
                },
                data: ar_4
              }
            ]

          };
          //mychart_2.setOption(optinon_2)
          chart_1.setOption(optinon_2)
          chart_2.setOption(optinon_3)
          chart_3.setOption(optinon_4)
          chart_4.setOption(optinon_5)
        },
    pa() {
      axios.post('http://127.0.0.1:5000/api/pa', { url: this.storeUrl })
        .then(response => {
          console.log(response.data);

          this.rootStoreData[12] = this.convertArrayToNumbers(this.rootStoreData[12])
          console.log("销量", this.rootStoreData[12]);
          /*this.rootStoreData = this.transposeArray(response.data.message);/*行列调换 */
          console.log(this.rootStoreData)
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
                data: this.combineArraysToObjectArray([this.rootStoreData[0], this.rootStoreData[7]], "name", "value")
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
              axisLabel: {
                formatter: function (value) {
                  // 根据实际情况对标签进行处理，比如只显示前几个字符等
                  return value.substring(0, 5) + '...';
                }
              },
              type: 'category',
              data: this.rootStoreData[0]
            },
            series: [
              {
                name: '评论数',
                type: 'bar',
                stack: 'total',
                label: {
                  show: true
                },
                emphasis: {
                  focus: 'series'
                },
                data: this.rootStoreData[7]
              },
              {
                name: '销量',
                type: 'bar',
                stack: 'total',
                label: {
                  show: true
                },
                emphasis: {
                  focus: 'series'
                },
                data: this.rootStoreData[12]
              },
            ]

          };
          mychart_2.setOption(optinon_2)
        })
        .catch(error => {
          console.error(error);
        });
    },
    paiLiTao(imgUrl) {
      axios.post("http://127.0.0.1:5000/api/paiLiTao", { imgurl: imgUrl }).then(response => {
        console.log(response)
        /*搜同款搜到的产品信息 */
        this.findSameProducts = response.data.message.items.item


      })
    },
    analyse(){
      this.loading = true
      this.dialogVisible = true
      //this.pb();
    },
    reset(){
      var dom_1 = document.getElementById('showgraph1');
      var dom_2 = document.getElementById('showgraph2');
      var dom_3 = document.getElementById('showgraph3');
      var dom_4 = document.getElementById('showgraph4');
      var dom_5 = document.getElementById('showgraph5');
      dom_1.style.display = 'block';
      dom_2.style.display = 'block';
      dom_3.style.display = 'block';
      dom_4.style.display = 'block';
      dom_5.style.display = 'none';
      this.dialogVisible = false;this.loading = false
    },
    nextPage(){
      this.isNext = false;
      var dom_1 = document.getElementById('showgraph1');
      var dom_2 = document.getElementById('showgraph2');
      var dom_3 = document.getElementById('showgraph3');
      var dom_4 = document.getElementById('showgraph4');

      var dom_5 = document.getElementById('showgraph5');

      var chart1 = echarts.getInstanceByDom(dom_1);
      var chart2 = echarts.getInstanceByDom(dom_2);
      var chart3 = echarts.getInstanceByDom(dom_3);
      var chart4 = echarts.getInstanceByDom(dom_4);
      chart1.dispose();
      chart2.dispose();
      chart3.dispose();
      chart4.dispose();
      //console.log(dom_1);
      dom_1.style.display = 'none';
      dom_2.style.display = 'none';
      dom_3.style.display = 'none';
      dom_4.style.display = 'none';
      console.log(dom_5)
      console.log(1)
      dom_5.style.display = 'block';

      var option = {
          backgroundColor: '#fff',
          series: [
            {
              type: 'wordCloud',
              sizeRange: [15, 70], // 用来调整字的大小范围
              rotationRange: [0, 0],// 每个词旋转的角度范围和旋转的步进
              rotationStep: 45,
              gridSize: 10, // 用来调整词之间的距离
              shape: 'pentagon',// shape这个属性虽然可配置，但是在词的数量不太多的时候，效果不明显，它会趋向于画一个椭圆
              //位置的配置
              width: '100%',
              height: '100%',
              drawOutOfBound: false,// 允许词太大的时候，超出画布的范围
              layoutAnimation: true,// 布局的时候是否有动画
              textStyle: {
                normal: {
                  // 颜色可以用一个函数来返回字符串，这里是随机色
                  color: function () {
                                return 'rgb(' + [
                                    Math.round(Math.random() * 160),
                                    Math.round(Math.random() * 160),
                                    Math.round(Math.random() * 160)
                                ].join(',') + ')';
                            },
                  fontFamily: 'sans-serif',
                  fontWeight: '550'
                }
              },
              //data格式是一个数组
              data: this.keyWords
            }
          ]

        };

      var chart_5 = echarts.init(dom_5)
      chart_5.setOption(option)
    },
    handleClose(done) {
        this.$confirm('确认关闭？')
        // eslint-disable-next-line
          .then(_ => {
            this.reset()
            done();
          })
          // eslint-disable-next-line
          .catch(_ => {});
      }
  },
  mounted() {
    /*为了不反复请求服务器，这里弄一个假数据来测试 */
    let message = [
      [
        "JOBBIE Chunky Classic Peanut Butter (380g)",
        "JOBBIE Chunky Pure Peanut Butter (380g) - No Added Sugar & Salt",
        "JOBBIE Peanut Butter Stirrer",
        "JOBBIE Creamy Classic Peanut Butter (380g)",
        "JOBBIE Love Spreader",
        "JOBBIE Secret To Inner Peace (Wooden Spoon + Wooden Fork + White Pouch)",
        "JOBBIE Key to Happiness",
        "JOBBIE x Famous Amos Cookie Crumbs Peanut Spread 380g",
        "JOBBIE Path To Paradise",
        "JOBBIE Peanut Butter - Classic Gift Set",
        "JOBBIE Peanut Butter - Pure Jr Gift Set",
        "JOBBIE Popping Candy Peanut Spread 380g",
        "JOBBIE Peanut Butter Twin Jar Series + 1x JOBBIE Key To Happiness Wooden Peanut Spoon",
        "JOBBIE Peanut Butter - Classic Jr Gift Set",
        "JOBBIE Peanut Butter - Pure Gift Set",
        "JOBBIE Creamy Almond Choco Spread (200g)",
        "JOBBIE Pure Almond Butter Spread 200g - No Added Sugar & Salt",
        "JOBBIE Peanut Butter - Starter Kit",
        "JOBBIE Creamy Pure Peanut Butter (380g)"
      ],
      [
        "213113014",
        "251404265",
        "2114998534",
        "213110826",
        "2114960965",
        "4146743783",
        "2115057537",
        "3657280759",
        "4146740885",
        "2115142312",
        "2115077737",
        "4043983167",
        "4006744693",
        "2115081659",
        "2115101606",
        "4227602125",
        "3787701442",
        "407069604",
        "251398638"
      ],
      [
        "https://my-live-01.slatic.net/p/90e137547657f3d0e4675041dc78f1b5.jpg",
        "https://my-live-01.slatic.net/p/b19cdc988d81da2ef044dd9743da999e.jpg",
        "https://my-live-01.slatic.net/p/8c0760ab6566ffe3f8f844f3ae564bf8.png",
        "https://my-live-01.slatic.net/p/37b2e864c355f38befdd108abe7894c5.jpg",
        "https://my-live-01.slatic.net/p/29e0b883c08f2b635cf7058ccbafd1ba.png",
        "https://my-live-01.slatic.net/p/ee12d94d7b979f5526b5f60519e47367.jpg",
        "https://my-live-01.slatic.net/p/38d9f8864e1691457757605e286cbb20.png",
        "https://my-live-01.slatic.net/p/4e8e466c10f68acf9b7f43c5ae3cb6b2.jpg",
        "https://my-live-01.slatic.net/p/d65519f4ddcfa0ce02245ee9f696f5ef.jpg",
        "https://my-live-01.slatic.net/p/61071e5de5bbc84c98dc554038cb0f47.png",
        "https://my-live-01.slatic.net/p/25150b1009dc4f7c9907126f7114f085.png",
        "https://my-live-01.slatic.net/p/7e4bb529e4ca9aa60c526f3063022a2f.jpg",
        "https://my-live-01.slatic.net/p/fc6e87900f7ce2d882332ac3d3bef1ff.jpg",
        "https://my-live-01.slatic.net/p/8aa5a463f3d20cded595cc3c04c9e39d.png",
        "https://my-live-01.slatic.net/p/c23556cddfe89478a9e1ba3b3e211d09.png",
        "https://my-live-01.slatic.net/p/8784e57000d5b3a989d9935505ce886b.jpg",
        "https://my-live-01.slatic.net/p/1057ea856415ab65cc9ef5ac5f8a7f9c.jpg",
        "https://my-live-01.slatic.net/p/3ac9d70721c70af538d724669897e628.png",
        "https://my-live-01.slatic.net/p/64ff0657fbc69124fedf6ae3cc4c2569.jpg"
      ],
      [
        "RM25.30",
        "RM27.50",
        "",
        "RM25.30",
        "",
        "",
        "",
        "RM35.90",
        "",
        "RM129.50",
        "RM63.40",
        "RM35.90",
        "RM56.50",
        "RM59.80",
        "RM138.30",
        "",
        "",
        "RM133.90",
        "RM27.50"
      ],
      [
        "<enum.verify object at 0x000001D565441A90>",
        "<enum.verify object at 0x000001D5654BFAD0>",
        "<enum.verify object at 0x000001D5654BE930>",
        "<enum.verify object at 0x000001D565442630>",
        "<enum.verify object at 0x000001D5654BD040>",
        "<enum.verify object at 0x000001D5654BCEC0>",
        "<enum.verify object at 0x000001D5654410D0>",
        "<enum.verify object at 0x000001D56536A750>",
        "<enum.verify object at 0x000001D5654428A0>",
        "<enum.verify object at 0x000001D5654BCB90>",
        "<enum.verify object at 0x000001D5654BDAC0>",
        "<enum.verify object at 0x000001D5654420C0>",
        "<enum.verify object at 0x000001D5654F8BF0>",
        "<enum.verify object at 0x000001D5654428A0>",
        "<enum.verify object at 0x000001D5654BD670>",
        "<enum.verify object at 0x000001D5654BD2B0>",
        "<enum.verify object at 0x000001D5654436B0>",
        "<enum.verify object at 0x000001D565035910>",
        "<enum.verify object at 0x000001D565443890>"
      ],
      [
        "RM24.90",
        "RM26.90",
        "RM19.90",
        "RM21.90",
        "RM7.90",
        "RM11.80",
        "RM5.90",
        "RM35.40",
        "RM5.90",
        "RM125.90",
        "RM60.90",
        "RM35.40",
        "RM53.90",
        "RM56.90",
        "RM132.90",
        "RM28.00",
        "RM27.90",
        "RM129.90",
        "RM25.90"
      ],
      [
        "4.975600739371534",
        "4.967707212055974",
        "5.0",
        "4.973096026490066",
        "4.947761194029851",
        "",
        "4.924050632911392",
        "4.932038834951456",
        "",
        "5.0",
        "4.956989247311828",
        "5.0",
        "4.968253968253968",
        "4.957055214723926",
        "5.0",
        "",
        "4.979838709677419",
        "4.826086956521739",
        "4.975452196382429"
      ],
      [
        "5410",
        "3716",
        "40",
        "2415",
        "134",
        "",
        "158",
        "103",
        "",
        "12",
        "279",
        "13",
        "63",
        "135",
        "7",
        "",
        "248",
        "23",
        "1547"
      ],
      [
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor",
        "Selangor"
      ],
      [
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter"
      ],
      [
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278",
        "100113278"
      ],
      [
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter",
        "Jobbie Nut Butter"
      ],
      [
        "16.5K sold",
        "13.0K sold",
        "128 sold",
        "7.5K sold",
        "483 sold",
        "483 sold",
        "576 sold",
        "285 sold",
        "285 sold",
        "42 sold",
        "689 sold",
        "30 sold",
        "128 sold",
        "529 sold",
        "33 sold",
        "6 sold",
        "646 sold",
        "73 sold",
        "5.2K sold"
      ],
      [
        "//www.lazada.com.my/products/jobbie-chunky-classic-peanut-butter-380g-i213113014.html",
        "//www.lazada.com.my/products/jobbie-chunky-pure-peanut-butter-380g-no-added-sugar-salt-i251404265.html",
        "//www.lazada.com.my/products/jobbie-peanut-butter-stirrer-i2114998534.html",
        "//www.lazada.com.my/products/jobbie-creamy-classic-peanut-butter-380g-i213110826.html",
        "//www.lazada.com.my/products/jobbie-love-spreader-i2114960965.html",
        "//www.lazada.com.my/products/jobbie-secret-to-inner-peace-wooden-spoon-wooden-fork-white-pouch-i4146743783.html",
        "//www.lazada.com.my/products/jobbie-key-to-happiness-i2115057537.html",
        "//www.lazada.com.my/products/jobbie-x-famous-amos-cookie-crumbs-peanut-spread-380g-i3657280759.html",
        "//www.lazada.com.my/products/jobbie-path-to-paradise-i4146740885.html",
        "//www.lazada.com.my/products/jobbie-peanut-butter-classic-gift-set-i2115142312.html",
        "//www.lazada.com.my/products/jobbie-peanut-butter-pure-jr-gift-set-i2115077737.html",
        "//www.lazada.com.my/products/jobbie-popping-candy-peanut-spread-380g-i4043983167.html",
        "//www.lazada.com.my/products/jobbie-peanut-butter-twin-jar-series-1x-jobbie-key-to-happiness-wooden-peanut-spoon-i4006744693.html",
        "//www.lazada.com.my/products/jobbie-peanut-butter-classic-jr-gift-set-i2115081659.html",
        "//www.lazada.com.my/products/jobbie-peanut-butter-pure-gift-set-i2115101606.html",
        "//www.lazada.com.my/products/jobbie-creamy-almond-choco-spread-200g-i4227602125.html",
        "//www.lazada.com.my/products/jobbie-pure-almond-butter-spread-200g-no-added-sugar-salt-i3787701442.html",
        "//www.lazada.com.my/products/jobbie-peanut-butter-starter-kit-i407069604.html",
        "//www.lazada.com.my/products/jobbie-creamy-pure-peanut-butter-380g-i251398638.html"
      ]
    
    ]
    // eslint-disable-next-line
    let keywords = [{"name":"男神","value":2.64},
                            {"name":"好身材","value":4.03},
                            {"name":"校草","value":24.95},
                            {"name":"酷","value":4.04},
                            {"name":"时尚","value":5.27},
                            {"name":"阳光活力","value":5.80},
                            {"name":"初恋","value":3.09},
                            {"name":"英俊潇洒","value":24.71},
                            {"name":"霸气","value":6.33},
                            {"name":"腼腆","value":2.55},
                            {"name":"蠢萌","value":3.88},
                            {"name":"青春","value":8.04},
                            {"name":"网红","value":5.87},
                            {"name":"萌","value":6.97},
                            {"name":"认真","value":2.53},
                            {"name":"古典","value":2.49},
                            {"name":"温柔","value":3.91},
                            {"name":"有个性","value":3.25},
                            {"name":"可爱","value":9.93},
                            {"name":"幽默诙谐","value":3.65}]
    this.rootStoreData = message
    //this.keyWords = keywords
    /*图片，文字等属性，组成一个个对象数组 */
    //this.products = this.combineArraysToObjectArray([this.rootStoreData[2], this.rootStoreData[0], this.rootStoreData[5],], "imgUrl", "name", "price")
    console.log(this.products);

  },

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url("//unpkg.com/element-ui@2.15.14/lib/theme-chalk/index.css");

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

.el-input{
  margin-bottom: 15px;
  width: 80%;
}


#averageRating {
  width: 500px;
  height: 10px;
}

.time {
  font-size: 20px;
  color: #999;
  position: absolute;
  left: 15%;
  top: 30%;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
  position: relative;
}

.button {
  float: right;
}

.image {
  width: 100%;
  display: block;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both
}

.main{
  width: 80%;
  margin: 0 auto;
}
.container{
  background-color: #f5f5dc;
  width: 80%;
  margin: 0 auto;
  position: relative;
}
.nameFence{
  height: 45px;
}
.cardMode{
  padding: 5px;
  margin-top: 10px;
}
.el-card {
    box-shadow: 5px 15px 20px rgba(0, 0, 0, 0.2); /* 自定义阴影大小 */
}
.buttonAnalyse{
  position: absolute;
  right: 5px;
  top: 5px;
}
.showgraph{
  width: 300px;
  height: 200px;
  margin: 0;
  padding: 0;
  /*background: #42b983;*/

  flex: 1 1 50%;
  height: 50%;
}
#showgraph1{
  
}
.graphContainer{
  display: flex;
  flex-wrap: wrap;
  height: 100vh;
}

.showgraph_next{
  width: 1200px;
  height: 800px;
  margin: 0;
  padding: 0;
  background-color: aqua;
  display: none;
}

</style>
