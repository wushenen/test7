<template>
  <div>
    <!--按钮-->
    <el-button
      type="primary"
      icon="el-icon-plus"
      style="margin: 10px 0px"
      @click="showDialog"
      >添加</el-button
    >
    <!--表格组件
    data：表格组件将来需要展示的数据----数组类型
    boder:给表格添加边框
    label:显示标题
    width:对应列的宽度
    align:标题的对齐方式
    prop:对应列内容的字段名
    注意1:ElementUI当中table组件，展示的数据是以一列一列进行展示数据
    -->
    <el-table style="width: 100%" border :data="list">
      <el-table-column type="index" label="序号" width="80px" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称" width="width">
      </el-table-column>
      <el-table-column prop="logoUrl" label="品牌logo" width="width">
        <template slot-scope="{ row, $index }">
          <img :src="row.logoUrl" alt="" style="width: 100px; height: 100px" />
        </template>
      </el-table-column>
      <el-table-column prop="prop" label="操作" width="width">
        <template slot-scope="{ row, $index }">
          <el-button
            type="warning"
            icon="el-icon-edit"
            size="mini"
            @click="updateTradeMark(row)"
            >修改</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-edit"
            size="mini"
            @click="deleteTradeMark(row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!--分页器
    当前第几页，数据总条数，每一页展示的数据，连续页码
    
     layout:可以实现分页器布局
     page-count:按钮的数量
    -->
    <el-pagination
      style="margin-top: 20px; textalign: center"
      :current-page="page"
      :total="total"
      :page-size="limit"
      :pager-count="7"
      :page-sizes="[3, 5, 10]"
      @current-change="getPageList"
      @size-change="handlesSizeChange"
      layout="prev,pager,next,jumper,->,sizes,total"
    >
    </el-pagination>
    <!--对话框
    ：visible.sync:控制对话框与隐藏用的
    Form 组件提供了表单验证的功能，只需要通过 rules 属性传入约定的验证规则，并将 Form-Item 的 prop 属性设置为需校验的字段名即可。
    -->
    <el-dialog
      :title="tmForm.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <!--form表单 :model属性：这个属性的作用是把表单的数据收集到哪个对象的身上。将来表单验证也需要这个属性-->
      <el-form style="width: 80%" :model="tmForm" :rules="rules" ref="ruleForm">
        <el-form-item label="品牌名称" label-width="100px" prop="tmName">
          <el-input autocomplete="off" v-model="tmForm.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌LOGO" label-width="100px" prop="logoUrl">
          <!--这里收集数据：不能使用v-model，因为不是表单元素
           action：设置图片上传的地址
           ：on-success:可以检测到图片上传成功会执行一次
           :before-upload：可以在上传图片之前会执行一次
           -->
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div slot="tip" class="el-upload__tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "tradeMark",
  data() {
    //自定义校验规
    var validateTmName = (rule, value, callback) => {
      if (value.length < 2 || value.length>10) {
        callback(new Error("品牌名称需要2-10位"));
      } else {
        callback();
      }
    };
    return {
      //代表的是分页器第几页
      page: 1,
      //当前页数展示数据的条数
      limit: 3,
      //总共条数
      total: 0,
      //列表展示的数据
      list: [],
      //对话框显示与隐藏控制的属性
      dialogFormVisible: false,
      //收集品牌的信息
      tmForm: {
        tmName: "",
        logoUrl: "",
      },
      //表单验证规则
      rules: {
        //品牌名称的验证规则
        tmName: [
          //required:必须要校验的字段（和前面*号有关） message：提升的信息  trigge:用户行为设置（时间的设置：blur,change）
          { required: true, message: "请输入品牌名称", trigger: "blur" },
          //min-max：长度最小2-10  自定义验证规则
          { validator: validateTmName, trigger: "change" },
        ],
        //品牌图片的验证规则
        logoUrl: [{ required: true, message: "请选择品牌的图片" }],
      },
    };
  },
  //组件挂载完毕发请求
  mounted() {
    //获取列表数据方法
    this.getPageList();
  },
  methods: {
    //获取品牌列表的数据
    async getPageList(pager = 1) {
      this.page = pager;
      //结构出参数
      const { page, limit } = this;
      //获取品牌列表的接口
      //在data中初始化两个字段，代表给服务器传递参数
      let result = await this.$API.trademark.reqTradeMarkList(page, limit);
      if (result.code == 200) {
        //分别是展示数据的条数与列表展示的数据
        this.total = result.data.total;
        this.list = result.data.records;
      }
    },
    //当分页器某一页需要展示数据条数发生变化的时候会触发
    handlesSizeChange(limit) {
      this.limit = limit;
      this.getPageList();
    },
    //点击添加品牌的按钮
    showDialog() {
      //显示对话框
      this.dialogFormVisible = true;
      //清空数据
      this.tmForm = { tmName: " ", logoUrl: " " };
    },
    //修改某个品牌
    updateTradeMark(row) {
      //row:当前用户选中的这个品牌信息
      //显示对话框
      this.dialogFormVisible = true;
      //将已有的品牌信息赋值给tmForm进行展示
      //将服务器返回的品牌信息赋值给了tmForm进行展示
      //也就是说teForm存储即为服务器返回的品牌信息
      this.tmForm = { ...row };
    },
    //上传图片成功的回调
    handleAvatarSuccess(res, file) {
      //res:上传成功之后返回前段数据
      //file：上传成功之后服务器返回前段的数据
      //收集品牌图片数据：因为将来需要带给服务器
      this.tmForm.logoUrl = res.data;
    },
    //图片上传之前到回调
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    //添加按钮（添加品牌｜修改品牌）
    addOrUpdateTradeMark() {
      //当全部字段验证通过，再去写业务逻辑
      this.$refs.ruleForm.validate(async (success) => {
        //如果全部字段符合验证
        if (success) {
          this.dialogFormVisible = false;
          //发请求(添加品牌｜修改品牌)
          let result = await this.$API.trademark.reqAddOrUpdateTradeMark(
            this.tmForm
          );
          if (result.code == 200) {
            //弹出信息；添加品牌成功｜修改品牌成功
            this.$message({
              type: "success",
              message: this.tmForm.id ? "修改品牌成功" : "添加品牌成功",
            });
            //成功以后，需要再次获取进行展示
            //如果添加品牌：停留在第一页，修改品牌停留在当前页
            this.getPageList(this.tmForm.id ? this.page : 1);
          } else {
            console.log("error submit!!");
            return false;
          }
        }
      });
    },
    //删除品牌的操作
    deleteTradeMark(row) {
      //弹窗
      this.$confirm(`你确定删除${row.tmName}吗`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async() => {
          //当用户点击确定触发
          //想服务器发请求
        let result=await this.$API.trademark.reqDeleteTradeMark(row.id)
          if(result.code==200){
            this.$message({
            type: "success",
            message: "删除成功!",
           });
           // 再次获取品牌列表
           this.getPageList(this.list.length>1?this.page:this.page-1)
          }
        })
        //当用户点击取消出发
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
};
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>