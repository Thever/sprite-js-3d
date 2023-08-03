<template>
  <div class="about">
    <p>开启控制旋转：layer.setOrbit({target: [x, y, z]});</p>
    <p>让元素支持点击事件：layer.setRaycast();</p>
    <div id="stage"></div>
  </div>
</template>
<script>
// @ is an alias to /src
import * as spritejs from 'spritejs';
import * as spriteExtend3d from 'sprite-extend-3d';
// 本地导入json
// import * as birdsJsonUrl from '@/assets/5f6911b7b91c88da.json'
export default {
  name: 'ElementInteraction',
  components: {
  },
  mounted() {
    this.init()
  },  
  methods: {
    init() {
      //  导入场景 
      const {Scene} = spritejs;
      //  导入圆柱/锥， 球体，立方体和着色器
      const {Cylinder, Sphere, Cube, shaders} = spriteExtend3d;
      //  渲染容器
      const container = document.getElementById('stage');
      //  实例化场景
      const scene = new Scene({
        container,
        // 显示比率
        displayRatio: 2,
      });
      //  图层使用3D渲染
      const layer = scene.layer3d('fglayer', {
        //  环境光
        // ambientColor: '#ff000080',
        //  方向光位置
        directionalLight: [1, 0, 0],
        //  方向光颜色
        directionalLightColor: [1, 1, 1, 0.5],
        //  点光源颜色
        pointLightColor: `hsl(${Math.floor(360 * Math.random())}, 50%, 50%)`,
        //  点光源颜色
        pointLightPosition: [5, 3, 6],
        camera: {
          fov: 35, // 相机的视野
          pos: [3, 3, 5], // 相机的位置
        },
      });

      //  实例化图层相机
      const camera = layer.camera;
      //  设置相机位置
      camera.attributes.pos = [5, 3, 6]
      //  设置相机朝向
      camera.lookAt([0, 0, 0]);
      
      //  创建program对象
      const program = layer.createProgram({
        //  使用通用做色器
        ...shaders.NORMAL_GEOMETRY,
        //  无剔除面
        cullFace: null,
      });

      // 实例化圆柱/椎体
      const cylinder = new Cylinder(program);
      //  设置位置
      cylinder.attributes.pos = [0, 1.3, 0];
      //  添加到图层
      layer.append(cylinder);
      //  添加动画
      cylinder.animate([
        {rotateY: 0},
        {rotateY: -360},
      ], {
        duration: 10000,
        iterations: Infinity,
      });

      //  实例化球体
      const sphere = new Sphere(program);
      //  phi(π)结束的度数
      sphere.attr({
        phiLength: Math.PI,
      });
      //  添加到图层
      layer.append(sphere);
      //  添加动画
      sphere.animate([
        {rotateY: 0},
        {rotateY: -360},
      ], {
        duration: 7500,
        iterations: Infinity,
      });

      //  实例化立方体
      const cube = new Cube(program);
      //  调整位置
      cube.attributes.pos = [0, -1.3, 0];
      //  添加到图层
      layer.append(cube);
      //  添加动画
      cube.animate([
        {rotateY: 0},
        {rotateY: -360},
      ], {
        duration: 5000,
        iterations: Infinity,
      });
      //  图层开启元素点击事件支持 
      layer.setRaycast();
      //  图层添加点击事件
      layer.addEventListener('click', (evt) => {
        console.log(evt)
        console.log(evt.target)
        //  这里的 target 不是一种类型，而是实例对象
        //  后续同一场景里多个对象交互的话要每个单独区分了
        if(evt.target === cube) {
          console.log(`cube`)
          const colors = [];
          for(let i = 0; i < 3; i++) {
            const randomColor = `hsl(${Math.floor(360 * Math.random())}, 50%, 50%)`;
            colors.push(randomColor, randomColor);
          }
          evt.target.attributes.colors = colors;
        } else if(evt.target !== layer) {
          console.log(`!== layer`)
          evt.target.attributes.colors = `hsl(${Math.floor(360 * Math.random())}, 50%, 50%)`;
        }
      });
    }
  }
}
</script>
<style scoped>
#stage{
  width:600px;
  height:360px;
  background-color:#FFF;
  margin:0 auto;
  border:2px solid skyblue;
}
</style>