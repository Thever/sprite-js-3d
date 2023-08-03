<template>
  <div class="about">
    <p>粒子渲染模式</p>
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
  name: 'RenderModePoints',
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
        attribute vec4 random;

        uniform mat4 modelMatrix;
        uniform mat4 viewMatrix;
        uniform mat4 projectionMatrix;
        uniform float uTime;

        varying vec4 vRandom;

        void main() {
            vRandom = random;
            
            // positions are 0->1, so make -1->1
            vec3 pos = position * 2.0 - 1.0;
            
            // Scale towards camera to be more interesting
            pos.z *= 10.0;
            
            // modelMatrix is one of the automatically attached uniforms when using the Mesh class
            vec4 mPos = modelMatrix * vec4(pos, 1.0);

            // add some movement in world space
            float t = uTime * 0.6;
            mPos.x += sin(t * random.z + 6.28 * random.w) * mix(0.1, 1.5, random.x);
            mPos.y += sin(t * random.y + 6.28 * random.x) * mix(0.1, 1.5, random.w);
            mPos.z += sin(t * random.w + 6.28 * random.y) * mix(0.1, 1.5, random.z);
            
            // get the model view position so that we can scale the points off into the distance
            vec4 mvPos = viewMatrix * mPos;
            gl_PointSize = 300.0 / length(mvPos.xyz) * (random.x + 0.1);
            gl_Position = projectionMatrix * mvPos;
        }
    `;

      const fragment = /* glsl */ `
          precision highp float;
          precision highp int;

          uniform float uTime;

          varying vec4 vRandom;

          void main() {
              vec2 uv = gl_PointCoord.xy;
              
              float circle = smoothstep(0.5, 0.4, length(uv - 0.5)) * 0.8;
              
              gl_FragColor.rgb = 0.8 + 0.2 * sin(uv.yxx + uTime + vRandom.y * 6.28) + vec3(0.1, 0.0, 0.3);
              gl_FragColor.a = circle;
          }
      `;
      //  导入场景 
      const {Scene} = spritejs;
      //  导入3d网格
      const {Mesh3d} = spriteExtend3d;
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
          z: 15,  // z轴坐标
        },
      })
      //  常量
      const num = 100;
      //  声明坐标
      const position = new Float32Array(num * 3);
      //  声明随机点
      const random = new Float32Array(num * 4);

      //  循环生成随机数据
      for(let i = 0; i < num; i++) {
        position.set([Math.random(), Math.random(), Math.random()], i * 3);
        random.set([Math.random(), Math.random(), Math.random(), Math.random()], i * 4);
      }
      //  生成modal数据
      const model = {
        position,
        random,
      };
      //  创建一个Program对象
      const program = layer.createProgram({
        //  顶点
        vertex,
        //  片段
        fragment,
        //  声明uTime值
        uniforms: {
          uTime: {value: 0},
        },
        depthTest: false,
      });
      //  生成网格颗粒
      const particles = new Mesh3d(
        program, 
        {
          //  粒子模式
          mode: 'POINTS',
          //  model数据 
          model
        }
      );
      //  添加到图层
      layer.append(particles);
      //  图层自动更新
      layer.bindTime(program);
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