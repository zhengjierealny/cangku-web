<template>

    <div class="loginBody">


        <div class="loginDiv">
            <div class="title">
                <h1 style="text-align: center"> 图书仓库管理系统</h1>
            </div>
            <div class="login-content">
                <h2 cLass="login-title">用户登录</h2>
                <el-form :model="loginForm" label-width="100px" :rules="rules"
                         ref="loginForm">
                    <el-form-item label="账号" prop="no">
                        <el-input style="width: 80%" type="text" v-model="loginForm.no"
                                  autocomplete="off" size="small"></el-input>
                    </eL-form-item>
                    <el-form-item label="密码" prop="password">
                        <el-input style="width: 80%" type="password" v-model="loginForm.password"
                              show-password autocomplete="off" size="small" 
                              @keyup.enter.native="confirm"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="confirm"
                                   style="margin-left: 15%" :disabled="confirm_disabled">确 定</el-button>
                        <el-button size="small" type="success" @click="add()">注册</el-button>
                        <el-button style="margin-left: 10%" size="small" type="success" @click="fotget()">忘记密码</el-button>
                    </el-form-item>
                </el-form>
                <el-dialog
                        title="用户注册"
                        :visible.sync="centerDialogVisible"
                        width="30%"
                        center>

                    <el-form ref="form" :rules="rules" :model="form" label-width="80px">
                        <el-form-item label="账号" prop="no">
                            <el-col :span="20">
                                <el-input v-model="form.no"></el-input>
                            </el-col>
                        </el-form-item>
                        <el-form-item label="名字" prop="name">
                            <el-col :span="20">
                                <el-input v-model="form.name"></el-input>
                            </el-col>
                        </el-form-item>
                        <el-form-item label="密码" prop="password">
                            <el-col :span="20">
                                <el-input v-model="form.password"></el-input>
                            </el-col>
                        </el-form-item>
