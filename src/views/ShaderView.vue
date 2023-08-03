<template>
  <div class="about">
    <a href="https://github.com/spritejs/sprite-extend-3d/blob/master/src/shader/index.js">查看着色器预定义的内容</a>
    <ul>
      <li>NORMAL 一个简单的根据法向量来确定表面颜色的着色器，主要用于测试</li>
      <li>NORMAL_GEOMETRY 支持光照、表面颜色的通用着色器，一般的几何元素可以使用它</li>
      <li>NORMAL_TEXTURE 支持光照、纹理的通用着色器，带纹理的几何元素可以使用它</li>
      <li>GEOMETRY_WITH_TEXTURE 支持光照、表面颜色和纹理的通用着色器，半透明纹理的几何元素可以使用它</li>
      <li>GEOMETRY_WITH_SHADOW 支持光照、表面颜色和阴影的通用着色器</li>
      <li>TEXTURE_WITH_SHADOW 支持光照、纹理和阴影的通用着色器</li>
      <li>GEOMETRY_WITH_TEXTURE_AND_SHADOW 支持光照、表面颜色、纹理和阴影的通用着色器</li>
      <li>TEXTURE_CUBE 支持立方体纹理的通用着色器</li>
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
  name: 'ShaderView',
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
      const {Cube} = spriteExtend3d;
      //  渲染容器
      const container = document.getElementById('stage');
      //  实例化场景
      const scene = new Scene({
        container,
      });
      //  图层使用3D渲染
      const layer = scene.layer3d('fglayer', {
        camera: {
          fov: 35, // 相机的视野
          pos: [3, 3, 5], // 相机的位置
        },
      });

      const vertex = `
        precision highp float;
        precision highp int;

        attribute vec3 position;
        attribute vec3 normal;

        uniform mat4 modelViewMatrix;
        uniform mat4 projectionMatrix;

        varying vec3 vNormal;

        void main() {
            vNormal = normalize(normal);
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
        `;

        const fragment = `
          precision highp float;
          precision highp int;

          varying vec3 vNormal;

          highp float noise(vec3 p) {
            vec3 i = floor(p);
            vec4 a = dot(i, vec3(1.0, 57.0, 21.0)) + vec4(0.0, 57.0, 21.0, 78.0);
            vec3 f = cos((p - i) * acos(-1.0)) * (-0.5) + 0.5;
            a = mix(sin(cos(a) * a), sin(cos(1.0 + a) * (1.0 + a)), f.x);
            a.xy = mix(a.xz, a.yw, f.y);
            return mix(a.x, a.y, f.z);
          }

          //  Function from Iñigo Quiles
          //  https://www.shadertoy.com/view/MsS3Wc
          vec3 hsb2rgb(vec3 c){
            vec3 rgb = clamp(abs(mod(c.x*6.0+vec3(0.0,4.0,2.0), 6.0)-3.0)-1.0, 0.0, 1.0);
            rgb = rgb * rgb * (3.0 - 2.0 * rgb);
            return c.z * mix(vec3(1.0), rgb, c.y);
          }

          uniform float uTime;

          void main() {
            vec3 normal = vNormal * uTime;
            gl_FragColor.rgb = hsb2rgb(vec3(noise(normal), 0.5, 0.5));
            gl_FragColor.a = 1.0;
          }
        `;
        // 创建Program对象
        const program = layer.createProgram({
          //  顶点
          vertex,
          //  片段
          fragment,
          //  无切面
          cullFace: null,
        }, {
          //  声明uTime值
          uniforms: {
            uTime: {value: 0},
          },
        });
        //  立方体使用program
        const cube = new Cube(program);
        //  添加到图层
        layer.append(cube);
        //  设置自动更新 program的 uniforms.uTime
        layer.bindTime(program, {playbackRate: 0.2});
        layer.setOrbit(); // 开启旋转控制
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