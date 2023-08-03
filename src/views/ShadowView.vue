<template>
  <div class="about">
    <p>绘制阴影</p>
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
  name: 'ShadowView',
  components: {
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      //  导入场景 
      const { Scene } = spritejs;
      //  导入相机、3d网格，平面和着色器
      const { Camera, Mesh3d, Plane, shaders } = spriteExtend3d;
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
          // pos: [3, 3, 5], // 相机的位置
        },
      });
      //  调整图层相机位置
      layer.camera.attributes.pos = [5, 4, 10];
      //  添加控制
      layer.setOrbit();

      //  设置图层光
      const light = new Camera(layer.gl, {
        left: -3,
        right: 3,
        bottom: -3,
        top: 3,
        near: 1,
        far: 20,
      });
      light.attributes.pos = [3, 10, 3];
      light.lookAt([0, 0, 0]);

      //  创建影子
      const shadow = layer.createShadow({ light });

      //  材质
      const texture = layer.createTexture('https://p2.ssl.qhimg.com/t01155feb9a795bdd05.jpg');
      //  模型
      const model = layer.loadModel('https://s0.ssl.qhres2.com/static/0baccc5ad3cd5b8c.json');
      //  创建 program
      const program = layer.createProgram({
        ...shaders.TEXTURE_WITH_SHADOW,
        cullFace: null,
        texture,
      });
      //  渲染飞机
      const plane = new Mesh3d(program, { model });
      //  飞机暴露到window下
      window.plane = plane;
      //  添加到图层
      layer.append(plane);

      //  水面材质
      const waterTexture = layer.createTexture('https://p0.ssl.qhimg.com/t01db936e50ab52f10a.jpg');
      //  水面program
      const program2 = layer.createProgram({
        ...shaders.TEXTURE_WITH_SHADOW,
        cullFace: null,
        texture: waterTexture,
      });
      //  创建平面-水面
      const ground = new Plane(program2, {
        rotateX: 90,
        scale: 6,
        y: -3,
      });
      //  添加水面到场景
      layer.append(ground);

      //  影子添加内容
      shadow.add(plane);
      shadow.add(ground);
      //  图层添加到图层
      layer.setShadow(shadow);

      //  图层变化
      layer.tick((t) => {
        // A bit of plane animation
        //  如果渲染了飞机
        if (plane) {
          //  调整飞机位置
          plane.attributes.z = Math.sin(t * 0.001);
          plane.attributes.rotateX = Math.sin(t * 0.001 + 2) * 18;
          plane.attributes.rotateY = Math.sin(t * 0.001 - 4) * -18;
        }
      });
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