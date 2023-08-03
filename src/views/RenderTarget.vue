<template>
  <div class="about">
    <p>有时候我们需要多次渲染，比如先将一些内容渲染到FrameBuffer中，然后再输出到屏幕</p>
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
  name: 'RenderTarget',
  components: {
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      const vertex = /* glsl */ `
          precision highp float;
          precision highp int;

          attribute vec3 position;
          attribute vec3 normal;
          attribute vec2 uv;

          uniform mat4 modelViewMatrix;
          uniform mat4 projectionMatrix;
          uniform mat3 normalMatrix;

          varying vec2 vUv;
          varying vec3 vNormal;

          void main() {
              vUv = uv;
              vNormal = normalize(normalMatrix * normal);
              
              gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
          }
      `;

      const fragment = /* glsl */ `
          precision highp float;
          precision highp int;

          uniform sampler2D tMap;

          varying vec2 vUv;
          varying vec3 vNormal;

          void main() {
              vec3 normal = normalize(vNormal);
              float lighting = 0.2 * dot(normal, normalize(vec3(-0.3, 0.8, 0.6)));
              vec3 tex = texture2D(tMap, vUv).rgb;
              gl_FragColor.rgb = tex + lighting + vec3(vUv - 0.5, 0.0) * 0.1;
              gl_FragColor.a = 1.0;
          }
      `;
      //  导入场景 
      const { Scene } = spritejs;
      //  导入立方体和渲染对象
      const { Cube, RenderTarget } = spriteExtend3d;
      //  渲染容器
      const container = document.getElementById('stage');
      //  实例化场景
      const scene = new Scene({
        container,
        // 显示比率
        displayRatio: 2
      });
      //  图层使用3D渲染
      const layer = scene.layer3d('fglayer', {
        //  设置相机位置
        camera: {
          fov: 35, // 相机的视野
          // pos: [3, 3, 5], // 相机的位置
        },
      });
      //  调整图层摄像机
      layer.camera.attributes.pos = [0, 1, 5];
      layer.camera.lookAt([0, 0, 0]);

      //  创建材质
      const texture = layer.createTexture({
        image: new Uint8Array([
          191, 25, 54, 255,
          96, 18, 54, 255,
          96, 18, 54, 255,
          37, 13, 53, 255,
        ]),
        width: 2,
        height: 2,
        magFilter: layer.gl.NEAREST,
      });

      //  顶点、片段、材质生成 program
      const program = layer.createProgram({
        vertex,
        fragment,
        texture,
      });

      //  实例化渲染对象
      const target = new RenderTarget(layer.gl, {
        width: 512,
        height: 512,
        camera: {
          fov: 35,
        },
      });
      //  设置渲染对象相机
      target.camera.attributes.pos = [0, 1, 5];
      target.camera.lookAt([0, 0, 0]);

      //  生成立方体
      const mesh = new Cube(program);
      //  添加到渲染对象中
      target.append(mesh);

      //  创建渲染对象 program
      const targetProgram = layer.createProgram({
        vertex,
        fragment,
        texture: target.texture,
      });

      //  监听事件处理
      target.addEventListener('beforerender', (e) => {
        console.log({e})
        layer.gl.clearColor(0.15, 0.05, 0.2, 1);
      });

      target.addEventListener('afterrender', (e) => {
        console.log({e})
        layer.gl.clearColor(1, 1, 1, 0);
      });

      //  实例化渲染对象
      const mesh2 = new Cube(targetProgram);
      layer.append(mesh2);
      
      //  监听变化
      layer.tick(() => {
        layer.renderTarget(target);
        mesh.attributes.rotateY -= 1.5;
        mesh2.attributes.rotateY -= 0.9;
        mesh2.attributes.rotateX -= 1;
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