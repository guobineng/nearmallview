<template>
  <el-tree :data="menus" 
    :props="defaultProps" 
    node-key="catId"
    ref="menuTree"
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
  },
};
</script>

<style lang="scss" scoped>

</style>