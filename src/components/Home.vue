<template>
    <div style="text-align: center;background-color: #ffffff;height: 100%;padding: 0px;margin: 0px;">
        <h1 style="font-size: 50px;">{{'欢迎你！'+user.name}}</h1>
        <el-descriptions  title="个人中心" :column="2" size="40" border>
            <el-descriptions-item>
                <template slot="label">
                    <i class="el-icon-s-custom"></i>
                    用户名
                </template>
                {{user.name}}
            </el-descriptions-item>
            <el-descriptions-item>
                <template slot="label">
                    <i class="el-icon-mobile-phone"></i>
                    电话
                </template>
                {{user.phone}}
            </el-descriptions-item>
            <el-descriptions-item>
                <template slot="label">
                    <i class="el-icon-location-outline"></i>
                    性别
                </template>
                <el-tag
                        :type="user.sex === '1' ? 'primary' : 'danger'"
                        disable-transitions><i :class="user.sex==1?'el-icon-male':'el-icon-female'"></i>{{user.sex==1?"男":"女"}}</el-tag>
            </el-descriptions-item>
            <el-descriptions-item>
                <template slot="label">
                    <i class="el-icon-tickets"></i>
                    角色
                </template>
                <el-tag
                        type="success"
                        disable-transitions>{{user.roleId==0?"超级管理员":(user.roleId==1?"管理员":"用户")}}</el-tag>

            </el-descriptions-item>

            <el-button size="small" type="success" @click="mod(scope.row)">编辑</el-button>

        </el-descriptions>

<!--        <div style="margin-left: -86%;margin-top: 40px">-->
<!--            <span  v-if="user.roleId!=0" style="font-size: 20px;">超级管理员联系方式：15824060166</span>-->
<!--        </div>-->


        <div style="margin-left: -86%;margin-top: 40px">
            <span  v-if="(user.roleId!=2)&&(this.tableData.length>0)" style="font-size: 20px;color: red">仓库异常</span>
        </div>

        <el-table :data="tableData"
                  :header-cell-style="{ background: '#f2f5fc', color: '#555555' }"
                  border
                  v-if="(user.roleId!=2)&&(this.tableData.length>0)"
                  style="width: 361px;margin-left: 4%;margin-top: 10px"
        >
            <el-table-column prop="name" label="仓库名" width="180px">
            </el-table-column>
            <el-table-column prop="state" label="状态" width="180px">
                <template slot-scope="scope">
                    <el-tag
                            :type="scope.row.totalcap-scope.row.currentcap < 100 ? 'danger' : 'success'"
                            disable-transitions >{{scope.row.totalcap-scope.row.currentcap < 100 ? '仓库容量不足' :
                        '仓库容量良好'}}</el-tag>
                </template>
            </el-table-column>
        </el-table>
        <div style="margin-left: -83%;margin-top: 40px">
            <span  v-if="user.roleId!=2" style="font-size: 20px;">待审核申请数: </span>
            <span v-if="user.roleId!=2">{{this.applysTotal}}</span>
        </div>
        <DateUtils style="margin-bottom: 10px"></DateUtils>
    </div>
</template>

<script>
    import DateUtils from "./DateUtils";
    export default {
        name: "Home",
        components: {DateUtils},
        data() {

            return {
                tableData: [],
                applysData:[],
                total:0,
                applysTotal:0,
                name:'',
                totalcap:'',
                currentcap:'',
                user:{},
                centerDialogVisible:false,

            }
        },
        computed:{

        },
        methods:{
            init(){
                this.user = JSON.parse(sessionStorage.getItem('CurUser'))
            },
            loadPost(){
                this.$axios.post(this.$httpUrl+'/storage/listPageSS',{
                }).then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.tableData=res.data
                        console.log("1"+this.tableData)
                        this.total=res.total
                    }else{
                        alert('获取数据失败')
                    }

                });
                this.$axios.post(this.$httpUrl+'/applys/listPageA',{
                }).then(res=>res.data).then(res=>{
                    console.log(res)
                    if(res.code==200){
                        this.applysData=res.data
                        this.applysTotal=res.total
                    }else{
                        alert('获取数据失败')
                    }

                })
            }
        },

        created(){
            this.init()
        },
        beforeMount() {
            this.loadPost()
        }
    }
</script>

<style scoped>
    .el-descriptions{
        width:90%;

        margin: 0 auto;
        text-align: center;
    }
</style>