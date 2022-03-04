<template>
  <div>
    <el-form ref="form" label-width="80px" :model="spu">
      <el-form-item label="spu名称">
        <el-input placeholder="spu名称" v-model="spu.spuName"></el-input>
      </el-form-item>
      <el-form-item label="品牌">
        <el-select placeholder="请选择品牌" v-model="spu.tmId">
          <el-option
            :label="tm.tmName"
            :value="tm.id"
            v-for="(tm, index) in tradeMarkList"
            :key="(tm.id, index)"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="SPU描述">
        <el-input
          type="textarea"
          rows="4"
          placeholder="描述"
          v-model="spu.description"
        ></el-input>
      </el-form-item>
      <el-form-item label="SPU图片">
        <el-upload
          action="/dev-api/admin/product/fileUpload"
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :file-list="spuImageList"
          :on-success="handlerSuccess"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="" />
        </el-dialog>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-select
          :placeholder="`还有${unSelectSaleAttr.length}未选择`"
          v-model="attrIdAndAttrName"
        >
          <el-option
            :label="unselect.name"
            :value="`${unselect.id}:${unselect.name}`"
            v-for="(unselect, index) in unSelectSaleAttr"
            :key="(unselect.id, index)"
          ></el-option>
        </el-select>
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!attrIdAndAttrName"
          @click="addSaleAttr"
          >添加销售属性</el-button
        >
        <el-table style="width: 100%" border :data="spu.spuSaleAttrList">
          <el-table-column
            type="index"
            label="序号"
            width="80px"
            align="center"
          ></el-table-column>
          <el-table-column
            prop="saleAttrName"
            label="属性值"
            width="width"
          ></el-table-column>
          <el-table-column prop="prop" label="属性值名称列表" width="width">
            <template slot-scope="{ row, $index }">
              <el-tag
                :key="(tag.id, index)"
                v-for="(tag, index) in row.spuSaleAttrValueList"
                closable
                :disable-transitions="false"
                @close="row.spuSaleAttrValueList.splice(index, 1)"
              >
                {{ tag.saleAttrValueName }}
              </el-tag>
              <el-input
                class="input-new-tag"
                v-if="row.inputVisible"
                v-model="row.inputValue"
                ref="saveTagInput"
                size="small"
                @blur="handleInputConfirm(row)"
              >
              </el-input>
              <el-button
                v-else
                class="button-new-tag"
                size="small"
                @click="addSaleAttrValue(row)"
                >添加</el-button
              >
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                @click="spu.spuSaleAttrList.splice($index, 1)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateSpu">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "SpuForm",
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      //spu初始化的时候是一个空对象
      spu: {
        category3Id: 0,
        description: "",
        id: 0,
        spuImageList: [
          {
           // id: 0,
          //  imgName: "",
           // imgUrl: "",
           // spuId: 0,
          },
        ],
        spuName: "",
        spuSaleAttrList: [
          //{
           // baseSaleAttrId: 0,
           // id: 0,
           // saleAttrName: "",
          //  spuId: 0,
           // spuSaleAttrValueList: [
           //   {
           //     baseSaleAttrId: 0,
           //     id: 0,
           //     isChecked: "",
           ///     saleAttrValueName: "",
           //    spuId: 0,
              //},
            //],
          //},
        ],
        tmId: "",
      }, //存储SPU信息的属性
      tradeMarkList: [], //存储品牌信息
      spuImageList: [], //存储图片的信息
      saleAttrList: [], //存储平台的全部属性
      attrIdAndAttrName: "", //收集未选择的销售属性的ID---
    };
  },
  methods: {
    handleRemove(file, fileList) {
      //file参数：代表的是删除那张图片
      //fileList照片墙删除某一张图片后，其他剩余的图片
      //收集照片墙的数据
      //对于已有的图片「照片墙中显示的图片，有name URL字段，因为照片墙显示数据必须要这两个属性」
      //对于服务器而言，不需要name，URL字段将来对于有的图片的数据在提交给服务器的时候需要处理
      this.spuImageList = fileList;
    },
    //照片墙图片预览的回调
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    //初始化spuForm数据
    async initSpuData(spu) {
      //获取SPIU信息的数据
      let spuResult = await this.$API.spu.reqSpu(spu.id);
      if (spuResult.code == 200) {
        //在修改spu的时候，需要向服务器发请求的，把服务器返回的数据（对象），赋值给spu属性
        this.spu = spuResult.data;
      }
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      if (tradeMarkResult.code == 200) {
        this.tradeMarkList = tradeMarkResult.data;
      }
      //获取spu图片的数据
      let spuImageResult = await this.$API.spu.reqSpuImageList(spu.id);
      if (spuImageResult.code == 200) {
        let listArr = spuImageResult.data;
        //由于照片墙显示图片的数据需要数组，数组里面的元素需要用name,url字段
        //需要把服务器返回的数据进行修改
        listArr.forEach((item) => {
          item.name = item.imgName;
          item.url = item.imgUrl;
        });
        //把整理好的数据赋值给服务器
        this.spuImageList = listArr;
      }
      //获取平台全部的销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.saleAttrList = saleResult.data;
      }
    },
    //文件上传成功的回调
    handlerSuccess(response, file, fileList) {
      this.spuImageList = fileList;
    },
    addSaleAttr() {
      //已经收集需要添加的属性的信息
      //把收集到的销售属性数据进行分割
      const [baseSaleAttrId, saleAttrName] = this.attrIdAndAttrName.split(":");
      //向SPU对象的spuSaleAttrList数组里面添加新的销售属性
      let newSaleAttr = {
        baseSaleAttrId,
        saleAttrName,
        spuSaleAttrValueList: [],
      };
      //添加新的销售属性
      this.spu.spuSaleAttrList.push(newSaleAttr);
      //清空数据
      this.attrIdAndAttrName = "";
    },
    //添加按钮的回调
    addSaleAttrValue(row) {
      //点击销售属性值当中添加按钮的时候，需要有button变为input,通过当前销售属性的inputVisible控制
      //挂载在销售属性身上的响应式数据inputVisible  控制着button与input切换
      this.$set(row, "inputVisible", true);
      //通过响应式数据inputValue字段收集新增的销售属性值
      this.$set(row, "inputValue", "");
    },
    //失去焦点的事件
    handleInputConfirm(row) {
      //修改这个字段的值为false
      //结构出销售属性里面的值
      const { baseSaleAttrId, inputValue } = row;
      //属性值不能为空
      if (inputValue.trim() == "") {
        this.$message("属性值不能为空");
        return;
      }
      //属性值不能重复
      let result = row.spuSaleAttrValueList.every(
        (item) => item.saleAttrValueName != inputValue
      );
      if (!result) return;
      //新增的销售属性值
      let newSaleAttrValue = { baseSaleAttrId, saleAttrValueName: inputValue };
      row.spuSaleAttrValueList.push(newSaleAttrValue);
      row.inputVisible = false;
    },
    //保存按钮的回调（有可能是添加或者是修改）
    async addOrUpdateSpu() {
      //需要整理照片墙的参数
      //携带参数：对于图片，需要携带imaageName与imageUrl字段
      this.spu.spuImageList = this.spuImageList.map((item) => {
        return {
          imageName: item.name,
          imageUrl: (item.response && item.response.data) || item.url,
        };
      });
      let result = await this.$API.spu.reqAddOrUpdateSpu(this.spu);
      if (result.code == 200) {
        //提示
        this.$message({ type: "success", message: "保存成功" });
        //通知父组件回到场景为0
        this.$emit("changeScene", {scene:0,flag:this.spu.id?'修改':'添加'});
        //添加｜修改的保存
      }
      //清除数据
      Object.assign(this._data,this.$options.data())
    },
    //点击添加按钮的时候，发请求的函数
    async addSpuDate(category3Id) {
      //添加SPU的时候收集三级分类的ID
      this.spu.category3Id=category3Id
      let tradeMarkResult =await this.$API.spu.reqTradeMarkList();
      if (tradeMarkResult.code == 200) {
        this.tradeMarkList = tradeMarkResult.data;
      }
      let saleResult =await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.saleAttrList = saleResult.data;
      }
    },
    //取消按钮
    cancel(){
      //取消按钮的回调，通知父组件切换场景为0，
     this.$emit('changeScene', {scene:0,flag:''})
     //清理数据
      //Object.assign:es6中新增的方法，可以合并对象，把A和B合并
     //组件实例this._data，可以操作data当中的响应式数据
      //this.$options可以获取配置对象，配置对象的data函数执行，返回的响应式数据为空
     Object.assign(this._data,this.$options.data())
    }
  },
  computed: {
    //计算出还未选择的销售属性
    unSelectSaleAttr() {
      //找个平台的销售属性一共三个 颜色，尺寸，版本 saleAttrList
      //当前SPU拥有的属于自己的销售属性spu.spuSaleAttrList
      //数组的过滤方法，可以从已有的数组当中过滤出用户需要的元素，并返回一个新的数据
      let result = this.saleAttrList.filter((item) => {
        //every数组的方法，会返回一个布尔值「真 假」
        return this.spu.spuSaleAttrList.every((item1) => {
          return item.name != item1.saleAttrName;
        });
      });
      return result;
    },
  },
};
</script>

<style>
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>