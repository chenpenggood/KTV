<template>
    <div class="allorders">
        <Row style="padding:20px 0 20px;">
            <el-col :span="10">
                <el-date-picker
                v-model="selectDate"
                size="small"
                type="datetimerange"
                :picker-options="pickerOptions"
                range-separator="至"
                start-placeholder="开始日期"
                format="yyyy/MM/dd HH:mm:ss"
                end-placeholder="结束日期"
                align="right">
                </el-date-picker>
                <el-button type="primary" size="small" v-if="selectDate" style="margin-left:10px;vertical-align: top;" @click="searchOrder('time')">查询</el-button>
            </el-col>
            <el-col :span="6">
                <el-input class="user-email" size="small" placeholder="请输入账号" prefix-icon="el-icon-search" clearable v-model="user_email"></el-input>
                <el-button type="primary" size="small" style="margin-left:10px;vertical-align: top;" @click="searchOrder('userEmail')">查询</el-button>
            </el-col>
            <el-col :span="4" style="text-align:right;vertical-align: top;float:right;">
                <el-button type="primary" size="small" @click="getAllOrders">所有订单</el-button>
            </el-col>
        </Row>
        <!-- 订单数据表 -->
        <div style="width:100%;background-color:#f40;">
            <el-table
                :data="allorders"
                class="order-table"
                style="width: 100%"
                border>
                <el-table-column
                    type="index"
                    label="序号"
                    align="center"
                    width="60">
                </el-table-column>
                <el-table-column
                    prop="order_id"
                    label="订单编号"
                    align="center"
                    width="250">
                </el-table-column>
                <el-table-column
                    label="账号"
                    prop="account"
                    align="center"
                    width="200">
                </el-table-column>
                <el-table-column
                    prop="publicpwd"
                    label="明文密码"
                    align="center"
                    width="150">
                </el-table-column>
                <el-table-column
                    prop="money"
                    align="center"
                    width="100"
                    label="金额">
                </el-table-column>
                <el-table-column
                    align="center"
                    prop="startTime"
                    width="200"
                    label="开始时间">
                </el-table-column>
                <el-table-column
                    align="center"
                    prop="endTime"
                    width="200"
                    label="结束时间">
                </el-table-column>
                <el-table-column
                    align="center"
                    prop="password"
                    width="300"
                    label="密文">
                </el-table-column>
                <el-table-column
                    prop="operation"
                    align='center'
                    label="操作"
                    fixed="right"
                    width="140">
                    <template slot-scope='scope'>
                        <el-button type="danger" icon='el-icon-delete' size="small" circle @click='onDeleteSong(scope.row,scope.$index)'></el-button>
                        <el-button type="primary" icon="el-icon-edit" size="small" circle @click="editTime(scope.row, scope.$index)"></el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <!-- 编辑弹窗 -->
        <el-button :plain="true" @click="notEdit">警告</el-button>
        <el-dialog title="编辑订单(30RMB=1H)" :visible.sync="editOrderFlash" center width="55%" @close="closeEdit()" :destroy-on-close="true">
            <el-form :model="order_detailed" :rules="rule" ref="edit_order">
                <el-form-item label="账号" :label-width="editOrderPop">
                <el-input v-model="order_detailed.account" autocomplete="off" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="文明密码" :label-width="editOrderPop">
                <el-input v-model="order_detailed.publicpwd" autocomplete="off" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="总金额" :label-width="editOrderPop">
                <el-input v-model="newEdit.money" autocomplete="off" :disabled="true" style="color:#000!important;"></el-input>
                </el-form-item>
                <el-form-item label="续费" :label-width="editOrderPop" prop="addMoney">
                <el-input type="text" v-model="order_detailed.addMoney" autocomplete="off" placeholder="30元兑换1小时!"></el-input>
                </el-form-item>
                <el-form-item label="开始时间" :label-width="editOrderPop">
                <el-input v-model="order_detailed.startTime" autocomplete="off" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="结束时间" :label-width="editOrderPop">
                <el-input v-model="newEdit.endTime" autocomplete="off" :disabled="true" style="color:#fff;"></el-input>
                </el-form-item>
                <el-form-item></el-form-item>
                <el-form-item>
                    <el-button @click="closeEdit()">取 消</el-button>
                    <el-button type="primary" @click="submitForm()">确 定</el-button>
                </el-form-item>
            </el-form>
            <!-- <div slot="footer" class="dialog-footer">
                <el-button @click="editOrderFlash = false">取 消</el-button>
                <el-button type="primary" @click="comfirEdit()">确 定</el-button>
            </div> -->
            </el-dialog>

        <!-- 分页 -->
        <Row>
            <Col span="24" style="text-align:right;padding:10px;">
                <el-pagination
                    v-if='paginations.total > 0'
                    :page-sizes="paginations.page_sizes"
                    :page-size="paginations.page_size"
                    :layout="paginations.layout"
                    :total="paginations.total"
                    :current-page.sync='paginations.page_index'
                    @current-change='handleCurrentChange'
                    @size-change='handleSizeChange'>
                </el-pagination>
            </Col>
        </Row>

        <!-- 验证密码 -->
        <Modal draggable v-model="isCheckPassword" :mask-closable="false" width="360">
            <p slot="header" style="color:#f60;">
                <Icon type="ios-information-circle"></Icon>
                <span>验证密码</span>
            </p>
            <div style="text-align:left;font-size:14px;">
                <Row>
                    <Col span="18">
                        <Input v-model="inputPassword" type="password" icon="ios-key" @keyup.enter.native="checkPwd()"  placeholder="请输入管理员密码..." style="width: 200px" />
                    </Col>
                    <Col span="6">
                        <Button type="primary" size="large" @click="checkPwd()">验证</Button>
                    </Col>
                </Row>
            </div>
            <div slot="footer">
            </div>
        </Modal>
    </div>
