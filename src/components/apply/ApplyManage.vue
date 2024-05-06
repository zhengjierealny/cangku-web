<template>
    <div>
        <div style="margin-bottom: 5px;">
            <el-input v-model="name" placeholder="请输入物品名" suffix-icon="el-icon-search" style="width: 200px;"
                      @keyup.enter.native="loadPost"></el-input>
            <el-select v-model="storage" placeholder="请选择仓库" style="margin-left: 5px;">
                <el-option
                        v-for="item in storageData"
                        :key="item.id"
                        :label="item.name"
                        :value="item.id">
                </el-option>
            </el-select>
            <el-select v-model="selectedOption" placeholder="请选择申请状态" style="margin-left: 5px;" v-if="user.userid!=2">
                <el-option
                        v-for="option in applyState"
                        :key="option.value"
                        :label="option.state"
                        :value="option.value">
                </el-option>
            </el-select>
            <el-select v-model="goodstype" placeholder="请选择分类" style="margin-left: 5px;">
                <el-option
                        v-for="item in goodstypeData"
                        :key="item.id"
                        :label="item.name"
                        :value="item.id">
                </el-option>
            </el-select>

            <el-button type="primary" style="margin-left: 5px;" @click="loadPost">查询</el-button>
            <el-button type="success" @click="resetParam">重置</el-button>


        </div>
        <el-table :data="tableData"
                  :header-cell-style="{ background: '#f2f5fc', color: '#555555' }"
                  border
        >
<!--            <el-table-column prop="id" label="ID" width="60">-->
<!--            </el-table-column>-->
            <el-table-column prop="goodsname" label="物品名" width="160">
            </el-table-column>
            <el-table-column prop="storagename" label="仓库" width="160">
            </el-table-column>
            <el-table-column prop="goodstypename" label="分类" width="160">
            </el-table-column>
            <el-table-column prop="adminname" label="操作人" width="160">
            </el-table-column>
            <el-table-column prop="username" label="申请人" width="160">
            </el-table-column>
            <el-table-column prop="count" label="数量" width="160">
            </el-table-column>
            <el-table-column prop="createtime" label="操作时间" width="160">
            </el-table-column>
            <el-table-column prop="remark" label="备注（不通过原因）">
            </el-table-column>
            <el-table-column prop="address" label="收货地址">
            </el-table-column>
            <el-table-column prop="state" label="状态" width="180">
                <template slot-scope="scope">
                    <el-tag
                            :type="scope.row.state === 0 ? 'danger' : (scope.row.state === 1 ? 'primary' : 'success')"
                            disable-transitions>{{scope.row.state === 0 ? '未处理' :
                        (scope.row.state === 1 ? '不通过' : '通过')}}</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="operate" label="操作">
                <template slot-scope="scope">
                    <el-button size="small" type="success" @click="mod(scope.row)"
                               v-if="(user.roleId!=2)&&(scope.row.state!==2)&&(scope.row.state!==1)" >审核
                    </el-button>
                    <el-popconfirm
                            title="确定取消吗？"
                            @confirm="cancel(scope.row.id)"
                            style="margin-left: 5px;"
                    >
                        <el-button slot="reference" size="small" type="danger"
                                   v-if="user.roleId==2&&scope.row.state==0">取消申请</el-button>
                    </el-popconfirm>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="pageNum"
                :page-sizes="[5, 10, 20,30]"
                :page-size="pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
        </el-pagination>

        <el-dialog
                title="审核"
                :visible.sync="centerDialogVisible"
                width="30%"
                center>

            <el-form ref="form" :rules="rules" :model="form" label-width="80px">
                <el-form-item label="物品名" prop="no">
                    <el-col :span="20">
                        <el-input v-model="form.goodsname"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="申请人" prop="name">
                    <el-col :span="20">
                        <el-input v-model="form.username"></el-input>
                    </el-col>
                </el-form-item>
