<template>
  <div>
    <el-button type="danger" @click="batchDelete()">批量删除</el-button>
    <el-tree :data="data"
             :props="defaultProps"
             ref="categoryTree"
             show-checkbox
             :default-expanded-keys="expandedKey"
             node-key="id"
             :expand-on-click-node="false">
     <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog title="添加/修改 分类" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="单位">
          <el-input v-model="category.proUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdate">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      dialogType: '',
      dialogVisible: false,
      category: {
        id: null,
        name: '',
        parentId: 0,
        catLevel: 0,
        isShow: 1,
        sort: 0,
        icon: '',
        proUnit: '',
        proCount: 0
      },
      expandedKey: [],
      defaultProps: {
        children: 'childrenCategories',
        label: 'name'
      }
    };
  },
  methods: {

    batchDelete() {
      var checkedNodes = this.$refs.categoryTree.getCheckedNodes()
      var ids = []
      var categoryNames = []
      for (var i = 0; i < checkedNodes.length; i++) {
        ids.push(checkedNodes[i].id)
        categoryNames.push(checkedNodes[i].name)
      }

      this.$confirm(`是否批量删除[${categoryNames}]菜单?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: 'http://localhost:9090/commodity/category/delete',
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            message: '批量删除成功OK',
            type: 'success',
          })
          this.getCategories()
        })
      }).catch(() => {

      })
    },

    addOrUpdate() {
      if (this.dialogType === 'add') {
        this.addCategory();
      } else if (this.dialogType === 'edit') {
        this.updateCategory();
      }
    },
    updateCategory() {
      var {id, name, icon, proUnit} = this.category
      this.$http({
        url: 'http://localhost:9090/commodity/category/update',
        method: 'post',
        data: this.$http.adornData({id, name, icon, proUnit}, false)
      }).then(({data}) => {
        this.$message({
          message: '分类信息修改成功OK',
          type: 'success',
        })
        this.dialogVisible = false;
        this.getCategories()
        this.expandedKey = [this.category.parentId]
      })
    },

    edit(data) {
      this.dialogType = 'edit';
      this.dialogVisible = true;
      this.$http({
        url: `http://localhost:9090/commodity/category/info/${data.id}`,
        method: 'get',
      }).then(({data}) => {
        this.category.name = data.category.name
        this.category.id = data.category.id
        this.category.icon = data.category.icon
        this.category.proUnit = data.category.proUnit
        this.category.parentId = data.category.parentId
      })
    },

    addCategory() {
      this.$http({
        url: 'http://localhost:9090/commodity/category/save',
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: '分类信息保存成功OK',
          type: 'success',
        })
        this.dialogVisible = false;
        this.getCategories()
        this.expandedKey = [this.category.parentId]
      })
    },

    append(data) {
      this.dialogType = 'add';
      this.dialogVisible = true;
      this.category.parentId = data.id;
      this.category.sort = 0;
      this.category.proUnit = '';
      this.category.proCount = 0;
      this.category.name = '';
      this.category.isShow = 1;
      this.category.id = null;
      this.category.icon = '';
      this.category.catLevel = data.catLevel * 1 + 1;
    },

    remove(node, data) {
      var ids = [data.id]
      this.$confirm(`是否删除[${data.name}]菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: 'http://localhost:9090/commodity/category/delete',
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            message: '操作成功',
            type: 'success',
          })
          this.getCategories()
          this.expandedKey = [node.parent.data.id]
        })
      }).catch(() => {

      })

    },
    // 获取数据列表
    getCategories() {
      this.$http({
        url: 'http://localhost:9090/commodity/category/list/tree',
        method: 'get',
      }).then(({data}) => {
        console.log(data);
        this.data = data.data
      })
    }
  },

  created() {
    this.getCategories();
  }
};
</script>

<style scoped>

</style>
