<template>
  <div class="about">
    <p>绘制几何体</p>
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
  name: 'DrawGeometry',
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
      const {Mesh3d, shaders} = spriteExtend3d;
      //  渲染容器
      const container = document.getElementById('stage');
      //  实例化场景
      const scene = new Scene({
        container,
        width: 600,
        height: 600
      });
      //  图层使用3D渲染
      const layer = scene.layer3d('fglayer', {
        //  设置相机位置
        camera: {
          fov: 35, // 相机的视野
          z: 5, //  z轴坐标
        },
      });

      //  创建Program对象
      const program = layer.createProgram({
        //  使用通用做色器
        ...shaders.NORMAL_GEOMETRY,
        //  无剔除面
        cullFace: null,
      });

      //  根号2 / 2
      const p = 1 / Math.sqrt(2);
      //  设置集合数据
      const model = {
        position: [
          -1, 0, -p,
          1, 0, -p,
          0, 1, p,
          -1, 0, -p,
          1, 0, -p,
          0, -1, p,
          1, 0, -p,
          0, -1, p,
          0, 1, p,
          -1, 0, -p,
          0, 1, p,
          0, -1, p],
      };

      const sprite = new Mesh3d(program, {
        //  几何数据
        model,
        //  定点颜色
        colors: 'red blue green orange',
      });
      layer.append(sprite);

      //  添加动画
      sprite.animate([
        //  y轴为中心从0-360度旋转
        {rotateY: 0},
        {rotateY: 360},
      ], {
        //  7秒1次
        duration: 7000,
        //  无限循环
        iterations: Infinity,
      });

      sprite.animate([
        //  z轴为中心从0-360度旋转
        {rotateZ: 0},
        {rotateZ: 360},
      ], {
        //  17秒1次
        duration: 17000,
        //  无限循环
        iterations: Infinity,
      });

      sprite.animate([
        //  x轴为中心从0-360度旋转
        {rotateX: 0},
        {rotateX: -360},
      ], {
        //  11秒1次
        duration: 11000,
        //  无限循环
        iterations: Infinity,
      });
      //  图层增加控制
      layer.setOrbit();
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