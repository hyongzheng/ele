<template>
  <div class="index">
    <!-- <h1>主页</h1> -->
    <router-view></router-view>
    <TabBar :data="tabbarData"></TabBar>
  </div>
</template>

<script>
import TabBar from '../components/TabBar';
export default {
  name: "index",
  data() {
    return {
      tabbarData: [
        { title: "首页", icon: "home", path: "/home" },
        { title: "订单", icon: "file-text-o", path: "/order" },
        { title: "我的", icon: "user", path: "/me" }
      ]
    };
  },
  components: {
    TabBar
  }
};
</script>
<style>
.index{
  /* width: 100%; */
  height: calc(100% - 45px); /*这是什么奇淫技巧啊？*/
  /* background-color: white; */
}

</style>