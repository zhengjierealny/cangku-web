<template>
    <div>
        <div style="margin-bottom: 5px">
            <el-input v-model="name" placeholder="请输入名字" suffix-icon="el-icon-search" style="width: 200px"
                @keyup.enter.native="loadPost"></el-input>
            <el-select v-model="sex" filterable placeholder="请选择性别" style="margin-left: 10px">
                <el-option
                        v-for="item in sexs"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                </el-option>
            </el-select>
            <el-button type="primary" style="margin-left: 10px" @click="loadPost">查询</el-button>
            <el-button type="success" @click="resetParam">清空</el-button>
            <el-button type="primary" style="margin-left: 10px" @click="add">新增</el-button>
        </div>
        <el-table :data="tableData"
            :header-cell-style="{backgroundColor:'#f2f5fc'}"
                  border
            >
            <el-table-column prop="id" label="ID" width="100">
            </el-table-column>
            <el-table-column prop="no" label="账号" width="160">
            </el-table-column>
            <el-table-column prop="name" label="姓名" width="120">
            </el-table-column>
            <el-table-column prop="age" label="年龄" width="100">
            </el-table-column>
            <el-table-column prop="sex" label="性别" width="100">
                <template slot-scope="scope">
                    <el-tag
                            :type="scope.row.sex === 1 ? 'primary' : 'success'"
                            disable-transitions>{{
                                scope.row.sex === 1 ? '男' : '女'}}</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="roleId" label="角色" width="140">
                <template slot-scope="scope">
                    <el-tag
                            :type="scope.row.roleId === 0 ? 'danger' :
                            (scope.row.roleId === 1 ? 'primary' : 'success')"
                            disable-transitions>{{
                                scope.row.roleId === 0 ? '超级管理员' :
                                (scope.row.roleId === 1 ? '管理员' : '用户')}}</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="phone" label="电话" width="160">
            </el-table-column>
            <el-table-column prop="operate" label="操作">
                <template slot-scope="scope">
                    <el-button size="small" type="success" @click="mod(scope.row)">编辑</el-button>
                    <el-popconfirm
                            title="确定删除吗？"
                            @confirm="del(scope.row.id)"
                    >
                        <el-button style="margin-left: 10px" slot="reference" size="small" type="danger"
                                   >删除
                        </el-button>
                    </el-popconfirm>

                </template>
            </el-table-column>
        </el-table>
        <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="pageNum"
                :page-sizes="[ 5, 10, 20,30]"
                :page-size="pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
        </el-pagination>
        <el-dialog
                title="提示"
                :visible.sync="centerDialogVisible"
                width="26%"
                :before-close="handleClose"
                center>
            <el-form ref="form" :rules="rules" :model="form" label-width="80px">

                <el-form-item label="账号" prop="no">
                    <el-col :span="20">
                    <el-input v-model="form.no"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-col :span="20">
                        <el-input v-model="form.password"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="姓名" prop="name">
                    <el-col :span="20">
                        <el-input v-model="form.name"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="年龄" prop="age">
                    <el-col :span="20">
                        <el-input v-model="form.age"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="性别" prop="sex">
                    <el-radio-group v-model="form.sex">
                        <el-radio label="1">男</el-radio>
                        <el-radio label="0">女</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="电话" prop="phone">
                    <el-col :span="20">
                        <el-input v-model="form.phone"></el-input>
                    </el-col>
                </el-form-item>


            </el-form>
            <span slot="footer" class="dialog-footer">
    <el-button @click="centerDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="save">确 定</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Main",
        data() {
            let checkAge=(rule,value,callback)=>{
                if(value>150){
                    callback(new Error('年龄过大'));
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
                        callback(new Error('账号已存在'));
                    }
                })
            }
            return {
                tableData: [],
                pageSize:10,
                pageNum:1,
                total:0,
                name:'',
                sex: '',
                sexs:[
                    {
                        value: '1',
                        label: '男'
                    }, {
                        value: '0',
                        label: '女'
                    }
                ],
                centerDialogVisible:false,
                form:{
                    id:'',
                    no:'',
                    password:'',
                    name:'',
                    age:'',
                    phone:'',
                    sex:'1',
                    roleId:'2'
                },
                rules: {
                    name: [
                        {required: true, message: '请输入姓名', trigger: 'blur'},
                    ],
                    no: [
                        {required: true, message: '请输入账号', trigger: 'blur'},
                        {min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur'},
                        {validator:checkDuplicate,trigger: 'blur'}
                    ],
                    password: [
                        {required: true, message: '请输入密码', trigger: 'blur'},
                        {min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur'}
                    ],
                    age: [
                        {required: true, message: '请输入年龄', trigger: 'blur'},
                        {min: 1, max: 3, message: '长度在 1 到 3 个字符', trigger: 'blur'},
                        {pattern:/^([1-9][0-9]*){1,3}$/,message: '年龄必须是正整数',trigger: "blur"},
                        {validator:checkAge,trigger: 'blur'}
                    ],
                    phone: [
                        {required: true, message: '请输入手机号', trigger: 'blur'},
                        {pattern:/^1[3|4|5|6|7|8|9][0-9]\d{8}$/, message: '请输入正确的11位手机号', trigger: 'blur'}
                    ],
                }
            }
        },
        methods:{
            resetForm() {
                this.$refs.form.resetFields();
            },
            add(){
                this.centerDialogVisible=true
                this.$nextTick(()=>{
                    console.log('sss')
                    this.resetForm()
                })
            },
            mod(row){
                //console.log(row)
                this.centerDialogVisible=true
                this.$nextTick(()=>{
                    this.form.id=row.id
                    this.form.no=row.no
                    this.form.name=row.name
                    this.form.password=''
                    this.form.age=row.age+''
                    this.form.sex=row.sex+''
                    this.form.phone=row.phone
                    this.form.roleId=row.roleId

                })


            },
            del(id){
                console.log(id)
                this.$axios.get(this.$httpUrl+'/user/del?id='+id)
                    .then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.$message({
                            message: '操作成功',
                            type: 'success'
                        });
                        this.loadPost()

                    }else {
                        this.$message({
                            message: '出错了',
                            type: 'error'
                        });
                    }

                })
            },
            doSave(){
                this.$axios.post(this.$httpUrl+'/user/save', this.form)
                    .then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.$message({
                            message: '操作成功',
                            type: 'success'
                        });
                        this.centerDialogVisible=false
                        this.loadPost()
                        this.resetForm()
                    }else {
                        console.log("sss")
                        this.$message({
                            message: '出错了',
                            type: 'error'
                        });
                    }

                })
            },
            doMod(){
                this.$axios.post(this.$httpUrl+'/user/update', this.form)
                    .then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.$message({
                            message: '操作成功',
                            type: 'success'
                        });
                        this.centerDialogVisible=false
                        this.loadPost()
                        this.resetForm()
                    }else {
                        this.$message({
                            message: '出错了',
                            type: 'error'
                        });
                    }

                })
            },
            save(){
                this.$refs.form.validate((valid) => {
                    if (valid) {
                        if(this.form.id){
                            this.doMod();

                        }else {
                            this.doSave();
                        }

                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                });


            },
            handleClose(done) {
                this.$confirm('确认关闭？')
                    .then(_ => {
                        done();
                    })
                    .catch(_ => {});
            },
            handleSizeChange(val) {
                console.log(`每页 ${val} 条`);
                this.pageNum=1
                this.pageSize=val
                this.loadPost()
            },
            handleCurrentChange(val) {
                console.log(`当前页: ${val}`);
                this.pageNum=val
                this.loadPost()
            },
            loadGet(){
                this.$axios.get(this.$httpUrl+'/user/list').then(res=>res.data).then(res=>{
                    console.log(res)
                })
            },
            resetParam(){
                this.name='',
                    this.sex=''
            },
            loadPost(){
                this.$axios.post(this.$httpUrl+'/user/listPageC2',{
                    pageSize:this.pageSize,
                    pageNum:this.pageNum,
                    param:{
                        name:this.name,
                        sex:this.sex
                    }

                }).then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.tableData=res.data
                        this.total=res.total
                    }else {
                        alert('获取数据失败')
                    }

                })
            },
        },
        beforeMount() {
            // this.loadGet();
            this.loadPost();
        }
    }
</script>

<style scoped>

</style>