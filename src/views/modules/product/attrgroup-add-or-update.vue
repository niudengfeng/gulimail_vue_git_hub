<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    @closed="close"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="180px"
    >
      <el-form-item label="组名" prop="attrGroupName">
        <el-input
          v-model="dataForm.attrGroupName"
          placeholder="组名"
        ></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model="dataForm.sort" placeholder="排序"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="descript">
        <el-input v-model="dataForm.descript" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="组图标" prop="icon">
        <!-- <el-input v-model="dataForm.icon" placeholder="组图标"></el-input> -->
        <single-upload v-model="dataForm.icon"></single-upload>
      </el-form-item>
      <el-form-item label="所属分类id" prop="catelogIdPath">
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
import singleUpload from "@/views/modules/upload/singleUpload";
export default {
  data() {
    return {
      categorys: [],
      visible: false,
      props: {
        label: "name",
        value: "catId",
        children: "childrenList",
      },
      dataForm: {
        attrGroupId: 0,
        attrGroupName: "",
        sort: "",
        descript: "",
        icon: "",
        catelogId: 0,
        catelogIdPathArray: [],
        catelogIdPath: "",
      },
      dataRule: {
        attrGroupName: [
          { required: true, message: "组名不能为空", trigger: "blur" },
        ],
        sort: [{ required: true, message: "排序不能为空", trigger: "blur" }],
        descript: [
          { required: true, message: "描述不能为空", trigger: "blur" },
        ],
        // icon: [{ required: true, message: "组图标不能为空", trigger: "blur" }],
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
    init(id) {
      this.dataForm.attrGroupId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.attrGroupId) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/attrgroup/info/${this.dataForm.attrGroupId}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.attrGroupName = data.attrGroup.attrGroupName;
              this.dataForm.sort = data.attrGroup.sort;
              this.dataForm.descript = data.attrGroup.descript;
              this.dataForm.icon = data.attrGroup.icon;
              this.dataForm.catelogId = data.attrGroup.catelogId;
              let arr = data.attrGroup.catelogIdPath.split(",");
              let numArr = arr.map((item, i) => {
                return parseInt(item);
              });
              this.dataForm.catelogIdPathArray = numArr;
              this.dataForm.catelogIdPath = data.attrGroup.catelogIdPath;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/attrgroup/${
                !this.dataForm.attrGroupId ? "save" : "update"
              }`
            ),
            method: "post",
            data: this.$http.adornData({
              attrGroupId: this.dataForm.attrGroupId || undefined,
              attrGroupName: this.dataForm.attrGroupName,
              sort: this.dataForm.sort,
              descript: this.dataForm.descript,
              icon: this.dataForm.icon,
              catelogId: this.dataForm.catelogIdPathArray[
                this.dataForm.catelogIdPathArray.length - 1
              ],
              catelogIdPath: this.dataForm.catelogIdPathArray.join(","),
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
  components: {
    // 组件的引用
    singleUpload,
  },
  created() {
    this.getTreeList();
  },
};
</script>
