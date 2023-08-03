<template>
  <div class="about">
    <p>渲染模型</p>
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
  name: 'RenderModel',
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
        width: 600,
        height: 600,
        // 显示比率
        // displayRatio: 2,
      });
      //  图层使用3D渲染
      const layer = scene.layer3d('fglayer', {
        camera: {
          fov: 35, // 相机的视野
          // pos: [-2, 2, 2], // 相机位置
          z: 8, // z轴位置
        },
        //  点光源
        // directionalLight: [0.5, 1.0, -0.3],
        // directionalLightColor: [1, 1, 1, 0.15]
      })
      //  创建一个Program对象,使用简单的根据法向量来确定表面颜色的着色器
      const program = layer.createProgram({
        ...shaders.NORMAL,
        //  无切面
        cullFace: null,
      });
      
      //  生成相关数据
      function normalize(v) {
        const len = Math.hypot(...v);
        return [v[0] / len, v[1] / len, v[2] / len];
      }

      // 两个向量的叉积就是这个向量的法向量
      function getNormal(a, b, c) {
        const ab = [a[0] - b[0], a[1] - b[1], a[2] - b[2]];
        const bc = [b[0] - c[0], b[1] - c[1], b[2] - c[2]];

        return normalize([
          ab[1] * bc[2] - ab[2] * bc[1],
          ab[0] * bc[2] - ab[2] * bc[0],
          ab[0] * bc[1] - ab[1] * bc[0],
        ]);
      }

      const p = 1 / Math.sqrt(2);

      const position = [
        -1, 0, -p,
        1, 0, -p,
        0, 1, p,
        -1, 0, -p,
        1, 0, -p,
        0, -1, p,
        1, 0, -p,
        0, -1, p,
        0, 1, p,
        -1, 0, -p,
        0, 1, p,
        0, -1, p,
      ];

      const normal = [];

      for(let i = 0; i < position.length; i += 9) {
        const a = [position[i], position[i + 1], position[i + 2]],
          b = [position[i + 3], position[i + 4], position[i + 5]],
          c = [position[i + 6], position[i + 7], position[i + 8]];

        const norm = getNormal(a, b, c);
        normal.push(...norm, ...norm, ...norm);
      }

      //  获取模型
      const model = {
        position,
        normal,
      };

      //  结合program和model生成3D网格
      const sprite = new Mesh3d(program, {
        model,
      });
      //  添加到图层
      layer.append(sprite);

      //  添加旋转动画
      sprite.animate([
        {rotateY: 0},
        {rotateY: 360},
      ], {
        duration: 7000,
        iterations: Infinity,
      });

      sprite.animate([
        {rotateZ: 0},
        {rotateZ: 360},
      ], {
        duration: 17000,
        iterations: Infinity,
      });

      sprite.animate([
        {rotateX: 0},
        {rotateX: -360},
      ], {
        duration: 11000,
        iterations: Infinity,
      });

      //  图层添加控制
      layer.setOrbit();
    }
  }
}
</script>
<style scoped>
#stage{
  width:600px;
  height:600px;
  background-color:#FFF;
  margin:0 auto;
  border:2px solid skyblue;
}
</style>