<!--                        <el-form-item label="重复密码" prop="password">-->
<!--                            <el-col :span="20">-->
<!--                                <el-input v-model="form.comfimPassword"></el-input>-->
<!--                            </el-col>-->
<!--                        </el-form-item>-->

                    </el-form>
                    <span slot="footer" class="dialog-footer">
                    <el-button @click="centerDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="save" :disabled="confirm_disabled">确 定</el-button>
                    </span>
                </el-dialog>
                <el-dialog
                        title="找回密码"
                        :visible.sync="passwordDialogVisible"
                        width="30%"
                        center>

                    <el-form ref="form" :rules="rules2" :model="form1" label-width="80px">
                        <el-form-item label="账号" prop="no">
                            <el-col :span="20">
                                <el-input v-model="form1.no"></el-input>
                            </el-col>
                        </el-form-item>
                        <el-form-item label="名字" prop="name">
                            <el-col :span="20">
                                <el-input v-model="form1.name"></el-input>
                            </el-col>
                        </el-form-item>
                        <el-form-item label="新密码" prop="password">
                            <el-col :span="20">
                                <el-input v-model="form1.password"></el-input>
                            </el-col>
                        </el-form-item>
                        <!--                        <el-form-item label="重复密码" prop="password">-->
                        <!--                            <el-col :span="20">-->
                        <!--                                <el-input v-model="form.comfimPassword"></el-input>-->
                        <!--                            </el-col>-->
                        <!--                        </el-form-item>-->

                    </el-form>
                    <span slot="footer" class="dialog-footer">
                    <el-button @click="passwordDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="findpwd()" :disabled="confirm_disabled">确 定</el-button>
                    </span>
                </el-dialog>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Login",
        data(){
            let checkAge = (rule, value, callback) => {
                if(value>150){
                    callback(new Error('年龄输入过大'));
                }else{
                    callback();
                }
            };
            let checkDuplicate =(rule,value,callback)=>{
                if(this.form.id){
                    return callback();
                }
                this.$axios.get(this.$httpUrl+"/user/findByNo?no="+this.form.no).then(res=>res.data).then(res=>{
                    if(res.code!=200){

                        callback()
                    }else{
                        callback(new Error('账号已经存在'));
                    }
                });

            };
            let checkDuplicate2 =(rule,value,callback)=>{
                if(this.form.id){
                    return callback();
                }
                this.$axios.get(this.$httpUrl+"/user/findByNo?no="+this.form1.no).then(res=>res.data).then(res=>{
                    if(res.code!=200){

                        callback(new Error('账号不存在'));
                    }else{

                        callback()
                    }
                });

            };
            let checkName=(rule,value,callback)=>{
                this.$axios.get(this.$httpUrl+"/user/findByName?name="+this.form1.name).then(res=>res.data).then(res=>{
                    if(res.code!=200){

                        callback(new Error('用户名不存在'));
                    }else{

                        callback()
                    }
                });
            };
            return{
                // name:'',
                // sex:'',
                // sexs:[
                //     {
                //         value: '1',
                //         label: '男'
                //     }, {
                //         value: '0',
                //         label: '女'
                //     }
                // ],
                form:{
                    id:'',
                    no:'',
                    name:'',
                    password:'',
                    comfimPassword:'',
                    // age:'',
                    // phone:'',
                    // sex:'0',
                    roleId:'2'
                },
                form1:{
                    id:'',
                    no:'',
                    name:'',
                    password:'',
                    comfimPassword:'',
                    // age:'',
                    // phone:'',
                    // sex:'0',
                    roleId:''
                },
                confirm_disabled:false,
                centerDialogVisible:false,
                passwordDialogVisible:false,
                loginForm:{
                    no:'',
                    password:''
                },
                rules:{
                    no:[
                        {required:true,message:'请输入账号',trigger:'blur'}
                    ],
                    password:[
                        {required:true,message:'请输入密码',trigger:'blur'}
                    ],
                    no: [
                        {required: true, message: '请输入账号', trigger: 'blur'},
                        {min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur'},
                        {validator:checkDuplicate,trigger: 'blur'}
                    ],
                    name: [
                        {required: true, message: '请输入名字', trigger: 'blur'}
                    ],
                    password: [
                        {required: true, message: '请输入密码', trigger: 'blur'},
                        {min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur'}
                    ],
                    age: [
                        {required: true, message: '请输入年龄', trigger: 'blur'},
                        {min: 1, max: 3, message: '长度在 1 到 3 个位', trigger: 'blur'},
                        {pattern: /^([1-9][0-9]*){1,3}$/,message: '年龄必须为正整数字',trigger: "blur"},
                        {validator:checkAge,trigger: 'blur'}
                    ],
                    phone: [
                        {required: true,message: "手机号不能为空",trigger: "blur"},
                        {pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/, message: "请输入正确的手机号码", trigger: "blur"}
                    ]
                },
                rules2:{
                    no: [
                        {required: true, message: '请输入账号', trigger: 'blur'},
                        {min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur'},
                        {validator:checkDuplicate2,trigger: 'blur'}
                    ],
                    name: [
                        {required: true, message: '请输入名字', trigger: 'blur'},
                        {validator:checkName,trigger: 'blur'}
                    ],
                    password: [
                        {required: true, message: '请输入密码', trigger: 'blur'},
                        {min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur'}
                    ],

                }
            }
        },
        methods:{
            resetForm() {
                this.$refs.form.resetFields();
            },
            add(){

                this.centerDialogVisible = true
                this.$nextTick(()=>{
                    this.resetForm()
                })

            },
            save(){
                // if(this.form.password==this.form.comfimPassword){
                    this.$axios.post(this.$httpUrl+'/user/save',this.form).then(res=>res.data).then(res=>{
                        console.log(res)
                        if(res.code==200){

                            this.$message({
                                message: '注册成功，请登录！',
                                type: 'success'
                            });
                            this.centerDialogVisible = false
                            this. resetForm()


                        }else{
                            this.$message({
                                message: '注册失败！',
                                type: 'error'
                            });
                        }

                    })
                // }else{
                //     this.$message({
                //         message: '两次输入密码不一致',
                //         type: 'error'
                //     });
                // }


            },
            fotget(){
                this.passwordDialogVisible = true
                this.$nextTick(()=>{
                    this.resetForm()
                })
            },
            findpwd(){
                if(this.form1.no.trim().length > 0&&this.form1.name.trim().length > 0&&this.form1.password.trim().length
                    > 0){
                    this.$axios.post(this.$httpUrl+'/user/update',this.form1).then(res=>res.data).then(res=>{
                        console.log(res)
                        if(res.code==200){

                            this.$message({
                                message: '重置密码成功，请登录！',
                                type: 'success'
                            });
                            this.passwordDialogVisible = false
                            this. resetForm()


                        }else{
                            this.$message({
                                message: '重置密码失败！',
                                type: 'error'
                            });
                        }

                    })
                }else {
                    alert('请输入所有信息');
                }

            },
            confirm(){
                this.confirm_disabled=true;
                this.$refs.loginForm.validate((valid)=>{
                    if(valid){
                        this.$axios.post(this.$httpUrl+'/user/login',this.loginForm)
                        .then(res=>res.data).then(res=>{
                            console.log(res)
                            if(res.code==200){
                                sessionStorage.setItem("CurUser",JSON.stringify(res.data.user))
                                console.log(res.data.menu)
                                this.$store.commit("setMenu",res.data.menu)
                                this.$router.replace('/Index');
                            }else{
                                this.confirm_disabled=false;
                                alert('校验失败，用户名密码错误')
                            }
                        });
                    }else {
                        this.confirm_disabled=false;
                        console.log('校验失败')
                        return false
                    }
                });
            }
        }
    }
</script>

<style scoped>
    .title{
        margin-top: -20%;
    }
    .loginBody{
        position: absolute;
        width: 100%;
        height: 100%;
        background-color: #B3C0D1;
    }
    .loginDiv{
        position: absolute;
        top: 50%;
        left: 50%;
        margin-top: -160px;
        margin-left: -250px;
        width: 450px;
        height: 300px;
        background: #fff;
        border-radius: 5%;
    }
    .login-title{
        margin: 20px 0;
        text-align: center;

    }
    .login-content{
        width: 400px;
        height: 220px;
        position: absolute;
        top: 25px;
        left: 25px;
    }
</style>