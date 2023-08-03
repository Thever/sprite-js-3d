<template>
  <div class="about">
    <p>绘制模型，简单的模型可以通过着色器+模型数据来实现渲染</p>
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
  name: 'DrawModel',
  components: {
  },
  mounted() {
    this.init()
  },  
  methods: {
    init() {
      //  导入场景 
      const {Scene} = spritejs;
      //  导入网格和着色器
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
        //  设置相机位置
        camera: {
          fov: 45, // 相机的视野
          pos: [-2, 2, 2] //  相机位置
        },
        //  方向光位置
        directionalLight: [0.5, 1.0, -0.3],
        //  方向光颜色
        directionalLightColor: [1, 1, 1, 0.15],
      });

      //  创建Program对象
      //  测试用简单的根据法向量来确定表面颜色的着色器
      const program = layer.createProgram(shaders.NORMAL);
      //  异步加载模型Json
      const model = layer.loadModel('https://s2.ssl.qhres2.com/static/bf607b5f64a91492.json');
      //  结合着色器和模型数据生成3D模型
      const macow = new Mesh3d(program, {model});
      //  图层添加3D模型
      layer.append(macow);
      //  将图层控制转向到 [0, 0.7, 0]
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