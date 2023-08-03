<template>
  <div class="about">
    <p>Group3d 分组</p>
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
  name: 'GroupView',
  components: {
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      //  导入场景 
      const { Scene } = spritejs;
      //  导入立方体,球体，3D分组，着色器
      const { Cube, Sphere, Group3d, shaders } = spriteExtend3d;
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
        //  设置相机位置
        camera: {
          fov: 35, // 相机的视野
          pos: [5, 3, 6], // 相机的位置
        },
      });
      //  控制摄像头朝向
      layer.camera.lookAt([0, -0.5, 0]);

      //  着色器一般渲染无横切面
      const program = layer.createProgram({
        ...shaders.NORMAL,
        cullFace: null,
      });

      //  定义球体
      const s1 = new Sphere(program, {
        phiLength: Math.PI, // 半球
        x: -1,
        rotateY: -90,
      });

      //  克隆球体
      const s2 = s1.cloneNode();
      //  更改位置，旋转角度
      s2.attr({
        x: 1,
        rotateY: 90,
      });

      //  立方体
      const c = new Cube(program, {
        rotateY: 45,
        scale: 0.5,
      });

      //  声明分组
      const group = new Group3d();

      //  添加到分组
      group.append(s1, s2, c);

      //  分组添加到图层
      layer.append(group);

      //  分组添加动画
      group.animate([
        { rotateY: 0 },
        { rotateY: 360 },
      ], {
        duration: 19000,
        iterations: Infinity,
      });

      c.animate([
        { rotateY: 0 },
        { rotateY: 360 },
      ], {
        duration: 11000,
        iterations: Infinity,
      });

      //  增加控制
      layer.setOrbit();
    }
  }
}
</script>
<style scoped>
#stage {
  width: 600px;
  height: 360px;
  background-color: #FFF;
  margin: 0 auto;
  border: 2px solid skyblue;
}
</style>