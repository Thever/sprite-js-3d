<template>
  <div class="about">
    <p>可以使用结合材质贴图+模型数据来渲染模型</p>
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
  name: 'DrawTextureModel',
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
      //  导入材质图片
      const texture = layer.createTexture('https://p1.ssl.qhimg.com/t01b4bd0e2fb9f47550.jpg');
      //  着色器渲染材质图
      const program = layer.createProgram({
        //  支持光照、纹理的通用着色器，带纹理的几何元素可以使用它
        ...shaders.NORMAL_TEXTURE,
        texture,
      });

      //  异步加载模型Json
      const model = layer.loadModel('https://s2.ssl.qhres2.com/static/bf607b5f64a91492.json');
      //  结合着色发和模型数据生成3D模型
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