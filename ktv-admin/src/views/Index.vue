<template>
    <div class="Index">
        <div class="user-info clearfix">
            <div class="user-box fl">
                <div class="user-name">
                    <div class="block avatar"><el-avatar :size="50" :src="circleUrl"></el-avatar></div>
                    <span class="name">{{adminInfo.username}}</span>
                </div>
                <p class="user-email">用户邮箱：{{adminInfo.email}}</p>
                <p class="user-date">注册时间：{{adminInfo.date}}</p>
                <p class="autograph">签名：{{autograph}}</p>
            </div>
            <div class="ad fr">
                <el-image :src="adSrc" fill="contain" lazy alt="广告位" title="广告位">
                    <div slot="placeholder" class="image-slot">
                        加载中<span class="dot">...</span>
                    </div>
                    <div slot="error" class="image-slot">
                        <i class="el-icon-picture-outline"></i>
                    </div>    
                </el-image>
            </div>
        </div>
        <el-divider content-position="right">Me</el-divider>
        <el-card class="menu-box">
            <ul>
                <li><router-link to="manage/music">歌曲管理</router-link></li>
                <li><router-link to="music/likes">推荐歌曲</router-link></li>
                <li><router-link to="user_service">开机服务</router-link></li>
                <li><router-link to="allorders">订单查询</router-link></li>
            </ul>
        </el-card>
        <el-card class="calendar-box">
            <el-calendar v-model="calendarVal"></el-calendar>
        </el-card>
    </div>
</template>
<script>
export default {
    name:"Index",
    data(){
        return{
            circleUrl: "https://cube.elemecdn.com/3/7c/3ea6beec64369c2642b92c6726f1epng.png",
            adSrc: "https://fuss10.elemecdn.com/3/28/bbf893f792f03a54408b3b7a7ebf0jpeg.jpeg",
            autograph: "醉和金甲舞，雷鼓动山川,调箭又呼鹰，俱闻出世能。",
            calendarVal: new Date(),
            adminInfo: {}
        }
    },
    created(){
        this.getAdminInfo();
    },
    methods: {
        getAdminInfo(){
            this.adminInfo = this.$store.getters.adminInfo;
        }
    }
}
</script>
<style lang="less" scoped>
.Index{
    width: 100%;
    padding: 0 20px;
    // background-image: url(../assets/image/info-bg.jpg);
    background-size: 100%;
    .user-info{
        width:90%;font-size:14px;margin-bottom:50px;
        margin: 30px auto;
        .user-box{width:70%;}
        .user-name{
            .avatar{display:inline-block;vertical-align:middle;}
            .name{font-size:22px;margin-left:20px;}
        }
        .user-email{margin-top:20px;}
        .user-date{margin-top:5px;}
        .autograph{margin-top:50px;font-size:12px;color:#F56C6C;}
        .ad{width:30%;}
    }

    .menu-box{width:58%;vertical:middle;display:inline-block;
        ul{display:flex;justify-content:center;align-items:center;
            li{width:200px;height:60px;border-radius: 4px;background-color:#60839c;flex:1;margin:20px;
                a{color:#fff;display:block;width:100%;height:100%;line-height: 60px;text-align: center;font-size:16px;
                    &:hover{box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.3);}
                }
            }
        }
    }
    .calendar-box{width:40%;float:right;}
}
</style>