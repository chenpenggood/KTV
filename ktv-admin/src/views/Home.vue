<template>
  <div class="home">
    <div class="nav">
        <wsmnav></wsmnav>
    </div>
    <div class="content">
        <router-view></router-view>
    </div>
  </div>
</template>

<script>
import wsmnav from "@/components/nav"
export default {
  name: 'home',
  components: {
      wsmnav,
  }
}
</script>
<style lang="less" scoped>
.home{
    min-width: 1200px;
    width: 100%;padding-top:60px;height:100%;
    .nav{
        width:100%;position:fixed;top:0;left:0;right:0;z-index:100;
    }

    .content{
        flex: 1;
        width: 100%;
        height: 100%;
        // overflow-y: scroll;
    }
}
</style>
