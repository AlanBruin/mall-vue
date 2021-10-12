<template>
  <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
let id = 1000
export default {
  name: 'category',
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  mounted () {
    this.getProduct()
  },
  methods: {
    getProduct () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        console.log(data.page)
        this.menus = data.page
      })
    }
  }
}
</script>

<style>

</style>
