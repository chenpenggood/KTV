<template>
    <div class="login">
        <div class="login-title">
            后台管理系统
        </div>
        <div class="form-box">
             <el-tabs class="sign-login" v-model="activeName" @tab-click="handleClick">
                <el-tab-pane label="Log in" name="log_in">
                    <el-form class="login-form" label-position="top" size="small" :inline-message="inlinemessage" :model="form" :rules="rule" ref="login_register" label-width="100px">
                        <el-form-item label="Email address" prop="email">
                            <el-input type="text" v-model="form.email" placeholder="请输入邮箱"></el-input>
                        </el-form-item>
                        <el-form-item label="Password" prop="password">
                            <el-input type="password" @keypress.enter.native="submitForm('form')" show-password v-model="form.password" placeholder="请输入密码"></el-input>
                        </el-form-item>
                        <el-form-item label="Security Code" prop="inputCaptcha">
                            <div class="yzm">
                                <el-input style="width:150px;" v-model="inputCaptcha" placeholder="验证码"></el-input>
                                <img width="80" style="background:#EEE9E9;margin-left:30px;" ref="captcha" height="32" src="http://localhost:8633/api/safecode" @click="refreshCaptcha">
                            </div>
                        </el-form-item>
                        <el-form-item>
                            <el-button class="login-btn" type="primary" @click="submitForm('form')">Log in</el-button>
                        </el-form-item>
                    </el-form>
                </el-tab-pane>
                <el-tab-pane label="Sign up" name="sign_up">
                    <el-form class="register-form" label-position="top" size="small" :inline-message="inlinemessage" :model="form" :rules="rule" ref="login_register" label-width="100px">
                        <el-form-item label="User name" prop="username">
                            <el-input type="text" v-model="form.username" placeholder="请输入昵称"></el-input>
                        </el-form-item>
                        <el-form-item label="Email address" prop="email">
                            <el-input type="text" v-model="form.email" placeholder="请输入邮箱"></el-input>
                        </el-form-item>
                        <el-form-item label="Password" prop="password">
                            <el-input type="password" @keypress.enter.native="submitForm('registerForm')" show-password v-model="form.password" placeholder="请输入密码"></el-input>
                        </el-form-item>
                        <el-form-item>
                            <el-button class="register-btn" type="primary" @click="submitForm('registerForm')">Sign up</el-button>
                        </el-form-item>
                    </el-form>
                </el-tab-pane>
            </el-tabs>    
        </div>
    </div>
</template>
<script>
import wsmLoading from "@/plugins/wsmLoading"
import jwt_decode from 'jwt-decode';
export default {
    name:"login",
    data(){
        const validateCaptcha = (rule, value, callback) => {
            if(!this.inputCaptcha.trim().length){
                callback(new Error('请输入验证码~'));
            }else{
                callback();
            }
        }
        return{
            inline:true,
            inlinemessage:false,
            activeName: 'log_in',
            form:{
                username: "",
                email:"",
                password:""
            },
            rule:{
                username: [
                    {min:3,max:12,message:"请输入正确的昵称",trigger:"blur"}
                ],
                email:[
                    {required:true,message:"邮箱不能为空",trigger:"blur"},
                    {type:"email",message:"请输入正确的邮箱",trigger:"blur"},
                ],
                password:[
                    {required:true,message:"密码不能为空",trigger:"blur"},
                    {min:6,max:20,message:"密码长度在6-20之间",trigger:"blur"}
                ],
                inputCaptcha:[
                    {required:true, validator:validateCaptcha,trigger:"blur"}
                ]
            },
            inputCaptcha:""
        }
    },
    methods:{
        handleClick(tab, event) {
            // console.log(tab, event);
        },
        // 登录
        submitForm(formName){
            this.$refs['login_register'].validate(valid => {
                if(valid){
                    if(formName == "form"){
                        wsmLoading.start("正在登录,请稍候...");
                        setTimeout(() => {
                            if(this.inputCaptcha == this.getCookie("captcha")){
                                this.login(this.form);
                            }else{
                                wsmLoading.end();
                                this.$Modal.confirm({
                                    title:"错误",
                                    content:"你输入的验证码有误,请重新输入验证码~",
                                    type:"error",
                                    onOk:()=>{
                                        // this.refreshCaptcha();
                                    }
                                })
                            }
                        }, 900);
                    }else if(formName == "registerForm"){
                        setTimeout(() => {
                            this.$axios.post('http://localhost:8633/api/admin/account/register', this.form)
                            .then(res => {
                                if(res.status == 200){
                                    this.login(this.form, 'register');
                                }
                            })
                        })
                    }
                }
            })
        },
        login(formData, type){
             this.$axios.post("http://localhost:8633/api/admin/account/login", formData)
                .then(res => {
                    if(res){
                        // 解析token
                        const {token} = res.data;
                        localStorage.setItem("adminToken", token);
                        const decoded = jwt_decode(token);
                        // console.log(decoded)
                        // 存储vuex中
                        this.$store.dispatch("setAdminInfo", decoded);
                        this.$store.dispatch("isAdminAuthorization", true);
                        if(type == 'login'){
                            this.$Message.success(`${decoded.username}登录成功`)
                        }else if(type == 'register'){
                            this.$Message.success(`${decoded.username}注册成功`)
                        }
                        wsmLoading.end();
                        this.$router.push("/");
                    }
                })
                .catch(error => {
                    console.error(error.response);
                    this.refreshCaptcha();
                })
        },
        // 获取验证码cookie
        getCookie(cname){
            var name = cname + "=";
            var ca = document.cookie.split(';');
            for(var i=0; i<ca.length; i++){
                var c = ca[i].trim();
                if (c.indexOf(name)==0) return c.substring(name.length,c.length);
            }
            return "";
        },
        // 刷新验证码
        refreshCaptcha(){
            this.$refs.captcha.src = "http://localhost:8633/api/safecode?d=" + Math.random();
        },
    }
}
</script>
<style lang="less" scoped>
.sign-login{
    width: 300px;
    height: 100%;
    margin: 0 auto;
}
.login{
    width: 100%;
    height: 100%;
    cursor: default;
    padding: 40px 0px;
    color:red;
    // background-image: url(../assets/image/login-bg.jpg);
    background-size: 100% 100%;
    display: flex;
    flex-direction: column;

    .login-title{
        width:100%;
        height: 150px;
        line-height: 150px;
        text-align: center;
        font-size: 40px;
        color: aliceblue;
        font-family: "隶书";
    }

    // 表单
    .form-box{
        width: 100%;
        // min-width: 500px;

        .login-form, .register-form{
            width: 300px;
            margin: 0px auto;
            background-color: rgba(255, 255, 255,.9);
            border: 1px solid #cdcdcd;
            padding: 10px 15px;
            border-radius: 5px;

            .login-btn,.register-btn{
                width: 100%;
                // background: linear-gradient(to bottom, rgba(47, 228, 89, 0.9),#26a744);
                font-weight: 600;
            }
            .login-btn:hover{
                background-color: rgb(94, 255, 132);
            }


            // 验证码区域
            .yzm{
                display:flex;
                align-content:center;
                input{
                    width: 160px;
                    height: 32px;
                    outline: none;
                    border: 1px solid #eee;
                    padding: 2px 15px;
                    border-radius: 5px;
                    font-size: 13px;
                }
                ::-webkit-input-placeholder{
                    color:#bbb;
                }
                img:hover{
                    cursor: pointer;
                }
            }

        }
    } 
}
</style>