<template>
  <div>
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
    <el-row v-show="showCards">
      <el-col :span="8" v-for="(o, index) in products" :key="o" :offset="index > 0 ? 2 : 0">
        <el-card :body-style="{ padding: '0px' }">
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
    <!--<el-button type="primary" style="margin-left: 16px" @click="change">
      打开
    </el-button>-->

    <el-drawer :visible.sync="drawer" title="I am the title" :with-header="false">
      <el-row>
        <el-col :span="8" v-for="(o, index) in products" :key="o" :offset="index > 0 ? 2 : 0">
          <el-card :body-style="{ padding: '0px' }">
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
      showCards: false
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
      this.showCards = true
      //console.log(this.storeUrl)
      this.products = this.combineArraysToObjectArray2(this.storeUrl,[this.rootStoreData[2], this.rootStoreData[0], this.rootStoreData[5],], "imgUrl", "name", "price")
    },
    pb(){ 
          //console.log(response.data);

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
    this.rootStoreData = message
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
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
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
</style>
