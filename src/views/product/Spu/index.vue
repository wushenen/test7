<template>
  <div>
    <el-card style="margin: 20px 0px">
      <CategorySelect
        @getCategoryId="getCategoryId"
        :show="scene != 0"
      ></CategorySelect>
    </el-card>
    <el-card>
      <!--底部这里有三部分进行切换-->
      <div v-show="scene == 0">
        <!--展示spu列表的结构-->
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addSpu"
          >添加SPU</el-button
        >
        <el-table style="width: 100%" border :data="records">
          <el-table-column
            type="index"
            label="序号"
            width="80"
            align="center"
          ></el-table-column>
          <el-table-column
            prop="spuName"
            label="spu名称"
            width="width"
          ></el-table-column>
          <el-table-column
            prop="description"
            label="spu描述"
            width="width"
          ></el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <!--这里的按钮将来用hintButton替代-->
              <hint-button
                type="success"
                icon="el-icon-plus"
                size="mini"
                title="添加sku"
                @click="addSku(row)"
              ></hint-button>
              <hint-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                title="修改spu"
                @click="updateSpu(row)"
              ></hint-button>
              <hint-button
                type="info"
                icon="el-icon-info"
                size="mini"
                title="查看当前全部spu的sku列表"
                @click="handler(row)"
              ></hint-button>
              <el-popconfirm
                title="这是一段内容确定删除吗？"
                @onConfirm="deleteSpu(row)"
              >
                <hint-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  title="删除spu"
                  slot="reference"
                ></hint-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          style="text-align: center"
          @size-change="handleSizechange"
          @current-change="getSpuList"
          :current-page="page"
          :page-size="limit"
          :page-sizes="[3, 5, 10]"
          :total="total"
          layout="prev,pager,next,jumper,->,sizes,total"
        >
        </el-pagination>
      </div>
      <SpuForm
        v-show="scene == 1"
        @changeScene="changeScene"
        ref="spu"
      ></SpuForm>
      <SkuForm
        v-show="scene == 2"
        ref="sku"
        @changeScenes="changeScenes"
      ></SkuForm>
    </el-card>
    <el-dialog :title="`${spu.spuName}的sku列表`" :visible.sync="dialogTableVisible" :before-close="close">
   <el-table :data="skuList" style="width:100%" border="" v-loading="loading">
    <el-table-column prop="skuName" label="名称" width="width"></el-table-column>
    <el-table-column prop="price" label="价格" width="width"></el-table-column>
    <el-table-column prop="weight" label="重量" width="width"></el-table-column>
    <el-table-column label="默认图片" width="width">
      <template slot-scope="{row,$index}">
        <img :src="row.skuDefaultImg" alt="" style="width:100px;heigth:100px">
      </template>
    </el-table-column>
  </el-table>
    </el-dialog>
  </div>
</template>

<script>
import SpuForm from "./SpuForm";
import SkuForm from "./SkuForm";
export default {
  name: "Spu",
  components: { SkuForm, SpuForm },
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "",
      page: 1, //分页器当前第几页
      limit: 3, //每一页展示多少条数据
      records: [], //存储spu列表的数据
      total: 0, //分页器一共需要展示数据的条数
      scene: 0, //0代表的是展示SPu列表的数据  1:添加｜修改SPU    2:添加SKU
      //控制对话框的显示与隐藏
      dialogTableVisible:false,
      //保存SPU的信息
      spu:{},
      //存储的是SKU列表的数据
      skuList:[],
      loading:true
    };
  },
  methods: {
    //三级联动的自定义事件，哭吧子组件的相应的ID传递给父组件
    getCategoryId({ categoryId, level }) {
      //categoryId：可以获取到一，二，三级分类的id，level：为了区分三几级id
      if (level == 1) {
        this.category1Id = categoryId;
        //清除二级三级分类的id
        this.category2Id = "";
        this.category3Id = "";
      } else if (level == 2) {
        this.category2Id = categoryId;
        this.category3Id = "";
      } else {
        this.category3Id = categoryId;
        //获取SPU列表进行展示
        this.getSpuList();
      }
    },
    //获取spu数据的方法
    async getSpuList(pages = 1) {
      this.page = pages;
      const { page, limit, category3Id } = this;
      //需要携带三个参数：page 第几页  limit  每一页需要展示多少条数据  catagory3Id:三级分类的ID
      let result = await this.$API.spu.reqSpuList(page, limit, category3Id);
      if (result.code == 200) {
        this.total = result.data.total;
        this.records = result.data.records;
      }
    }, //当分页器的某一个展示数据条数发生变化的回调
    handleSizechange(limit) {
      this.limit = limit;
      this.getSpuList();
    },
    //添加SPU的回调
    addSpu() {
      this.scene = 1;
      //通知子组件发请求两个
      this.$refs.spu.addSpuDate(this.category3Id);
    },
    //修改某一个SPU
    updateSpu(row) {
      this.scene = 1;
      //获取子组件spuForm
      //在父组件当中，可以通过$ref获取子组件
      this.$refs.spu.initSpuData(row);
    },
    //自定义事件的回调（SpuForm取消按钮）
    changeScene({ scene, flag }) {
      //切换结构（）
      this.scene = scene;
      //flag这个行参是为了区分保存按钮是添加还是修改

      //子组件通知父组件切换场景，需要再次获取SPU列表的数据进行展示
      if (flag == "修改") {
        this.getSpuList(this.page);
      } else {
        this.getSpuList();
      }
    },
    //删除SPU的操作
    async deleteSpu(row) {
      let result = await this.$API.spu.reqDeleteSpu(row.id);
      if (result.code == 200) {
        this.$message({ type: "success", message: "删除成功" });
        this.getSpuList(this.records.length > 1 ? this.page : this.page - 1);
      }
    },
    //添加SKU的回调
    addSku(row) {
      this.scene = 2;
      //父组件调用子组件的方法，让子组件发请求---三个请求
      this.$refs.sku.getData(this.category1Id, this.category2Id, row);
    },
    //skuForm通知父组件修改场景
    changeScenes(scene) {
      this.scene = scene;
    },
    //点击查看SKU列表的回调
   async handler(spu){
     //点击按钮的时候,显示对话框
     this.dialogTableVisible=true
     this.spu=spu
     //发请求
     let result=await this.$API.spu.reqSkuList(spu.id)
     if(result.code==200){
       this.skuList=result.data
       this.loading=false
     }
    },
    //关闭对话框的回调
    close(done){
     //loading属性再次变为真
     this.loading=true
     //清除sku列表的数据
     this.skuList=[]
     //关闭对话框
     done()
    }
  },
};
</script>

<style>
</style>