<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
        <!-- 头部警告区域 -->
        <el-alert
        title="注意，只允许为第三级分类参数设置相关参数"
        type="warning"
        :closable="false"
        show-icon>
        </el-alert>
        <!-- 选择商品分类区域 -->
        <el-row class="cat_opt">
            <el-col>
                <span>选择商品分类：</span>
                <!-- 选择商品分类的级联 -->
                <el-cascader
                expand-trigger="hover"
                :options="catelist"
                :props="cateProps"
                v-model="selectedCateKeys"
                @change="cateChanged"
                class="cas">
                </el-cascader>
            </el-col>
        </el-row>
        <!-- tab页签区域 -->
        <el-tabs v-model="activeName" @tab-click="handleTabClick">
          <el-tab-pane label="动态参数" name="many">
            <!-- 动态参数按钮 -->
            <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addParamsDialogVisible=true">添加参数</el-button>
            <!-- 动态参数表格 -->
            <el-table :data="manyTableData" border stripe>
              <!-- 展开行 -->
              <el-table-column type="expand">
                <template v-slot="scope">
                  <!-- 循环渲染tag标签 -->
                  <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">
                    {{item}}
                  </el-tag>
                  <!-- 输入的文本框 -->
                  <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="$event.target.blur( )"
                  @blur="handleInputConfirm(scope.row)"
                  >
                  </el-input>
                  <!-- 添加按钮 -->
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index"></el-table-column>
              <el-table-column label="参数名称" prop="attr_name"></el-table-column>
              <el-table-column label="操作">
                <template v-slot="scope">
                  <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                  <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="静态属性" name="only">
            <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addParamsDialogVisible=true">添加属性</el-button>
            <!-- 静态属性表格 -->
            <el-table :data="onlyTableData" border stripe>
                <!-- 展开行 -->
              <el-table-column type="expand">
                <template v-slot="scope">
                  <!-- 循环渲染tag标签 -->
                  <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">
                    {{item}}
                  </el-tag>
                  <!-- 输入的文本框 -->
                  <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="$event.target.blur( )"
                  @blur="handleInputConfirm(scope.row)"
                  >
                  </el-input>
                  <!-- 添加按钮 -->
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index"></el-table-column>
              <el-table-column label="参数名称" prop="attr_name"></el-table-column>
              <el-table-column label="操作">
                <template v-slot="scope">
                  <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                  <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog :title="'添加' + titleText" :visible.sync="addParamsDialogVisible" width="50%" @close="addDialogClose">
      <!-- 内容主体区域 -->
      <el-form
        :model="addParamsForm"
        :rules="addParamsFormRules"
        ref="ruleParamsFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数的对话框 -->
    <el-dialog :title="'修改' + titleText" :visible.sync="editParamsDialogVisible" width="50%" @close="editDialogClose">
      <!-- 内容主体区域 -->
      <el-form
        :model="editParamsForm"
        :rules="editParamsFormRules"
        ref="editParamsFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类列表
      catelist: [],
      //   级联选择器配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //   级联选择框双向绑定到的数组
      selectedCateKeys: [],
      //   被激活的页签名称
      activeName: 'many',
      //   动态参数数据
      manyTableData: [],
      //   静态参数数据
      onlyTableData: [],
      // 添加参数对话框的显示与隐藏
      addParamsDialogVisible: false,
      //   添加参数的表单数据
      addParamsForm: {
        attr_name: ''
      },
      //   添加参数表单的规则对象
      addParamsFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      },
      // 修改参数对话框的显示与隐藏
      editParamsDialogVisible: false,
      // 修改参数的表单数据
      editParamsForm: {
        attr_name: '',
        attr_id: ''
      },
      //   修改参数表单的规则对象
      editParamsFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      },
      // 控制按钮和文本框的切换显示
      inputVisible: false,
      // 文本框输入的内容
      inputValue: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获得商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {})
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      //   console.log找到数据存储在result
      this.catelist = res.data
    },
    // 级联选择项发生变化触发这个函数
    async cateChanged() {
      this.getParamsData()
    },
    // tab页签点击事件的处理函数
    handleTabClick() {
      console.log(this.activeName)
      this.getParamsData()
    },
    async getParamsData() {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
      }
      console.log(this.selectedCateKeys)
      //   根据所选分类的ID，和当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本框的显示与隐藏
        item.inputVisible = false
        // 文本框输入的值
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') this.manyTableData = res.data
      this.onlyTableData = res.data
    },
    // 监听添加对话框的关闭事件
    addDialogClose() {
      this.$refs.ruleParamsFormRef.resetFields()
    },
    // 点击按钮添加参数
    addParams() {
      this.$refs.ruleParamsFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return false
        // 发起角色请求
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addParamsForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        // 隐藏添加角色的对话框
        this.addParamsDialogVisible = false
        // 重新获取角色列表
        this.getParamsData()
      })
    },
    //   点击按钮展示修改参数的对话框
    async showEditDialog(attrid) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attrid}`, {
        params: { attr_sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败！')
      }
      this.editParamsForm = res.data
      this.editParamsDialogVisible = true
    },
    // 监听修改对话框的关闭事件
    editDialogClose() {
      this.$refs.editParamsFormRef.resetFields()
    },
    // 点击按钮修改参数
    editParams() {
      this.$refs.editParamsFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return false
        // 发起角色请求
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editParamsForm.attr_id}`, {
          attr_name: this.editParamsForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }
        this.$message.success('修改参数成功！')
        // 隐藏添加角色的对话框
        this.addParamsDialogVisible = false
        // 重新获取角色列表
        this.getParamsData()
      })
    },
    //   根据ID删除对应的参数项
    async removeParams(attrid) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户点击确定返回字符串confirm   取消则返回cancel
      console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrid}`)
      if (res.meta.status !== 200) { this.$message.error('删除用户失败') }
      this.$message.success('删除用户成功')
      this.getParamsData()
    },
    // 将对attr_vals的操作保存到数据库
    async saveAttrVals(row) {
      // 发起请求保存数据
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(',')
      })
      if (res.meta.status !== 200) { this.$message.error('修改参数项失败') }
      this.$message.success('修改参数项成功')
    },
    // 点击按钮，展示文本输入框
    showInput(row) {
      row.inputVisible = true
      // 让文本框自动获得焦点
      // $nextTick，当页面上的元素被重新渲染之后才会指定回调函数的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    // 删除对应参数可选项
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 如果按钮需要被禁用则返回true
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) return true
      return false
    },
    // 当前选中的三级分类ID
    cateId() {
      if (this.selectedCateKeys.length === 3) return this.selectedCateKeys[2]
      return null
    },
    // 动态计算标题的文本
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt {
    margin-top:15px;
}
.cas {
    width: 30%;
}
.el-tag {
  margin: 15px;
}
.input-new-tag {
  width: 130px;
}
</style>
