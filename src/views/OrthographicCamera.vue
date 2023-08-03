<template>
  <div class="about">
    <p>正交相机</p>

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
  name: 'OrthographicCamera',
  components: {
  },
  mounted() {
    this.init()
  },  
  methods: {
    init() {
      const vertex = /* glsl */ `
          attribute vec2 uv;
          attribute vec3 position;

          uniform mat4 modelViewMatrix;
          uniform mat4 projectionMatrix;

          varying vec2 vUv;
          varying vec4 vMVPos;
          varying vec3 vPos;

          void main() {
              vUv = uv;
              vPos = position;
              vMVPos = modelViewMatrix * vec4(position, 1.0);
              gl_Position = projectionMatrix * vMVPos;
          }
      `;

      const fragment = /* glsl */ `
          precision highp float;

          uniform sampler2D tMap;

          varying vec2 vUv;
          varying vec4 vMVPos;
          varying vec3 vPos;

          void main() {
              vec3 tex = texture2D(tMap, vUv).rgb;
              
              float dist = length(vMVPos);
              float fog = smoothstep(2.0, 15.0, dist);
              tex = mix(tex, vec3(1), fog * 0.8);
              tex = mix(tex, vec3(1), smoothstep(1.0, 0.0, vPos.y)); 
              
              gl_FragColor.rgb = tex;
              gl_FragColor.a = 1.0;
          }
      `;

      //  导入场景 
      const {Scene} = spritejs;
      //  导入正交相机，3d网格，圆柱/锥体，着色器
      const {Camera, Mesh3d, Cylinder, shaders} = spriteExtend3d;
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
          // pos: [3, 3, 5], // 相机的位置
        },
        //  白色环境光
        ambientColor: 'white',
      });
      //  
      layer.gl.clearColor(1, 1, 1, 1);
      //  调整相机角度
      layer.camera.attributes.pos = [6, 6, 12];
      //  调整相机走向
      // layer.camera.lookAt([0, 0, 0]);

      //  材质
      const texture = layer.createTexture('https://p5.ssl.qhimg.com/t01749f23f82ef86c9f.jpg');
      //  顶点，片段，材质，生成 program
      const program = layer.createProgram({
        vertex,
        fragment,
        texture,
      });

      //  modal
      const model = layer.loadModel('https://s4.ssl.qhres2.com/static/4d2c8de20e171997.json');
      //  尺寸
      const size = 20;
      //  数量
      const num = size * size;

      //  循环生成数据
      for(let i = 0; i < num; i++) {
        const tree = new Mesh3d(program, {model});
        layer.append(tree);
        tree.attributes.pos = [(i % size - size * 0.5) * 2, 0, (Math.floor(i / size) - size * 0.5) * 2];
        tree.attributes.y += Math.sin(tree.attributes.x * 0.5) * Math.sin(tree.attributes.z * 0.5) * 0.5;
        tree.attributes.rotateY = Math.random() * Math.PI * 2;
        tree.attributes.scale = 0.8 + Math.random() * 0.3;
      }

      //  
      const cameraProgram = layer.createProgram({
        //  使用通用着色器
        ...shaders.NORMAL,
        //  无剔除面
        cullFace: null,
      });

      // const frustumTransform = new Group3d();

      //  生成圆锥形状
      const cameraShape = new Cylinder(cameraProgram, {
        radiusBottom: 0.2,
        height: 0.7,
        radialSegments: 4,
        openEnded: true,
        rotateOrder: 'XYZ',
        rotateX: -90,
        rotateY: 45,
      });

      //  生成相机路径
      function cameraPath(time, y) {
        const x = 4 * Math.sin(time);
        const z = 2 * Math.sin(time * 2);
        return [x, y, z];
      }

      //  生成正交相机
      const frustumCamera = new Camera(layer.gl, {
        //  广角
        fov: 65,
        //  距离
        far: 10,
      });
      //  将相机添加圆锥外形
      frustumCamera.append(cameraShape);
      //  添加相机到图层
      layer.append(frustumCamera);
      //  图层添加控制
      layer.setOrbit();
      //  图层变化
      layer.tick((t) => {
        //  更改相机位置
        frustumCamera.attributes.pos = cameraPath(t * 0.001, 2);
        //  更改相机朝向
        const target = cameraPath(t * 0.001 + 1, 1);
        frustumCamera.lookAt(target);
        //  worldMatrix
        frustumCamera.updateMatrixWorld();
        //  更新视锥体
        frustumCamera.updateFrustum();

        // Traverse all meshes in the scene
        // 遍历图层中的所有内容
        layer.traverse((node) => {
          //  过滤没有绘制的
          if(!node.body.draw) return;
          //  过滤正交相机外形
          if(node === cameraShape) return;
          //  只显示相机看的到的部分
          node.attributes.display = frustumCamera.frustumIntersects(node) ? '' : 'none';
        });
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