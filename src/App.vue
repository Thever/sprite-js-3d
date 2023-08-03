<template>
  <div id="app">
    <nav>
      <router-link to="/">Home</router-link> |
      <router-link to="/demo">Demo</router-link> | 
      <router-link to="/coordinatecamera">坐标与相机</router-link> | 
      <router-link to="/drawgeometry">绘制几何体</router-link> | 
      <router-link to="/drawmodel">绘制模型</router-link> | 
      <router-link to="/drawtexturemodel">绘制材质模型</router-link> | 
      <router-link to="/thelight">光照</router-link> | 
      <router-link to="/elementinteraction">与元素交互</router-link> | 
      <router-link to="/shaderview">着色器</router-link> | 
      <router-link to="/rendermodepoints">粒子模式</router-link> | 
      <router-link to="/rendermodeother">其他渲染模式</router-link> | 
      <router-link to="/rendermodel">渲染模型</router-link> | 
      <router-link to="/rendermodelmany">批量渲染模型</router-link> | 
      <router-link to="/sharemodel">共享模型</router-link> | 
      <router-link to="/textplane">文字平面</router-link> | 
      <router-link to="/groupview">3D分组</router-link> | 
      <router-link to="/groupviewmore">更多组合</router-link> | 
      <br>
      <router-link to="/orthographiccamera">正交相机</router-link> | 
      <router-link to="/shadowview">阴影</router-link> | 
      <router-link to="/rendertarget">渲染对象</router-link> | 
      <router-link to="/postprocessingchannel">后期处理通道</router-link> | 
      <router-link to="/gpu">通用GPU</router-link> | 
      <!-- <router-link to="/demo">Demo</router-link> | 
      <router-link to="/demo">Demo</router-link> |  -->
    </nav>
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}
</style>
