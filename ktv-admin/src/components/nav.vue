<template>
    <div class="wsmnav">
        <Row>
            <Menu mode="horizontal" :theme="theme">
                <el-col :span="4">
                    <div class="nav-logo" @click="goIndex">KTV Admin</div>
                </el-col>
                <el-col :span="18">
                    <div class="current-time">日期：{{nowTime}}</div>
                </el-col>
                <el-col :span="2">
                    <el-menu class="el-menu-demo" mode="horizontal" @select="handleSelect" background-color="#515a6e" text-color="#fff" active-text-color="#fff">
                        <el-submenu index="1" trigger="click">
                            <template slot="title">
                                <Avatar :src="avatar" />
                                <!-- <Icon type="ios-arrow-down" style="margin-left:5px;color:#bbb;"></Icon> -->
                            </template>
                            <el-menu-item index="1-1" @click.native="goIndex">你好,{{adminInfo.username}}</el-menu-item>
                            <el-menu-item index="1-2" @click.native="goManageMusic">歌曲管理</el-menu-item>
                            <el-menu-item index="1-3" @click.native="goAdminLikes">推荐歌曲</el-menu-item>
                            <el-menu-item index="1-4" @click.native="goUser_service">开机服务</el-menu-item>
                            <el-menu-item index="1-5" @click.native="allorders">订单查询</el-menu-item>
                            <el-menu-item index="1-6" @click.native="logout">退出登录</el-menu-item>
                        </el-submenu>
                    </el-menu>
                </el-col>
            </Menu>
        </Row>
    </div>
</template>
<script>
import "@/plugins/Date"
export default {
    name:"wsmnav",
    data(){
        return{
            theme:"dark",
            avatar:require("../assets/avatar/admin.jpg"),
            adminInfo:"",
            nowTime:"",
        }
    },
    created(){
        this.getAdminInfo();
        this.currentTime();
    },
    methods:{
        handleSelect(key, keyPath) {
            // console.log(key, keyPath);
        },
        goIndex(){
            this.$router.push("/admin/index");
        },
        allorders(){
            this.$router.push("/admin/allorders");
        },
        goUser_service(){
            this.$router.push("/admin/user_service");
        },
        goManageMusic(){
            this.$router.push("/admin/manage/music");
        },
        goAdminLikes(){
            this.$router.push("/admin/music/likes");
        },
        // 刷新时间
        currentTime(){
            setInterval(() => {
                this.nowTime = new Date().format("yyyy/MM/dd HH:mm:ss")
            }, 1000)
        },
        // 退出
        logout(){
            this.$confirm('确定退出吗？每一片贫瘠的土地都需要坚定的挖掘者！', '退出提示', {
                confirmButtonText: '退出',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$store.dispatch("clearAdminAuthorization");
                this.$router.push("/login");
            }).catch(() => {})
            
        },
        // 获取管理员信息
        getAdminInfo(){
            this.adminInfo = this.$store.getters.adminInfo;
        }
    }
}
</script>
<style lang="less" scoped>
.wsmnav{width:100%;
    .opera-btn{
        position: relative;
    }
    .current-time{
        font-size: 16px;
        color: rgb(173, 166, 166);
        text-align: right;
    }
    .nav-logo{
        height: 60px;
        line-height: 60px;
        color: rgb(192, 190, 190);
        font-size: 30px;
        width: 100%;cursor: pointer;
        text-align: center;
        font-family: "隶书";
    }
}
.el-menu{padding-left:10px;}
.el-menu--popup-bottom-start{margin-top:45px;}
</style>