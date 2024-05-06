<template>
    <div style="display: flex;line-height: 60px">
        <div style="cursor: pointer">
            <i :class="icon" style="font-size: 24px;line-height: 60px"
            @click="collapse"
            ></i>
        </div>
        <div style="flex: 1;text-align: center;font-size: 22px" >
            <span>欢迎来到仓库管理系统</span>
        </div>
        <span>{{user.name}}</span>
        <el-dropdown>

            <i class="el-icon-arrow-down" style="margin-left: 5px"></i>
            <el-dropdown-menu slot="dropdown">
                <el-dropdown-item @click.native="toUser">个人中心</el-dropdown-item>
                <el-dropdown-item @click.native="logout">退出登录</el-dropdown-item>

            </el-dropdown-menu>
        </el-dropdown>

    </div>

</template>

<script>
    export default {
        name: "Header",
        data(){
            return{
               user:JSON.parse(sessionStorage.getItem('CurUser'))
            }
        },
        props:{
            icon:String
        },
        methods:{
            toUser(){
               console.log('to_User')

                this.$router.push("/Home")
            },
            logout(){
                console.log('log_out')

                this.$confirm('此操作将退出登录, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$message({
                        type: 'success',
                        message: '退出登录成功!'
                    });
                    this.$router.push("/")
                    sessionStorage.clear()
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消'
                    });
                });

            },
            collapse(){
                this.$emit('doCollapse')
            }
        },
        created() {
            this.$router.push("/Home")
        }
    }
</script>

<style scoped>

</style>