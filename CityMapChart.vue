<template>
  <div class="chat">
    <div ref="map" class="map"></div>
    <div class="contextMenu" @click="returnBack">返回上一级</div>
  </div>
</template>
<script>
import Echarts from "echarts";
import Axios from "axios";
import CityMap from "../../map/city-map";
//import CityMap from "./china-main-city-map.js";
import ProvinceMap from "../../map/province-map";

export default {
  data() {
    return {
      mapJsonData: null,
      chart: null,
      curMap: {
        mapCode: null,
        mapName: null
      },
      mapStack: [],
      option_map:null,
      geoCoordMap: {
        武汉: [114.31, 30.52]
      },
      testData: [
        {
          name: "武汉",
          value: 36
        }
      ]
    };
  },
  mounted() {
    let _this = this;
    //地图配置选项
    this.option_map ={
      
        title: {
          show: false,
          text: "湖北省检查院地图"
        },
        grid: {
          left: "5%",
          right: "4%",
          top: "0%",
          bottom: "0%",
          width: "100%",
          containLabel: true
        },
        series: [
          {
            data: [
              { ReplayNum:100,LiveCastNum: 335, name: "十堰市" },
              { ReplayNum:60,LiveCastNum: 310, name: "武汉市" },
              { ReplayNum:30,LiveCastNum: 274, name: "襄阳市" },
              { ReplayNum:70,LiveCastNum: 235, name: "咸宁市" },
              { ReplayNum:22,LiveCastNum: 400, name: "荆州市" }
            ],
            roam: true, //是否允许鼠标滚动放大，缩小
            type: "map",
            mapType: "hubei",
            selectedMode: "single",
            label: {
              normal: {
                show: true,
                areaColor: "#ffefd5", //区域颜色
                textStyle: { color: "#05ffff" } //省份标签字体颜色
              }
            },
            itemStyle: {
              borderWidth: 0.5,
              borderColor: "#4779c9",
              // 高亮时点的颜色。
              color: "blue",
              areaColor: "rgba(128, 128, 128, 0.1)"
            },
            emphasis: {
              //高亮状态下的地图和标签样式。
              label: {
                //文本
                color: "#05ffff"
              },
              itemStyle: {
                //区域
                areaColor: "#285aa0"
              }
            }
          }
        ],
        tooltip: {
          formatter: function(params) {
            for (var i = 0; i < _this.option_map.series[0].data.length; i++) {
              if (_this.option_map.series[0].data[i].name == params.name) {
                var info = '<p style="font-size:18px">' + params.name + '</p><p style="font-size:14px">正在直播数:'+_this.option_map.series[0].data[i].LiveCastNum+'</p><p style="font-size:14px">即将直播数:'+_this.option_map.series[0].data[i].ReplayNum+'</p>'
                return info;
              }
            }
          }
        }
    }
    //注册地图
    this.myChart = Echarts.init(this.$refs.map);
    //初始化地图数据
    this.loadMap("420000", "hubei");
    //地图下探
    this.myChart.on("mapselectchanged", this.onCityClick);
  },
  methods: {
    loadMap(mapCode, mapName) {
      this.$axios
        .get("../../../static/china-main-city/" + mapCode + ".json")
        .then(response => {
          this.mapJsonData = response.data;
          if (this.mapJsonData) {
            this.myChart.clear();
            console.log(mapName);
            Echarts.registerMap(mapName, this.mapJsonData);
            this.myChart.setOption(this.option_map);
            this.curMap.mapCode = mapCode;
            this.curMap.mapName = mapName;
          } else {
            alert("无法加载此地图");
          }
        });
    },
    onCityClick(e) {
      let name = e.batch[0].name;
      var mapCode = CityMap[name];
      if (!mapCode) {
        alert("无此区域地图显示");
        return;
      } else {
        this.option_map.series[0].mapType = mapCode;
        this.loadMap(mapCode, mapCode);
        this.mapStack.push({
          mapCode: this.curMap.mapCode,
          mapName: this.curMap.mapName
        });
      }
    },
    returnBack() {
      //获取上一级地图信息
      var map = this.mapStack.pop();
      if (!this.mapStack.length && !map) {
        console.log(this.mapStack.length);
        alert("已经到达最上一级地图了");
        return;
      }
      console.log(map.mapCode + ":" + map.mapName);
      this.option_map.series[0].mapType = map.mapCode;
      this.loadMap(map.mapCode, map.mapCode);
    }
  }
};
</script>

<style lang="less">
.chat {
  width: 100%;
  height: 100%;
}
.map {
  width: 100%;
  height: 100%;
}
.contextMenu {
  position: absolute;
  //background: #000;
  background-image: url("../../assets/images/btnDefault.png");
  background-size: 100% 107%;
  opacity: 0.8;
  cursor: pointer;
  border-radius: 2px;
  padding: 8px 30px;
  color: #fff;
  font-size: 14px;
  right: 10%;
  bottom: 10%;
}
.contextMenu:hover {
  background-image: url("../../assets/images/btnActive.png");
}
</style>