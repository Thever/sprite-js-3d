<template>
  <div class="about">
    <p>共享模型</p>
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
  name: 'ShareModel',
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
        },
      });

      //  创建Program对象
      const program = layer.createProgram({
        //  使用通用做色器
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

      //  共享Program对象
      const cube2 = new Cube(program, {
        //  配置颜色
        colors: 'green green green green green green',
      });
      //  调整位置
      cube2.attributes.pos = [1, 1, 1];
      //  图层添加立方体
      layer.append(cube2);


      //  使用 cloneNode 方法来拷贝内容
      const cube3 = cube.cloneNode();
      //  调整位置
      cube3.attributes.pos = [-1, -1, -1];
      //  缩放
      cube3.attributes.scale = 0.5;
      //  添加到图层
      layer.append(cube3);

      //  图层开启旋转控制
      layer.setOrbit(); // 开启旋转控制
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