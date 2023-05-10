<!--
* @description  参数1
* @fileName  category
* @author niudengfeng
* @date 2021-04-02 09:51:38
* @version V1.0.0
!-->
<template>
  <div id="category">
    <el-tree
      :data="menus"
      node-key="catId"
      :props="defaultProps"
      ref="menuTree"
      @node-click="nodeClick"
    >
    </el-tree>
  </div>
</template>

<script>
export default {
  props: {
    // 父辈向子辈传参
  },
  name: "category",
  data() {
    return {
      menus: [],
      defaultProps: {
        children: "childrenList",
        label: "name",
      },
    };
  },
  created() {
    // 实例被创建之后执行代码
    this.getDataList();
  },
  computed: {
    // 计算属性
  },
  components: {
    // 组件的引用
  },
  methods: {
    nodeClick(data, node, component) {
      console.log("节点被点击了：", data, node, component);
      //向父节点发送通知
      this.$emit("tree-node-click",data, node, component);
    },
    // 方法
    getDataList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/menu/list"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.list;
        console.log("获取到数据：" + data);
      });
    },
  },
  mounted() {
    // 页面进入时加载内容
  },
  watch: {
    // 监测变化
  },
};
</script>
<style scoped>
</style>