<!--                <el-form-item label="操作人" prop="name">-->
<!--                    <el-col :span="20">-->
<!--                        <el-input v-model="form.adminname"></el-input>-->
<!--                    </el-col>-->
<!--                </el-form-item>-->
                <el-form-item label="数量" prop="password">
                    <el-col :span="20">
                        <el-input v-model="form.count"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="收货地址" prop="address">
                    <el-col :span="20">
                        <el-input type="textarea" v-model="form.address"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="备注（原因）" prop="remark">
                    <el-col :span="20">
                        <el-input type="textarea" v-model="form.remark"></el-input>
                    </el-col>
                </el-form-item>

            </el-form>
            <span slot="footer" class="dialog-footer">
    <el-button @click="centerDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="save">通过</el-button>
                <el-button type="primary" @click="del">驳回申请</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "ApplyManage",
        data() {

            return {
                user : JSON.parse(sessionStorage.getItem('CurUser')),
                storageData:[],
                goodstypeData:[],
                selectedOption: '',
                applyState:[
                    {
                        value: '0',
                        state: '未审核'
                    }, {
                        value: '1',
                        state: '未通过'
                    }, {
                        value: '2',
                        state: '已通过'
                    }
                ],
                tableData: [],
                pageSize:10,
                pageNum:1,
                total:0,
                name:'',
                storage:'',
                goodstype:'',
                address:'',
                centerDialogVisible:false,
                problemTotal:0,
                form:{
                    applysid:'',
                    id:'',
                    goods:'',
                    adminId:'',
                    adminname:'',
                    goodsname:'',
                    userid:'',
                    username:'',
                    storage:'',
                    goodstype:'',
                    count:'',
                    remark:'',
                    address:'',
                    state:'0'
                },
                rules:{

                }
            }
        },
        watch:{
            problemTotal:function (newVal,oldVal) {
                if(newVal>0){
                    alert('仓库容量不足,请前往处理');
                }
            }
        },
        methods:{
            // del(){
            //
            //
            //     this.$axios.post(this.$httpUrl+'/applys/del').then(res=>res.data).then(res=>{
            //         console.log(res)
            //         if(res.code==200){
            //
            //             this.$message({
            //                 message: '操作成功！',
            //                 type: 'success'
            //             });
            //             this.loadPost()
            //         }else{
            //             this.$message({
            //                 message: '操作失败！',
            //                 type: 'error'
            //             });
            //         }
            //
            //     })
            // },
            mod(row){
                console.log(this.user)
                this.centerDialogVisible = true
                this.$nextTick(()=>{
                    //赋值到表单
                    this.form.id=row.id
                    this.form.applysid = row.id
                    this.form.goods=row.goods
                    this.form.userid=row.userid
                    this.form.goodsname = row.goodsname
                    this.form.username = row.username
                    this.form.adminname=row.adminname
                    this.form.adminId=this.user.id
                    this.form.count = row.count
                    this.form.remark = row.remark
                    this.form.address=row.address
                })
            },
            save(){
                this.$axios.post(this.$httpUrl+'/applys/save',this.form).then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){

                        this.$message({
                            message: '审核操作成功！',
                            type: 'success'
                        });
                        this.centerDialogVisible = false

                        this.loadPost()
                        this. resetInForm()
                    }else{
                        this.$message({
                            message: '审核操作失败！',
                            type: 'error'
                        });
                    }

                })
                this.$axios.post(this.$httpUrl+'/record/save',this.form).then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){

                        this.$message({
                            message: '记录保存操作成功！',
                            type: 'success'
                        });
                        this.applyDialogVisible = false
                        this.loadPost()
                        this. resetInForm()
                    }else{
                        this.$message({
                            message: '记录保存操作失败！',
                            type: 'error'
                        });
                    }

                })
            },
            del(){
                if(this.form.remark.trim().length > 0){
                    this.$axios.post(this.$httpUrl+'/applys/del',this.form).then(res=>res.data).then(res=>{
                        console.log(res)
                        if(res.code==200){

                            this.$message({
                                message: '操作成功！',
                                type: 'success'
                            });
                            this.centerDialogVisible = false
                            this.loadPost()
                            this. resetInForm()
                        }else{
                            this.$message({
                                message: '操作失败！',
                                type: 'error'
                            });
                        }

                    })
                }else {
                    alert('请输入原因');
                }

            },
            cancel(id){
                console.log(id)

                this.$axios.get(this.$httpUrl+'/applys/cancel?id='+id).then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){

                        this.$message({
                            message: '操作成功！',
                            type: 'success'
                        });
                        this.loadPost()
                    }else{
                        this.$message({
                            message: '操作失败！',
                            type: 'error'
                        });
                    }

                })
            },
            formatStorage(row){
                let temp =  this.storageData.find(item=>{
                    return item.id == row.storage
                })

                return temp && temp.name
            },
            formatGoodstype(row){
                let temp =  this.goodstypeData.find(item=>{
                    return item.id == row.goodstype
                })

                return temp && temp.name
            },
            resetInForm(){
                this.$refs.form.resetFields();
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
            resetParam(){
                this.name=''
                this.storage=''
                this.goodstype=''
                this.loadPost()
            },
            resetForm() {
                this.$refs.form.resetFields();
            },
            loadStorage(){
                this.$axios.get(this.$httpUrl+'/storage/list').then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.storageData=res.data
                    }else{
                        alert('获取数据失败')
                    }

                })
            },
            loadGoodstype(){
                this.$axios.get(this.$httpUrl+'/goodstype/list').then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.goodstypeData=res.data
                    }else{
                        alert('获取数据失败')
                    }

                })
            },
            loadPost(){
                console.log(this.user)
                this.$axios.post(this.$httpUrl+'/applys/listPage',{
                    pageSize:this.pageSize,
                    pageNum:this.pageNum,
                    param:{
                        name:this.name,
                        goodstype:this.goodstype+'',
                        storage:this.storage+'',
                        roleId:this.user.roleId+'',
                        state: this.selectedOption+'',
                        userId:this.user.id+''
                    }

                }).then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.tableData=res.data
                        this.total=res.total
                    }else{
                        alert('获取数据失败')
                    }

                });
                this.$axios.post(this.$httpUrl+'/storage/isProblem',{
                    pageSize:this.pageSize,
                    pageNum:this.pageNum,
                    param:{

                    }
                }).then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){

                        this.problemTotal=res.total
                    }else{
                        alert('获取数据失败')
                    }

                });
            },
        },
        beforeMount() {
            this.loadStorage()
            this.loadGoodstype()
            this.loadPost()

        }
    }
</script>

<style scoped>

</style>