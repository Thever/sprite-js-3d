<template>
  <div class="about">
    <p>渲染模式：</p>
    <ul>
      <li>POINTS 以gl.POINTS模式渲染</li>
      <li>LINES 以gl.LINES模式渲染</li>
      <li>LINE_LOOP 以gl.LINE_LOOP模式渲染</li>
      <li>LINE_STRIP 以gl.LINE_STRIP模式渲染</li>
      <li>TRIANGLES 默认值，以gl.TRIANGLES模式渲染</li>
    </ul>
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
  name: 'RenderModeOther',
  components: {
  },
  mounted() {
    this.init()
  },  
  methods: {
    init() {
      //  导入场景 
      const {Scene} = spritejs;
      //  导入3d网格,着色器
      const {Mesh3d, shaders} = spriteExtend3d;
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
        camera: {
          fov: 45, // 相机的视野
          pos: [-2, 2, 2], // 相机位置
        },
        //  点光源
        directionalLight: [0.5, 1.0, -0.3],
        directionalLightColor: [1, 1, 1, 0.15]
      })
      //  创建一个Program对象,使用简单的根据法向量来确定表面颜色的着色器
      const program = layer.createProgram(shaders.NORMAL);
      //  加载model数据
      const model = layer.loadModel('https://s2.ssl.qhres2.com/static/bf607b5f64a91492.json');

      //  着色器配置+mode数据+渲染模式来生成3d网格
      //  mode选择
      //  LINES 纯点连线
      //  LINE_LOOP 点回路
      //  LINE_STRIP 点条纹
      //  TRIANGLES 多边形面
      const macow = new Mesh3d(program, {model, mode: 'LINE_STRIP'});
      //  添加到图层
      layer.append(macow);
      //  加载控制，转动到特定角度
      layer.setOrbit({target: [0, 0.7, 0]});
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