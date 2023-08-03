<template>
  <div class="about">
    <p>与2D不同的是，3D中的一切元素都是Group3d的派生类，因此我们也可以将元素直接append到任意一个3D元素上。</p>
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
  name: 'GroupViewMore',
  components: {
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      //  导入场景 
      const { Scene } = spritejs;
      //  导入球体,立方体
      const { Sphere, Cube } = spriteExtend3d;
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
          pos: [1, 7, 0], // 相机的位置
        },
      });
      //  控制摄像头朝向
      layer.camera.lookAt([0, 0, 0]);

      const vertex = /* glsl */ `
          precision highp float;
          precision highp int;

          attribute vec3 position;
          attribute vec3 normal;

          uniform mat4 modelViewMatrix;
          uniform mat4 projectionMatrix;
          uniform mat3 normalMatrix;

          varying vec3 vNormal;
          varying vec4 vMVPos;

          void main() {
              vNormal = normalize(normalMatrix * normal);

              vMVPos = modelViewMatrix * vec4(position, 1.0);
              gl_Position = projectionMatrix * vMVPos;
          }
      `;

      const fragment = /* glsl */ `
          precision highp float;
          precision highp int;

          varying vec3 vNormal;
          varying vec4 vMVPos;

          void main() {
              vec3 normal = normalize(vNormal);
              float lighting = dot(normal, normalize(vec3(-0.3, 0.8, 0.6)));
              vec3 color = vec3(1.0, 0.5, 0.2) * (1.0 - 0.5 * lighting) + vMVPos.xzy * 0.1;

              float dist = length(vMVPos);
              float fog = smoothstep(4.0, 10.0, dist);
              color = mix(color, vec3(1.0), fog);

              gl_FragColor.rgb = color;
              gl_FragColor.a = 1.0;
          }
      `;

      //  创建 Program
      const program = layer.createProgram({
        vertex,
        fragment,
      });

      //  球体
      const sphere = new Sphere(program, {
        radius: 0.15,
      });

      //  立方体
      const cube = new Cube(program, {
        width: 0.3,
        height: 0.3,
        depth: 0.3
      });

      //  图形数组
      const shapes = [cube];

      //  更改速度
      cube.speed = -0.5;
      //  添加到图层
      layer.append(cube);

      //  随机创建形状
      for (let i = 0; i < 50; i++) {
        const mesh = Math.random() > 0.5 ? cube : sphere;
        const shape = mesh.cloneNode();
        shape.attr({
          scale: Math.random() * 0.3 + 0.7,
          x: (Math.random() - 0.5) * 3,
          y: (Math.random() - 0.5) * 3,
          z: (Math.random() - 0.5) * 3,
        });
        shape.speed = (Math.random() - 0.5) * 0.7;

        // Attach them to a random, previously created shape
        const parent = shapes[Math.floor(Math.random() * shapes.length)];
        parent.append(shape);
        shapes.push(shape);
      }

      //  添加变化
      layer.tick((t) => {
        console.log({t})
        shapes.forEach((shape) => {
          shape.attributes.rotateY += 2 * shape.speed;
          shape.attributes.rotateX += 1.5 * shape.speed;
          shape.attributes.rotateZ += shape.speed;
        });
      });

      //  增加控制
      layer.setOrbit();
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