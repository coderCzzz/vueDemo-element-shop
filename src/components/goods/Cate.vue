<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加分类 -->
      <el-row>
        <el-col :span="2">
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!-- 分类列表 -->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
      >
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen;"
          ></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            type="success"
            size="mini"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button
            type="primary"
            class="el-icon-edit"
            size="mini"
            @click="editCateById(scope.row.cat_id)"
            >编辑</el-button
          >
          <el-button
            type="danger"
            class="el-icon-delete"
            size="mini"
            @click="removeCateById(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 30]"
        @current-change="handleCurrentChange"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="handleSizeChange"
        :page-size="queryInfo.pagesize"
        :total="total"
      ></el-pagination>
    </el-card>
    <el-dialog
      title="添加分类"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="addCateClosed"
    >
      <el-form
        :model="editForm"
        ref="editFormRef"
        label-width="100px"
        :rules="addCateRules"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="editForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类: ">
          <el-cascader
            :options="parentCateList"
            v-model="selectedKeys"
            expand-trigger="hover"
            @change="parentCateChanged"
            :props="cascaderProps"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCateInfo">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  created() {
    this.getCateList()
  },
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      addCateRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      catelist: [],
      parentCateList: [],
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      total: 0,
      editForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      editDialogVisible: false,
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedKeys: []
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败！')
      }
      console.log(res.data)
      this.catelist = res.data.result
      this.total = res.data.total
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    async editCateById(id) {
      const { data: res } = await this.$http.get(`categories/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类详情失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    async editCateInfo() {
      const { data: res } = await this.$http.put(
        'categories/' + this.editForm.cat_id,
        this.editForm
      )
      if (res.meta.status !== 200) {
        return this.$message.error('更新商品分类失败！')
      }
      this.editDialogVisible = false
      this.getCateList()
      this.$message.success('更新商品分类成功！')
    },
    async removeCateById(id) {
      const confirmResult = await this.$confirm(
        '该操作将永久删除该分类，是否继续',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除操作')
      }
      const { data: res } = await this.$http.delete(`categories/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败！')
      }
      this.$message.success('删除分类成功！')
      this.getCateList()
    },
    showAddCateDialog() {
      this.getParentCateList()
      this.editDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级数据失败')
      }
      this.parentCateList = res.data
    },
    parentCateChanged() {
      if (this.selectedKeys.length > 0) {
        this.editForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.editForm.cat_level = this.selectedKeys.length
        return
      }
      this.editForm.cat_pid = 0
      this.editForm.cat_level = this.selectedKeys.length
    },
    addCateClosed() {
      this.$refs.editFormRef.resetFields()
      this.selectedKeys = []
      this.editForm.cat_pid = 0
      this.editForm.cat_level = 0
    },
    async addCateInfo() {
      this.$refs.editFormRef.validate(async value => {
        if (!value) return
        const { data: res } = await this.$http.post(
          'categories',
          this.editForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.editDialogVisible = false
      })
    }
  }
}
</script>
<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
