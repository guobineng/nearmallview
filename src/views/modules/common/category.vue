<template>
  <el-tree :data="menus" 
    :props="defaultProps" 
    node-key="catId"
    ref="menuTree"
    @node-click="nodeclick"
    >
    </el-tree>
</template>

<script>
export default {

  components: { },

  data() {
    return {
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },

  mounted() {
    this.getMenus()
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
    nodeclick (data, node, component) {
      console.log('子组件category的节点被点击', data, node, component)
      // 向父组件发送事件
      this.$emit('tree-node-click', data, node, component);
    }
  },
};
</script>

<style lang="scss" scoped>

</style>