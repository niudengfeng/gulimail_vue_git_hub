<!--
* @description  参数1
* @fileName  category
* @author niudengfeng
* @date 2021-03-25 16:23:40
* @version V1.0.0
!-->
<template>
  <div id="menu">
    <el-button type="danger" @click="batchDel">批量删除</el-button>
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expands"
      :props="defaultProps"
      :expand-on-click-node="false"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(node, data)"
          >
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(node, data)">
            Edit
          </el-button>
          <el-button
            type="text"
            size="mini"
            v-if="(!data.childrenList)||data.childrenList.length == 0"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <!-- Form -->
    <el-dialog
      :title="title"
      :visible.sync="dialogFormVisible"
      :close-on-click-modal="closeModelBoolean"
    >
      <el-form :model="categoryProduct">
        <el-form-item label="分类名称" :label-width="formLabelWidth">
          <el-input
            v-model="categoryProduct.name"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="排序" :label-width="formLabelWidth">
          <el-input
            v-model="categoryProduct.sort"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="计量单位" :label-width="formLabelWidth">
          <el-input
            v-model="categoryProduct.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="商品数量" :label-width="formLabelWidth">
          <el-input
            v-model="categoryProduct.productCount"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="save">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      closeModelBoolean: false,
      formLabelWidth: "200px",
      title: "",
      type: "",
      dialogFormVisible: false,
      categoryProduct: {
        //分类id
        catId: null,
        //分类名称
        name: "",
        //父分类id
        parentCid: 0,
        //层
        catLevel: 0,
        //排序
        sort: 0,
        //图标地址
        icon: null,
        //计量单位
        productUnit: "",
        //商品数量
        productCount: 0,
      },
      menus: [],
      expands: [],
      defaultProps: {
        children: "childrenList",
        label: "name",
      },
    };
  },
  methods: {
    //批量删除
    batchDel() {
      //获取选中节点数据
      var checkedNodes = this.$refs.menuTree.getCheckedNodes();
      let catIds = [];
      let catNames = [];
      let cIds = [];
      if (checkedNodes && checkedNodes.length > 0) {
        for (let i = 0; i < checkedNodes.length; i++) {
          catIds.push(checkedNodes[i].catId);
          catNames.push(checkedNodes[i].name);
          cIds.push(checkedNodes[i].parentCid);
        }

        this.$confirm(`确定对[名称=${catNames}]进行删除操作?`, "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        })
          .then(() => {
            this.$http({
              url: this.$http.adornUrl("/product/category/delete"),
              method: "post",
              data: this.$http.adornData(catIds, false),
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.$message({
                  message: "操作成功",
                  type: "success",
                  duration: 1500,
                });
                //重新获取数据
                this.getDataList();
                //展开所选节点的父节点
                this.expands = cIds;
              } else {
                this.$message.error(data.msg);
              }
            });
          })
          .catch(() => {});
      }
    },
    edit(node, data) {
      //根据菜单id查询菜单明细
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log("category:" + data.category);
        this.title = "编辑菜单";
        this.type = "edit";
        this.dialogFormVisible = true;
        this.categoryProduct.catId = data.category.catId;
        this.categoryProduct.name = data.category.name;
        this.categoryProduct.parentCid = data.category.parentCid;
        this.categoryProduct.catLevel = data.category.catLevel;
        this.categoryProduct.sort = data.category.sort;
        this.categoryProduct.icon = data.category.icon;
        this.categoryProduct.productUnit = data.category.productUnit;
        this.categoryProduct.productCount = data.category.productCount;
      });
    },
    getDataList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/menu/list"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.list;
      });
    },
    append(node, data) {
      console.log(node, data);
      this.title = "新增菜单";
      this.type = "add";
      this.dialogFormVisible = true;
      this.categoryProduct.catId = null;
      this.categoryProduct.name = "";
      this.categoryProduct.parentCid = data.catId; //当前节点对应的id
      this.categoryProduct.catLevel = data.catLevel * 1 + 1; //当前节点对应层级加1
      this.categoryProduct.sort = "";
      this.categoryProduct.icon = null;
      this.categoryProduct.productUnit = "";
      this.categoryProduct.productCount = "";
    },

    save() {
      this.dialogFormVisible = false;
      let requestUrl = "";
      if (this.type == "add") {
        requestUrl = "/product/category/save";
      } else if (this.type == "edit") {
        requestUrl = "/product/category/update";
      }
      this.$http({
        url: this.$http.adornUrl(requestUrl),
        method: "post",
        data: this.$http.adornData(this.categoryProduct, false),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "保存成功",
            type: "success",
            duration: 1500,
            onClose: () => {
              this.getDataList();
              this.expands = [this.categoryProduct.parentCid];
            },
          });
        } else {
          this.$message.error("保存失败");
        }
      });
    },

    remove(node, data) {
      let parentCid = data.parentCid;
      let param = [data.catId];
      this.$confirm(`确定对[${data.name}]进行删除操作?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(param, false),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.getDataList();
                  this.expands = [parentCid];
                },
              });
            } else {
              this.$message.error("操作失败");
            }
          });
        })
        .catch(() => {});
    },
  },
  props: {
    // 父辈向子辈传参
  },
  name: "category",

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
