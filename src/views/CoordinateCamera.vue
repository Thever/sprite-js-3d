<template>
  <div class="about">
    <p>坐标与相机</p>
    <img src="../assets/coordinate.png" alt="">
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
  name: 'CoordinateCamera',
  components: {
  },
  mounted() {
    this.init()
  },  
  methods: {
    init() {
      //  导入场景 
      const {Scene} = spritejs;
      //  导入立方体和着色器
      const {Cube, shaders} = spriteExtend3d;
      //  渲染容器
      const container = document.getElementById('stage');
      //  实例化场景
      const scene = new Scene({container});
      //  图层使用3D渲染
      const layer = scene.layer3d('fglayer', {
        //  设置相机位置
        camera: {
          fov: 35, // 相机的视野
          pos: [3, 3, 5], // 相机的位置
          // near: 相机可以拍摄到的最近距离，默认为 0.1
          // far: 相机可以拍摄到的最远距离，默认为 100
          // preserveAspect: 默认为true，根据画布宽高比来保持相机宽高比，这样元素就不会被拉伸或压缩
        },
      });
      
      //  实例化相机
      const camera = layer.camera;
      // 默认情况下，相机的方向是朝着z轴负向无穷远处，而相机有一个方法叫lookAt，传入一个坐标，可以让相机朝向该位置拍摄。
      //  设置相机的朝向
      camera.lookAt([0, 0, 0]);

      //  创建Program对象
      const program = layer.createProgram({
        //  使用通用着色器
        ...shaders.NORMAL_GEOMETRY,
        //  无剔除面
        cullFace: null,
      });

      //  实例化立方体
      const cube = new Cube(program, {
        //  配置颜色
        colors: 'red red blue blue green green',
      });

      //  图层添加立方体
      layer.append(cube);
      //  图层开启旋转控制
      // layer.setOrbit(); // 开启旋转控制
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
