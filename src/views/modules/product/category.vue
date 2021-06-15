<template>
  <div>
    <el-tree :data="menus" 
            :props="defaultProps" 
            :expand-on-click-node="false" 
            show-checkbox node-key="catId"
            :default-expanded-keys="expandedKey"
            draggable
            :allow-drop="allowDrop"
            @node-drop="handleDrop"
            >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" 
          @click="() => append(data)">
            添加
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            修改
          </el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini" 
          @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
      <el-form-item label="分类名称">
        <el-input v-model="category.name" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="图标">
        <el-input v-model="category.icon" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="计量单位">
        <el-input v-model="category.productUnit" autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="submitData(data)">确 定</el-button>
    </span>
  </el-dialog>
  </div>

</template>

<script>
  export default {
    data () {
      return {
        menus: [],
        expandedKey: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        },
        dialogVisible: false,
        category: {
          name: '',
          parentCid: 0,
          catLevel: 0,
          showStatus: 1,
          sort: 0,
          catId: null,
          icon: '',
          productUnit: ''
        },
        dialogType: '',
        title: '',
        maxLevel: 0,
        updateNodes: []
      }
    },
    methods: {
      getMenus () {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'post'
        }).then(({data}) => {
          console.log('成功请求到接口：', data.data)
          this.menus = data.data
        })
      },
      submitData () {
        if (this.dialogType === 'add') {
          this.addCategory()
        }
        if (this.dialogType === 'edit') {
          this.editCategory()
        }
      },
      append (data) {
        console.log('append:', data)
        this.dialogType = 'add'
        this.title = '添加分类'
        this.dialogVisible = true
        // 添加赋值
        console.log('添加获取到的参数:', data)
        this.category.parentCid = data.catId
        this.category.catLevel = data.catLevel * 1 + 1
        this.category.catId = null
        this.category.name = ''
        this.category.icon = ''
        this.category.productUnit = ''
      },
      edit (data) {
        console.log('要修改的数据：', data)
        this.dialogType = 'edit'
        this.title = '修改分类'
        this.dialogVisible = true
        // 发送请求获取最新的数据
        this.$http({
          // url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
          url: this.$http.adornUrl('/product/category/info'),
          method: 'post',
          data: this.$http.adornData(data.catId, false)
        }).then(({data}) => {
          console.log('修改请求到的数据：', data)
          this.category.name = data.data.name
          this.category.catId = data.data.catId
          this.category.icon = data.data.icon
          this.category.productUnit = data.data.productUnit
          this.category.parentCid = data.data.parentCid
        })
      },
      addCategory () {
        console.log('添加菜单提交的数据', this.category)
        // 发送添加请求
        this.$http({
          url: this.$http.adornUrl('/product/category/save'),
          method: 'post',
          data: this.$http.adornData(this.category, false)
        }).then(({data}) => {
          this.$message({
            message: '恭喜您，菜单添加成功',
            type: 'success'
          })
          // 关闭对话框
          this.dialogVisible = false
          // 重新请求刷新
          this.getMenus()
          // 设置默认展开(删除节点的父节点id)
          this.expandedKey = [this.category.parentCid]
        })
      },
      editCategory () {
        var {catId, name, icon, productUnit} = this.category
        this.$http({
          url: this.$http.adornUrl('/product/category/update'),
          method: 'post',
          data: this.$http.adornData({catId, name, icon, productUnit}, false)
        }).then(({data}) => {
          this.$message({
            message: '恭喜您，菜单修改成功',
            type: 'success'
          })
          // 关闭对话框
          this.dialogVisible = false
          // 重新请求刷新
          this.getMenus()
          // 设置默认展开(删除节点的父节点id)
          this.expandedKey = [this.category.parentCid]
        })
      },

      remove (node, data) {
        console.log('remove:', node, data)
        let ids = [data.catId]
        // 弹框提示是否删除
        this.$confirm(`请确认是否删除【${data.name}】菜单?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // 确认删除
          // 请求逻辑删除
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            this.$message({
              message: '恭喜您，菜单删除成功',
              type: 'success'
            })
            // 重新请求刷新
            this.getMenus()
            // 设置默认展开(删除节点的父节点id)
            this.expandedKey = [node.parent.data.catId]
          })
        }).catch(() => {

        })
      },
      allowDrop (draggingNode, dropNode, type) {
        // 被拖动的当前节点以及所在的父节点总层数不能大于3
        // 1获取当前被拖动节点的总层数
        console.log('拖动：', draggingNode, dropNode, type)
        this.countNodeLevel(draggingNode.data)
        // 当前正在拖动的节点-父节点的深度<3即可
        let deep = this.maxLevel - draggingNode.data.catLevel + 1
        console.log('深度：', deep)
        // 如果是同一级别节点拖动
        if (type === 'inner') {
          return deep + dropNode.level <= 3
        } else {
          return deep + dropNode.parent.level <= 3
        }
      },
      countNodeLevel (node) {
        // 找到所有子节点，求出最大深度
        if (node.children != null && node.children.length > 0) {
          for (let i = 0; i < node.children.length; i++) {
            if (node.children[i].catLevel > this.maxLevel) {
              this.maxLevel = node.children[i].catLevel
            }
            // 递归遍历
            this.countNodeLevel(node.children[i])
          }
        }
      },
      //
      handleDrop (draggingNode, dropNode, dropType, ev) {
        console.log('handleDrop: ', draggingNode, dropNode, dropType)
        // 1,当前节点最新的父节点id
        let pCid = 0
        let siblings = null
        if (dropType === 'before' || dropType === 'after') {
          pCid = dropNode.parent.data.catId
          siblings = dropNode.parent.childNodes
        } else {
          pCid = dropNode.data.catId
          siblings = dropNode.childNodes
        }

        // 2,当前拖拽节点的最新排序
        for (let i = 0; i < siblings.length; i++) {
          if (siblings[i].data.catId === draggingNode.data.catId) {
            // 如果遍历的正是当前拖拽的节点
            this.updateNodes.push({catId: siblings[i].data.catId, sort: i, parentCid: pCid})
          } else {
            this.updateNodes.push({catId: siblings[i].data.catId, sort: i})
          }
        }

        // 3,当前拖拽节点的最新层级

        console.log('updateNodes=', this.updateNodes)
      }
    },
    created () {
      this.getMenus()
    }
  }
</script>

<style lang="scss" scoped>

</style>