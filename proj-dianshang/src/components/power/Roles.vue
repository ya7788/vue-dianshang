<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="roleslist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <!-- 作用域插槽 -->
          <template v-slot="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                closable
                @close="removeRightById(scope.row, item1.id)"
                >
                {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级三级权限 -->
              <el-col :span="19">
                <!-- 循环嵌套渲染二级权限 -->
                <el-row
                :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                v-for="(item2, i2) in scope.row.children"
                :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                    type="success"
                    closable
                    @close="removeRightById(scope.row, item2.id)"
                    >
                    {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                    v-for="(item3) in item2.children"
                    :key="item3.id"
                    type="warning"
                    closable
                    @close="removeRightById(scope.row, item3.id)"
                    >
                    {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template v-slot="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit"
              >编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete"
              >删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)"
              >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限 -->
    <el-dialog
    title="分配权限"
    :visible.sync="setRightDialogVisible"
    width="50%"
    @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id"
      default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      //   所有角色列表
      roleslist: [],
      // 控制分配权限对话框显示和隐藏
      setRightDialogVisible: false,
      // 所有权限数据
      rightslist: [],
      // 树形控件的绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点ID值
      defKeys: [],
      // 即将分配权限的角色ID
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.roleslist = res.data
      console.log(this.roleslist)
    },
    // 根据ID删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('取消删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) { return this.$message.error('删除用户权限失败') }
      this.$message.success('删除用户权限成功')
      // 只重新为角色赋值权限
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有的权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('查询权限数据失败！')
      }
      // 获取到的权限数据保存到data中
      this.rightslist = res.data
      // 获取三级节点的id
      // console.log(role)
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 递归获取角色三级权限的ID，并保存到refKeys数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // 字符串拼接
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