</template>
<script>
import wsmLoading from "@/plugins/wsmLoading"
import "@/plugins/Date"
export default {
    name:"allorders",
    data(){
        const validNum = (rule, value, callback) => {
            const flag = /^[1-9]\d*$/.test(value.trim());
            if(!flag){
                value = 0;
                this.changeOrder(value);
                return callback(new Error('请输入金额！'));
            }else{
                this.changeOrder(value);
                return callback()
            }
        }
        return{
            user_email:"",
            isCheckPassword:false,
            editOrderFlash: false,
            saveOriginData: {},
            newEdit:{
                money:"",
                endTime: ""
            },
            order_detailed: {},
            editOrderPop: '80px',
            rule: {
                addMoney: [
                    {required:true,min:1,max:12,message:"金额不能为空！",trigger: "blur"},
                    {validator: validNum,message:'请输入正整金额！'}
                ]
            },
            inputPassword:'',
            noChangeData:[],
            selectDate:"",
            allorders:[],
            allTableData:[],
            paginations: {  // 分页属性
                page_index: 1, // 当前位于哪页
                total: 0, // 总数
                page_size: 8, // 1页显示多少条
                page_sizes: [8, 15, 20, 25], //每页显示多少条
                layout: "total, sizes, prev, pager, next, jumper" // 翻页属性
            },
            pickerOptions: {
                shortcuts: [{
                    text: '最近一周',
                    onClick(picker) {
                    const end = new Date();
                    const start = new Date();
                    start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
                    picker.$emit('pick', [start, end]);
                    }
                }, {
                    text: '最近一个月',
                    onClick(picker) {
                    const end = new Date();
                    const start = new Date();
                    start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
                    picker.$emit('pick', [start, end]);
                    }
                }, {
                    text: '最近三个月',
                    onClick(picker) {
                    const end = new Date();
                    const start = new Date();
                    start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
                    picker.$emit('pick', [start, end]);
                    }
                }]
            },
            delRow:""
        }
    },
    created(){
        this.getAllOrders();
    },
    methods:{
        checkPwd(){
            if(this.inputPassword.trim().length){
                this.isCheckPassword = false;
                wsmLoading.start("正在验证密码,请稍候...");
                setTimeout(() => {
                    this.$axios.post("http://localhost:8633/api/admin/islegal",{
                        password:this.inputPassword,
                        email:this.$store.getters.adminInfo.email
                    }).then(res => {
                        if(res.data == true){
                            this.$axios.post("http://localhost:8633/api/admin/orders/del", this.delRow)
                                    .then(res => {
                                        this.$nextTick(() => {
                                            this.getAllOrders();
                                            wsmLoading.end();
                                            this.$Message.success(res.data.result);
                                        })
                                    })
                        }
                    }).catch(() => this.isCheckPassword = true)
                }, 300)
            }else{
                this.isCheckPassword = false;
                this.$Modal.warning({
                    title:"提示",
                    content:"密码不能为空,请输入管理员口令...",
                    onOk:()=>{
                        this.isCheckPassword = true;
                    }
                })
            }
        },
        getAllOrders(){
            this.selectDate = '';
            this.$axios.get("http://localhost:8633/api/admin/orders/all")
                    .then(res => {
                        this.noChangeData = res.data;
                        this.allorders = res.data;
                        this.allTableData = res.data;
                        this.setPaginations();
                    })
        },
        onDeleteSong(order){
            this.$confirm("你确定要删除这条订单信息吗?不可恢复是否继续?", '提示', {
                confirmButtonText:"确定",
                cancleTextButtonText:"取消",
                type:"warning"
            }).then(() => {
                this.isCheckPassword = true;
                this.delRow = order;
            }).catch(() => console.log("cancle"))
        },
        // 订单过期不能编辑
        notEdit(){
            this.$message({
                message: '订单已经使用结束，不可续费！',
                type: 'warning'
            })
        },
        // 编辑订单，续费时长
        editTime(rowData){
            const theOrderEndTime = new Date(rowData.endTime).getTime();
            const nowTime = new Date().getTime();
            if(theOrderEndTime >= nowTime){
                this.saveOriginData = rowData;
                this.order_detailed = rowData;
                this.newEdit.money = rowData.money;
                this.newEdit.endTime = rowData.endTime;
                this.editOrderFlash = true;
            }else{
                this.notEdit();
            }
        },
        // 提交订单修改
        submitForm(){
            var that = this;
            that.$refs['edit_order'].validate(valid => {
                if(valid){
                    that.editOrderFlash = false;
                    const newEditData = that.newEdit;
                    const newOrderData = that.order_detailed;
                    console.log(newOrderData, newEditData)
                    let editData = {};
                    for(var k in newOrderData){
                        if(newEditData[k]){
                            editData[k] = newEditData[k]
                        }else{
                            editData[k] = newOrderData[k]
                        }
                    }
                    // const editData = Object.assign(newOrderData, newEditData); // 有问题，改变了order_detailed值
                    
                    // console.log(editData)

                    that.$axios.post("http://localhost:8633/api/admin/orders/edit", editData)
                    .then(res => {
                        if(res.status == 200){
                            // that.order_detailed = res.data.orders; // 重新获取全部订单，没必要
                            that.getAllOrders();
                        }
                    }).catch(err => {
                        that.order_detailed = that.saveOriginData;
                        wsmLoading.start(res.data.result)
                    })
                }
            })
        },
        // 修改订单动态显示
        changeOrder(value){
            const oldMoney = Number(this.order_detailed.money);
            const oldEndTime = this.order_detailed.endTime;
            const newEndTime = value*(60*60*1000/30) + new Date(oldEndTime).getTime();
            this.newEdit.money = oldMoney + Number(value);
            this.newEdit.endTime = new Date(newEndTime).format("yyyy/MM/dd HH:mm:ss");
        },
        // 取消、关闭修改订单
        closeEdit(){
            setTimeout(() => {
                this.order_detailed.addMoney = "";
            },300);
            this.editOrderFlash = false;
        },
        searchOrder(data){
            var that = this;
            switch(data){
                case 'time':
                    const startTime = new Date(that.selectDate[0]).getTime();
                    const endTime = new Date(that.selectDate[1]).getTime();
                    const result = [];
                    that.noChangeData.forEach(item => {
                        const time = new Date(item.startTime).getTime();
                        if(time >= startTime && time <= endTime){
                            result.push(item)
                        }
                    })
                    that.findorders(result);
                    break;
                case 'userEmail':
                    const userEmail = that.user_email;
                    const res = [];
                    that.noChangeData.forEach(item => {
                        const account = item.account;
                        if(account.indexOf(userEmail) > -1){
                            res.push(item)
                        }
                    });
                    that.findorders(res);
                    break;
            }
            
        },
        findorders(result){
            this.allorders = result;
            this.allTableData = result;
            this.setPaginations();
        },
        // 设置当前页
        handleCurrentChange(page) {
            // 获取当前页
            let sortnum = this.paginations.page_size * (page - 1);
            let table = this.allTableData.filter((item, index) => {
                return index >= sortnum;
            });
            // 设置默认分页数据
            this.allorders = table.filter((item, index) => {
                return index < this.paginations.page_size;
            });
            this.allorders = table.filter((item, index) => {
                return index < this.paginations.page_size;
            });
        },
        // 改变每页条数
        handleSizeChange(page_size) {
            // 切换size
            this.paginations.page_index = 1;
            this.paginations.page_size = page_size;
            this.allorders = this.allTableData.filter((item, index) => {
                return index < page_size;
            });
        },
        // 设置的分页
        setPaginations() {
            // 总页数
            this.paginations.total = this.allTableData.length;
            this.paginations.page_index = 1;
            this.paginations.page_size = 8;
            // 设置默认分页数据
            this.allorders = this.allTableData.filter((item, index) => {
                return index < this.paginations.page_size;
            });
        },
    }
}
</script>
<style lang="less" scoped>
.allorders{
    width: 100%;
    height: 100%;
    padding: 20px 60px;
    .user-email{width:74%;}
    .el-dialog{
        .el-form-item{width:50%;display: inline-block;
            &:nth-last-of-type(1){text-align: right;}
        }
    }
}
</style>