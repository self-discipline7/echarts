<template>
  <div ref="mapContainer" :style="{ width: width, height: height }"></div>
</template>

<script>
import * as echarts from 'echarts';
import chinaMapData from '@/china.json';

export default {
  name: 'ChinaMap',
  props: {
    width: {
      type: String,
      default: '800px'
    },
    height: {
      type: String,
      default: '600px'
    }
  },
  data() {
    return {
      chart: null,
      // 中国主要城市坐标
      majorCities: [
        { name: '北京', value: [116.4074, 39.9042] },
        { name: '上海', value: [121.4737, 31.2304] },
        { name: '广州', value: [113.2644, 23.1291] },
        { name: '深圳', value: [114.0579, 22.5431] },
        { name: '杭州', value: [120.1918, 30.2642] },
        { name: '成都', value: [104.0665, 30.5728] },
        { name: '武汉', value: [114.3054, 30.5928] },
        { name: '重庆', value: [106.504962, 29.533155] },
        { name: '天津', value: [117.1902, 39.1256] },
        { name: '南京', value: [118.778074, 32.041545] },
        { name: '苏州', value: [120.619585, 31.299379] },
        { name: '西安', value: [108.948024, 34.263161] },
        { name: '长沙', value: [112.982279, 28.19409] },
        { name: '青岛', value: [120.369585, 36.067191] },
        { name: '郑州', value: [113.665412, 34.757975] },
        { name: '大连', value: [121.614722, 38.914001] },
        { name: '厦门', value: [118.089429, 24.479833] },
        { name: '济南', value: [117.02193, 36.651221] },
        { name: '合肥', value: [117.272535, 31.86119] },
        { name: '福州', value: [119.306239, 26.075302] }
      ],
      // 飞线颜色池
      lineColors: ['#FFFF00', '#FFD700', '#FFA500', '#FF6347', '#FF4500', '#FF00FF', '#9932CC', '#8A2BE2', '#4B0082', '#0000FF']
    };
  },
  mounted() {
    this.initChart();
    window.addEventListener('resize', this.handleResize);
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
    }
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    initChart() {
      // 初始化图表
      this.chart = echarts.init(this.$refs.mapContainer);
      
      // 注册地图
      echarts.registerMap('china', chinaMapData);
      
      // 随机选择十个城市（包括北京）
      const selectedCities = this.getRandomCities(10);
      
      // 准备散点数据
      const scatterData = selectedCities.map(city => ({
        name: city.name,
        value: city.value
      }));
      
      // 准备飞线数据（从北京到其他城市）
      const beijing = selectedCities.find(city => city.name === '北京');
      const otherCities = selectedCities.filter(city => city.name !== '北京');
      
      const linesData = otherCities.map((city, index) => {
        return {
          from: beijing.name,
          to: city.name,
          coords: [beijing.value, city.value],
          lineStyle: {
            color: this.lineColors[index % this.lineColors.length],
            type: 'smooth'
          }
        };
      });
      
      // 配置项
      const option = {
        backgroundColor: '#2A2A2A',
        title: {
          text: '中国主要城市分布及飞线图',
          left: 'center',
          textStyle: {
            color: '#fff'
          }
        },
        tooltip: {
          trigger: 'item',
          formatter: function(params) {
            if (params.seriesType === 'scatter') {
              return params.name;
            } else if (params.seriesType === 'lines') {
              return `${params.data.from} → ${params.data.to}`;
            }
            return params.name;
          }
        },
        geo: {
          type: 'map',
          map: 'china',
          label: {
            emphasis: {
              show: true,
              color: '#fff'
            }
          },
          roam: true,
          itemStyle: {
            normal: {
              areaColor: '#1a237e',
              borderColor: '#2196f3'
            },
            emphasis: {
              areaColor: '#283593'
            }
          }
        },
        series: [
          // 飞线系列
          {
            type: 'lines',
            zlevel: 1,
            effect: {
              show: true,
              period: 6,
              trailLength: 0.7,
              color: '#fff',
              symbol: 'arrow',
              symbolSize: 15
            },
            lineStyle: {
              type: 'solid',
              width: 1,
              opacity: 0.6
            },
            data: linesData
          },
          // 散点系列
          {
            type: 'scatter',
            coordinateSystem: 'geo',
            zlevel: 2,
            symbol: 'pin',
            symbolSize: 20,
            label: {
              show: true,
              formatter: '{b}',
              position: 'right',
              color: '#fff'
            },
            itemStyle: {
              color: '#FF6B6B'
            },
            data: scatterData
          }
        ]
      };
      
      // 设置配置项
      this.chart.setOption(option);
    },
    // 随机选择n个城市（确保包含北京）
    getRandomCities(n) {
      // 确保至少选择北京
      const result = [this.majorCities.find(city => city.name === '北京')];
      
      // 随机选择其他城市
      const otherCities = this.majorCities.filter(city => city.name !== '北京');
      
      while (result.length < n) {
        const randomIndex = Math.floor(Math.random() * otherCities.length);
        const randomCity = otherCities[randomIndex];
        
        // 确保不重复选择同一城市
        if (!result.find(city => city.name === randomCity.name)) {
          result.push(randomCity);
        }
      }
      
      return result;
    },
    handleResize() {
      if (this.chart) {
        this.chart.resize();
      }
    }
  }
};
</script>