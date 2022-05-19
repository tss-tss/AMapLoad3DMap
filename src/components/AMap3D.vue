<template>
  <div class="amap" id="container">amap3d</div>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader';
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'

export default {
  name: 'amap',
  data() {
    return {
      map: null
    }
  },
  created() {
    this.$bus.$on('echartsMapClick', res => {
      this.map.destroy();
      this.map = null;
      this.mapLoad();
    })
  },
  mounted() {
    this.mapLoad()
  },
  methods: {
    mapLoad() {
      AMapLoader.load({
        "key": "your key", // 申请好的Web端开发者Key，首次调用 load 时必填
        "version": "2.0",   // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        "plugins": [],      // 需要使用的的插件列表，如比例尺'AMap.Scale'等
      }).then((AMap) => {
        this.map = new AMap.Map('container', {
          zoom: 10.5,//级别
          center: [114.530148, 24.742353],//中心点坐标
          viewMode: '3D',//使用3D视图
          mapStyle: "amap://styles/darkblue",
          pitch: 40,
          pitchEnable: false,
          features: [], //设置地图上显示的元素种类具体查看官方文档，这里设置为空达到地图透明的效果
          showLabel: false, // 不显示label文字
          opacity: 0, // 透明度
          skyColor: "rgba(0,0,0,0)", // 天空透明度
        });
        this.load3D();
        // this.drawDistrictLine();
      }).catch(e => {
        console.log(e);
      })
    },
    // 创建 GL 图层，加载3d模型
    load3D() {
      let camera;
      let renderer;
      let scene;
      // 数据转换工具
      let customCoords = this.map.customCoords;
      let gllayer = new AMap.GLCustomLayer({
        // 图层的层级
        zIndex: 10,
        // 初始化的操作，创建图层过程中执行一次。
        init: (gl) => {
          // 这里我们的地图模式是 3D，所以创建一个透视相机，相机的参数初始化可以随意设置，因为在 render 函数中，每一帧都需要同步相机参数，因此这里变得不那么重要。
          // 如果你需要 2D 地图（viewMode: '2D'），那么你需要创建一个正交相机
          camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 100, 1 << 30);

          renderer = new THREE.WebGLRenderer({
            context: gl,  // 地图的 gl 上下文
            // alpha: true,
            // antialias: true,
            // canvas: gl.canvas,
          });

          // 自动清空画布这里必须设置为 false，否则地图底图将无法显示
          renderer.autoClear = false;
          scene = new THREE.Scene();

          // 环境光照和平行光
          let aLight = new THREE.AmbientLight(0xffffff, 1);
          let dLight = new THREE.DirectionalLight(0xffffff, 0.3);
          dLight.position.set(1000, -100, 900);
          scene.add(dLight);
          scene.add(aLight);

          // 数据使用转换工具进行转换，这个操作必须要提前执行（在获取镜头参数 函数之前执行），否则将会获得一个错误信息。
          let data = customCoords.lngLatsToCoords([
            [114.522148, 24.836353]
          ]);

          const gltfLoader = new GLTFLoader()
          gltfLoader.load('/gltf/quannan.gltf', (gltf) => {
            let model = gltf.scene;
            model.position.set(data[0][0], data[0][1], 10)
            const scaleNum = 600
            model.scale.set(scaleNum, scaleNum, scaleNum)
            model.rotation.set(0.5 * Math.PI, 0, 0)
            scene.add(model)
          })
        },
        render: () => {
          // 这里必须执行！！重新设置 three 的 gl 上下文状态。
          renderer.state.reset();
          let { near, far, fov, up, lookAt, position } = customCoords.getCameraParams();

          // 2D 地图下使用的正交相机
          // let { near, far, top, bottom, left, right, position, rotation } = customCoords.getCameraParams();

          // 这里的顺序不能颠倒，否则可能会出现绘制卡顿的效果。
          camera.near = near;
          camera.far = far;
          camera.fov = fov;
          camera.position.set(...position);
          camera.up.set(...up);
          camera.lookAt(...lookAt);
          camera.updateProjectionMatrix();

          // 2D 地图使用的正交相机参数赋值
          // camera.top = top;
          // camera.bottom = bottom;
          // camera.left = left;
          // camera.right = right;
          // camera.position.set(...position);
          // camera.updateProjectionMatrix();

          renderer.render(scene, camera);
        },
      });

      this.map.add(gllayer);
    },
    drawDistrictLine() {
      AMap.plugin('AMap.DistrictSearch', () => {
        // 创建行政区查询对象
        let district = new AMap.DistrictSearch({
          // 返回行政区边界坐标等具体信息
          extensions: 'all',
          // 设置查询行政区级别为 区 
          level: 'district'
        })

        district.search('全南县', (status, result) => {
          // 获取全南县的边界信息
          let bounds = result.districtList[0].boundaries
          let polygons = []
          if (bounds) {
            for (let i = 0, l = bounds.length; i < l; i++) {
              //生成行政区划polygon
              let polygon = new AMap.Polygon({
                map: this.map,
                strokeWeight: 1,
                path: bounds[i],
                fillOpacity: 0,
                fillColor: '#CCF3FF',
                strokeColor: '#CC66CC'
              })
              polygons.push(polygon)
            }
            // 地图自适应
            this.map.setFitView()
          }
        })
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.amap {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 0;
}
</style>
<style lang="scss">
.amap-container {
  background: none !important;
}
</style>
