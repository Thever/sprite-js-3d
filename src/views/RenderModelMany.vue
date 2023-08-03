<template>
  <div class="about">
    <p>批量渲染模型</p>
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
  name: 'RenderModelMany',
  components: {
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      const fragment = `
        precision highp float;
        precision highp int;

        varying vec3 vNormal;
        varying vec4 vColor;

        uniform vec4 lighting;

        void main() {
          vec3 normal = normalize(vNormal);
          float l = dot(normal, normalize(lighting.xyz));
          gl_FragColor.rgb = vColor.rgb + l * lighting.w;
          gl_FragColor.a = vColor.a;
        }
      `;
      const vertex = `
        precision highp float;
        precision highp int;

        attribute vec3 position;
        attribute vec3 normal;
        attribute vec4 color;
        attribute vec3 offset;
        attribute vec2 random;

        uniform mat4 modelViewMatrix;
        uniform mat4 projectionMatrix;
        uniform mat3 normalMatrix;

        varying vec3 vNormal;
        varying vec4 vColor;

        void rotate2d(inout vec2 v, float a){
          mat2 m = mat2(cos(a), -sin(a), sin(a),  cos(a));
          v = m * v;
        }

        void main() {
          vNormal = normalize(normalMatrix * normal);
          vColor = color;
          vec3 pos = position;
          rotate2d(pos.xz, random.x * 6.28);
          rotate2d(pos.xy, random.y * 6.28);
          gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0) + vec4(offset, 1.0);
        }    
      `;
      //  导入场景 
      const { Scene } = spritejs;
      //  导入立方体
      const { Cube } = spriteExtend3d;
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
          fov: 35, // 相机的视野
          pos: [0, 0, 7], // 相机位置
          // z: 8, // z轴位置
        },
        //  点光源
        // directionalLight: [0.5, 1.0, -0.3],
        // directionalLightColor: [1, 1, 1, 0.15]
      })
      //  创建一个Program对象,使用简单的根据法向量来确定表面颜色的着色器
      const program = layer.createProgram({
        //  顶点
        vertex,
        //  片段
        fragment,
        //  无切断横切面
        cullFace: null,
        //  设置uniforms属性
        uniforms: {
          lighting: { value: [-0.3, 0.8, 0.6, 0.1] },
        },
      }, {
        attributes: {
          offset(node, geometry) {
            console.log({ node, geometry })
            const data = new Float32Array(5 * 5 * 3);
            for (let i = 0; i < 5; i++) {
              const p = -5 + 2 * i;
              for (let j = 0; j < 5; j++) {
                const q = -5 + 2 * j;
                data.set([p, q, 0], (i * 5 + j) * 3);
              }
            }
            return { instanced: 1, size: 3, data };
          },
          random(node, geometry) {
            console.log({ node, geometry })
            const data = new Float32Array(5 * 5 * 2);
            for (let i = 0; i < 25; i++) {
              data.set([Math.random() * 2 - 1, Math.random() * 2 - 1], i * 2);
            }
            return { instanced: 1, size: 2, data };
          },
        },
      });
      //  实例化立方体
      const cube = new Cube(program);
      //  位置
      cube.attributes.pos = [0, 0, 0];
      //  面颜色
      cube.attributes.colors = 'red red blue blue orange orange';
      //  缩放
      cube.attributes.scale = 0.5;
      //  添加到图层
      layer.append(cube);
      //  添加动画
      cube.animate([
        { rotateY: 0 },
        { rotateY: -360 },
      ], {
        duration: 5000,
        iterations: Infinity,
      });
      //  图层添加控制
      layer.setOrbit();
    }
  }
}
</script>
<style scoped>
#stage {
  width: 600px;
  height: 600px;
  background-color: #FFF;
  margin: 0 auto;
  border: 2px solid skyblue;
}
</style>