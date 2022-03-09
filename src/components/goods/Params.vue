<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>   

    <!-- 卡片视图区域 -->   
    <el-card>
        <!-- 警告区域 -->
        <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" show-icon :closable="false">
        </el-alert> 

         <!-- 选择商品属性分类区域 -->
          <el-row class="cat_opt">
            <el-col>
                 <span>选择商品分类：</span>
                 <!-- 选择商品分类的级联选择框 -->
                <el-cascader
                  v-model="selectedCateKeys"
                  :options="cateList"
                  :props="cateProps"
                  @change="handleChange"
                  size= medium ></el-cascader>                 
             </el-col>
          </el-row>

          <!-- tab 页签区域 -->
          <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <!-- 添加动态参数的面板 -->
            <el-tab-pane label="动态参数" name="many">
              <el-button @click="addDialogVisible=true" type="primary" size="mini" :disabled="isBtnDisabled">添加参数</el-button>
              <el-table :data="manyTableData" stripe >
              <!-- 展开行 -->
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <!-- 循环渲染Tag标签 -->
                  <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(scope.row,i)"> {{item}}</el-tag>
                  <el-input v-model="scope.row.inputValue" v-if="scope.row.inputVisible" placeholder="请输入内容" :autofocus="true" @change="handleInputChange(scope.row)" class="input-new-tag"></el-input>
                  <el-button v-else @click="showInput(scope.row)" size="small">+添加新的属性</el-button>
                </template>
              </el-table-column>
              <!-- 索引列 -->
              <el-table-column type="index"></el-table-column>                
              <el-table-column prop="attr_name" label="参数名称" width="180"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row)">编辑</el-button>
                  <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row)">删除</el-button>
                </template>
              </el-table-column>              
              </el-table>              
            </el-tab-pane>

            <!-- 添加静态属性的面板 -->
            <el-tab-pane label="静态属性" name="only">
              <el-button @click="addDialogVisible=true" type="primary" size="mini" :disabled="isBtnDisabled">添加参数</el-button>
              <el-table :data="onlyTableData" stripe >
              <!-- 展开行 -->
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <!-- 循环渲染Tag标签 -->
                  <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(scope.row,i)"> {{item}}</el-tag>
                  <el-input v-model="scope.row.inputValue" v-if="scope.row.inputVisible" placeholder="请输入内容" :autofocus="true" @change="handleInputChange(scope.row)" class="input-new-tag"></el-input>
                  <el-button v-else @click="showInput(scope.row)" size="small">+添加新的属性</el-button>
                </template>
              </el-table-column>
              <!-- 索引列 -->
              <el-table-column type="index"></el-table-column>                
              <el-table-column prop="attr_name" label="参数名称" width="180"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row)" >编辑</el-button>
                  <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row)">删除</el-button>
                </template>
              </el-table-column>              
              </el-table>              
            </el-tab-pane>
          </el-tabs>   

          <!-- 添加参数的对话框 -->
          <el-dialog
            :title="'添加'+titleText"
            :visible.sync="addDialogVisible"
             @close="addDialogClosed"
            width="50%">
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
              <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addForm.attr_name"></el-input>
              </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
              <el-button @click="addDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
          </el-dialog>


          <!-- 修改参数的对话框 -->
          <el-dialog :title="'修改' + titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <!-- 添加参数的对话框 -->
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
              <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="editForm.attr_name"></el-input>
              </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
              <el-button @click="editDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
          </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data(){
    return{
      selectedCateKeys:[],
      cateList:[],
      cateProps:{
        expandTrigger: 'hover',
        value:'cat_id',
        label:'cat_name',
        children:'children'
      },
      activeName:'many',
      manyTableData:[],
      onlyTableData:[],
      addForm:{
        attr_name:''
      },
      addFormRules:{
        attr_name:[
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      addDialogVisible:false,
      editForm:{
        attr_name:''
      },
      editDialogVisible:false,
      editFormRules:{
        attr_name:[
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created(){
    this.getCateList()
  },
  methods:{
    async getCateList(){
      const {data:res}=await this.$http.get('categories')
      if(res.meta.status!==200){
        return this.$message.error('获取商品分类失败！')
      }
      console.log(res,'商品列表')
      this.cateList=res.data
    },
    handleChange(){
      this.getParamsData()
    },
    handleTabClick(){
      this.getParamsData()
    },
    async getParamsData(){
      console.log(this.selectedCateKeys,'选中的商品类')
      // 证明选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }      
      const {data:res}=await this.$http.get(`categories/${this.cateId}/attributes`,
        {
          params:{sel:this.activeName}
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }    
      res.data.forEach(item => {
        //考虑参数为空时的显示，如："".split('  ')  输出：[""]，显示一个空标签
        item.attr_vals=item.attr_vals?item.attr_vals.split(' '):[]
        //在此处加入2个参数，每行独有，不是共有
        item.inputVisible=false
        item.inputValue=''
      });  
      console.log(res.data,'商品参数')
      if(this.activeName==='many'){
        this.manyTableData = res.data
        console.log(this.manyTableData,'动态参数')
      }else{
        this.onlyTableData = res.data
        console.log(this.onlyTableData,'静态参数')
      }
    },
    async saveAttrVals(row){
      const {data:res}=await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
      {
        attr_name:row.attr_name,
        attr_sel:row.attr_sel,
        attr_vals:row.attr_vals.join(' ')
      }      
      )
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败！')
      }
      this.$message.success('修改参数项成功！')
    },
    handleClose(row,i){
      row.attr_vals.splice(i,1)
      this.saveAttrVals(row)
    },
    showInput(row){
      row.inputVisible=true
      
    },
    handleInputChange(row){
      if(row.inputValue.trim().length === 0){
        row.inputValue = ''
        row.inputVisible=false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue=''
      row.inputVisible=false
      // 需要发起请求，保存这次操作
      this.saveAttrVals(row)
    },
    addParams(){
      this.$refs.addFormRef.validate(async valid=>{
        if(!valid)return
        const{data:res}=await this.$http.post(`categories/${this.cateId}/attributes`,
        {
          attr_name:this.addForm.attr_name,
          attr_sel:this.activeName
        }
        )
        if(res.meta.status!==201){
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        this.addDialogVisible = false
        this.getParamsData() 
      }
      )

    },
    async showEditDialog(row) {
      const  {data:res}=await this.$http.get(`categories/${this.cateId}/attributes/${row.attr_id}`,{
        params:{
          attr_sel: this.activeName
        }
      }

      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 点击按钮，修改参数信息
    editParams(){
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        )

        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }

        this.$message.success('修改参数成功！')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 重置修改的表单
    editDialogClosed(){
       this.$refs.editFormRef.resetFields()
    },
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    removeParams(row){
        this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async() => {
            const {data:res}=await this.$http.delete(`categories/${this.cateId}/attributes/${row.attr_id}`)
            if (res.meta.status !== 200) {
              return this.$message.error('删除参数失败！')
            }
            this.$message.success('删除参数成功！')
            this.getParamsData()
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });
        

    }
  },
  computed:{
    //选中分类的id
    cateId(){
      if(this.selectedCateKeys.length===3){
        return this.selectedCateKeys[2]
      }
      return null
    },
    isBtnDisabled(){
      if(this.selectedCateKeys.length===3){
        return false
      }
      return true
    },
    titleText(){
      if(this.activeName==="many"){
        return '动态参数'
      }
      return '静态参数'
    }
  }
  
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>