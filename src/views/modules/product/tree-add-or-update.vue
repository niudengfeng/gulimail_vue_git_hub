<template>
  <el-dialog
    title="新增"
    :close-on-click-modal="false"
    :visible.sync="visible"
    @closed="close"
    width="30%"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="100px"
    >
      <el-form-item label="所属分类id" prop="catelogId">
        <el-cascader
          v-model="dataForm.catelogIdPathArray"
          :options="categorys"
          :props="props"
          filterable
        ></el-cascader>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      categoryBrandRelation: {
        brandId: 0,
        catelogId: 0,
      },
      brandId: 0,
      categorys: [],
      visible: false,
      props: {
        label: "name",
        value: "catId",
        children: "childrenList",
      },
      dataForm: {
        catelogId: 0,
        catelogIdPathArray: [],
        catelogIdPath: "",
      },
      dataRule: {
        catelogIdPathArray: [
          { required: true, message: "所属分类id不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    close() {
      this.dataForm.catelogIdPathArray = [];
    },
    getTreeList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/menu/list"),
        method: "get",
      }).then(({ data }) => {
        this.categorys = data.list;
      });
    },
    init(brandId) {
      this.visible = true;
      this.brandId = brandId;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          let categoryId = this.dataForm.catelogIdPathArray[
            this.dataForm.catelogIdPathArray.length - 1
          ];
          this.categoryBrandRelation.brandId = this.brandId;
          this.categoryBrandRelation.catelogId = categoryId;
          this.$http({
            url: this.$http.adornUrl("/product/categorybrandrelation/save"),
            method: "post",
            data: this.$http.adornData(this.categoryBrandRelation, false),
          }).then(({ data }) => {
            if (data && data.code == 0) {
              console.log("保存成功", data);
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
              });
              this.visible = false;
              this.$emit("refreshRelationTable",this.brandId);
            }
          });
        }
      });
    },
  },
  created() {
    this.getTreeList();
  },
};
</script>
