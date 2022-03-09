<template>
<el-container class="home-container">
    <!--头部区域-->
    <el-header>
        <div>
            <img src="../assets/heima.png" alt="">
            <span>电商管理系统</span>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!--页面主体区-->
    <el-container>
    <!--侧边栏-->
    <el-aside :width="isCollapse?'64px':'200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
        background-color="#333744"
        text-color="#fff"
        active-text-color="#409EFF"
        :unique-opened="true"
        :collapse="isCollapse"
        :collapse-transition="false"
        :router="true"
        :default-active="activePath">
            <!--一级菜单-->
            <el-submenu :index="item.id+''" v-for=" item in menuList" :key=item.id>
                <!--一级菜单的模板区域-->
                <template slot="title">
                    <!--图标-->
                    <i :class="iconsObj[item.id]"></i>
                    <!-- 文本 -->
                    <span>{{item.authName}}</span>
                </template>
                <!-- 二级菜单 -->
                <!--:index="'/'+subItem.path"-->
                <!--是否使用 vue-router (前面)的模式，启用该模式会在激活导航时以 index 作为 path 进行路由跳转-->
                <!-- default-active	当前激活菜单的 index -->
                <el-menu-item :index="subItem.path" v-for="subItem in item.children" 
                :key="subItem.id" @click="saveNavState(subItem.path)">
                    <template slot="title">
                        <!--图标-->
                        <i class="el-icon-menu"></i>
                        <!-- 文本 -->
                        <span>{{subItem.authName}}</span>
                    </template>
                </el-menu-item>
            </el-submenu>
        </el-menu>        
    </el-aside>
    <!--右侧内容主体-->
    <el-main>
        <router-view></router-view>
    </el-main>
    </el-container> 
</el-container>
</template>

<script>
export default {
    data(){
        return{
            //左侧菜单数据
            menuList:[],
            iconsObj: {
                '125':'iconfont icon-user',
                '103':'iconfont icon-tijikongjian',
                '101':'iconfont icon-shangpin',
                '102':'iconfont icon-danju',
                '145':'iconfont icon-baobiao'
            },
            // 是否折叠
            isCollapse:false,
            // 被激活的链接地址
            activePath:''

        }
    },
    created(){
        this.getMenuList()
        this.activePath=window.sessionStorage.getItem('activePath')
    },
    methods:{
        logout(){
            window.sessionStorage.clear();
            this.$router.push('/login');
        },
        // 获取所有的菜单
        async getMenuList(){
            const {data:res}=await this.$http.get('menus')
            if(res.meta.status!=200)return this.$message.error(res.meta.msg)
            this.menuList=res.data
            console.log(res,'菜单')
        },
        // 点击按钮，切换菜单的折叠与展开
        toggleCollapse(){
            this.isCollapse=!this.isCollapse
        },
        saveNavState(activePath){
            //保存状态，到时候取出来（creared里）,例如刷新时
            window.sessionStorage.setItem('activePath',activePath)
            this.activePath=activePath
        }
    }

}
</script>

<style lang="less" scoped>
.home-container{
    height: 100%;
}
.el-header{
    background-color: #373d41;
    display: flex;
    //项目位于各行之前、之间、之后都留有空白的容器内。
    justify-content: space-between;
    padding-left: 0;
    //align-items 属性定义flex子项在flex容器的当前行的侧轴（纵轴）方向上的对齐方式。
    align-items: center;
    color: #fff;
    font-size: 20px;
    >div{
        display: flex;
        align-items: center;
        span{
            margin-left: 15px;
        }
    }
}
.el-aside{
    background-color: #333744;
    .el-menu{
        border-right: none;
    }
}
.el-main{
    background-color: #eaedf1;
}
//每个图标都对应一个类iconfont
.iconfont{
    margin-left: 10px;
    margin-right: 10px;
}
.toggle-button{
    background-color: #4a5064;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
}
</style>

  
  
  
