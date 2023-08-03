<template>
  <div class="about">
    <p>文字平面</p>
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
  name: 'TextPlane',
  components: {
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      //  导入场景 
      const { Scene } = spritejs;
      //  导入平面和着色器
      const { Plane, shaders } = spriteExtend3d;
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
          // pos: [-2, 2, 2] //  相机位置
        },
        //  方向光位置
        // directionalLight: [0.5, 1.0, -0.3],
        //  方向光颜色
        // directionalLightColor: [1, 1, 1, 0.15],
      });

      //  调整相机位置
      layer.camera.attributes.pos = [2, 2, 3];
      layer.camera.lookAt([0, 1.5, 0]);

      //  声明文字材质
      const texture = layer.createText('你好 SpriteJS 3D', {
        font: '48px Arial',
        fillColor: 'red',
      });
      //  声明 Program
      const program = layer.createProgram({
        //  一般材质
        ...shaders.NORMAL_TEXTURE,
        //  指定材质
        texture,
        //  无横切面
        cullFace: null,
      });

      //  文字平面
      const label = new Plane(program, {
        //  宽度
        width: 1,
        //  高度
        height: texture.image.height / texture.image.width,
        //  顶点颜色
        colors: 'transparent',
      });
      //  添加动画
      label.animate([
        { rotateY: 0 },
        { rotateY: 360 },
      ], {
        duration: 20000,
        iterations: Infinity,
      });
      //  添加到图层
      layer.append(label);
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