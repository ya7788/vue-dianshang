<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        class="treeTable"
      >
        <!-- 是否有效 -->
        <template v-slot:isok="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: green"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template v-slot:order="scope">
          <el-tag size:mini v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size:mini v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" size:mini v-else-if="scope.row.cat_level === 2"
            >三级</el-tag
          >
        </template>
        <!-- 操作 -->
        <template v-slot:opt>
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClose"
    >
      <!-- 添加分类表单 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="ruleCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader
            expand-trigger="hover"
            check-strictly:
            true
            :options="parentCateList"
            :props="cascaderProps"
            v-model="selectedKeys"
            @change="parentCateChanged"
          >
          </el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      catelist: [],
      queryInfo: {
        type: 3,
        // 当前页数
        pagenum: 1,
        // 当前每页显示几条数据
        pagesize: 5
      },
      total: 0,
      //   为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          //   表示将当前列定义为模版列
          type: 'template',
          //   表示当前这一列使用模版名称
          template: 'isok'
        },
        {
          label: '排序',
          //   表示将当前列定义为模版列
          type: 'template',
          //   表示当前这一列使用模版名称
          template: 'order'
        },
        {
          label: '操作',
          //   表示将当前列定义为模版列
          type: 'template',
          //   表示当前这一列使用模版名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        cat_name: '',
        // 父级分类ID
        cat_pid: 0,
        // 当前分类等级
        cat_level: 0
      },
      //   添加分类表单的规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      //   父级分类列表
      parentCateList: [],
      //   指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //   选中的父级分类的ID数组
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获得商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      //   console.log找到数据存储在result
      this.catelist = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
      //   console.log(res.data)
    },
    // 监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮展示对话框
    showAddCateDialog() {
      //   先获取父级分类的数据列表
      this.getParentCateList()
      //   再展示对话框
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) return this.$message.error('获取父级分类失败')
      //   console.log(res.data)
      this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChanged() {
      console.log(this.selectedKeys)
      //   如果this.selectedKeys长度大于0，证明选中父级分类
      if (this.selectedKeys.length > 0) {
        //   最后一项父级ID
        this.addCateForm.cat_pid =
          this.selectedKeys[this.selectedKeys.length - 1]
        //   为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮添加新的分类
    addCate() {
      this.$refs.ruleCateFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return false
        // 发起分类请求
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        // 隐藏添加分类的对话框
        this.addCateDialogVisible = false
        // 重新获取分类列表
        this.getCateList()
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClose() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>
<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
</style>
