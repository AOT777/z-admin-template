<template>
  <div>
    <el-form label-width="80px" :data="spu">
      <el-form-item label="SPU名称">
        <el-input v-model="spu.spuName" placeholder="SPU名称" />
      </el-form-item>
      <el-form-item label="品牌">
        <el-select v-model="spu.tmId" placeholder="请选择品牌">
          <el-option v-for="tm in tradeMarkList" :key="tm.id" :label="tm.tmName" :value="tm.id" />
        </el-select>
      </el-form-item>
      <el-form-item label="SPU描述">
        <el-input v-model="spu.description" type="textarea" rows="4" placeholder="SPU描述" />
      </el-form-item>
      <el-form-item label="SPU图片">
        <el-upload
          action="/dev-api/admin/product/fileUpload"
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :file-list="spuImagesList"
          :on-success="handleSuccess"
        >
          <i class="el-icon-plus" />
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="">
        </el-dialog>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-select v-model="attrIdAndName" :placeholder="`还有${unSelectSaleAttr.length}个未选择`">
          <el-option v-for="unSaleSttr in unSelectSaleAttr" :key="unSaleSttr.id" :label="unSaleSttr.name" :value="`${unSaleSttr.id}:${unSaleSttr.name}`" />
        </el-select>
        <el-button type="primary" icon="el-icon-plus" :disabled="!attrIdAndName" @click="addSaleAttr">添加销售属性</el-button>
        <el-table
          style="width: 100%"
          border
          :data="spu.spuSaleAttrList"
        >
          <el-table-column
            type="index"
            label="序号"
            width="80px"
            align="center"
          />
          <el-table-column
            prop="saleAttrName"
            label="属性名"
            width="200px"
          />
          <el-table-column
            prop="prop"
            label="属性值名称列表"
            width="width"
          >
            <template slot-scope="{row}">
              <!--  @close="handleClose(tag)" -->
              <el-tag
                v-for="(tag,index) in row.spuSaleAttrValueList"
                :key="tag.id"
                closable
                :disable-transitions="false"
                @close="row.spuSaleAttrValueList.splice(index, 1)"
              >
                {{ tag.saleAttrValueName }}
              </el-tag>
              <!-- @keyup.enter.native="handleInputConfirm"
                @blur="handleInputConfirm" -->
              <el-input
                v-if="row.inputVisible"
                ref="saveTagInput"
                v-model="row.inputValue"
                class="input-new-tag"
                size="small"
                @blur="handleInputConfirm(row)"
                @keyup.enter.native="handleInputConfirm(row)"
              />
              <!-- @click="showInput" -->
              <el-button v-else class="button-new-tag" size="small" @click="addSaleAttrValue(row)">添加</el-button>
            </template>
          </el-table-column>
          <el-table-column
            prop="prop"
            label="操作"
            width="200px"
          >
            <template slot-scope="{$index}">
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="spu.spuSaleAttrList.splice($index,1)" />
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
  name: 'SpuForm',
  data() {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      spu: {
        // 三级分类的id
        category3Id: 0,
        // 描述
        description: '',
        // spu名称
        spuName: '',
        // 平台的id
        tmId: '',
        // 收集SPU图片的信息
        spuImageList: [
          // {
          //   id: 0,
          //   imgName: "string",
          //   imgUrl: "string",
          //   spuId: 0,
          // },
        ],
        // 平台属性与属性值收集
        spuSaleAttrList: [
          // {
          //   baseSaleAttrId: 0,
          //   id: 0,
          //   saleAttrName: "string",
          //   spuId: 0,
          //   spuSaleAttrValueList: [
          //     {
          //       baseSaleAttrId: 0,
          //       id: 0,
          //       isChecked: "string",
          //       saleAttrName: "string",
          //       saleAttrValueName: "string",
          //       spuId: 0,
          //     },
          //   ],
          // },
        ]
      },
      tradeMarkList: [],
      spuImagesList: [],
      spuSaleAttrList: [],
      attrIdAndName: ''
    }
  },
  computed: {
    unSelectSaleAttr() {
      const result = this.spuSaleAttrList.filter(item => {
        return this.spu.spuSaleAttrList.every(item1 => {
          // eslint-disable-next-line eqeqeq
          return item.name != item1.saleAttrName
        })
      })
      return result
    }
  },
  methods: {
    handleRemove(file, fileList) {
      // 除去的图片和剩下的图片
      // console.log(file, fileList)
      this.spuImagesList = fileList
    },
    handleSuccess(response, file, fileList) {
      this.spuImagesList = fileList
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    async initSpuData(spu) {
      // 1
      const spuResult = await this.$API.spu.reqSpu(spu.id)
      if (spuResult.code === 200) {
        this.spu = spuResult.data
      }
      // 2
      const tradeMarkResult = await this.$API.spu.reqTradeMarkList()
      if (tradeMarkResult.code === 200) {
        this.tradeMarkList = tradeMarkResult.data
      }
      // 3
      const SpuImagesListResult = await this.$API.spu.reqSpuImagesList(spu.id)
      if (SpuImagesListResult.code === 200) {
        const listArr = SpuImagesListResult.data
        // 照片墙需要规定的两属性才能显示(name,url)
        listArr.forEach(item => {
          item.name = item.imgName
          item.url = item.imgUrl
        })
        this.spuImagesList = listArr
      }
      // 4
      const spuSaleAttrListResult = await this.$API.spu.reqSpuSaleAttrList()
      if (spuSaleAttrListResult.code === 200) {
        this.spuSaleAttrList = spuSaleAttrListResult.data
      }
    },
    async addSpuData(category3Id) {
      this.spu.category3Id = category3Id
      const tradeMarkResult = await this.$API.spu.reqTradeMarkList()
      if (tradeMarkResult.code === 200) {
        this.tradeMarkList = tradeMarkResult.data
      }
      const spuSaleAttrListResult = await this.$API.spu.reqSpuSaleAttrList()
      if (spuSaleAttrListResult.code === 200) {
        this.spuSaleAttrList = spuSaleAttrListResult.data
      }
    },
    addSaleAttr() {
      const [baseSaleAttrId, saleAttrName] = this.attrIdAndName.split(':')
      // eslint-disable-next-line prefer-const
      let newSaleAttr = { baseSaleAttrId, saleAttrName, spuSaleAttrValueList: [] }
      this.spu.spuSaleAttrList.push(newSaleAttr)
      this.attrIdAndName = ''
    },
    addSaleAttrValue(row) {
      this.$set(row, 'inputVisible', true)
      this.$set(row, 'inputValue', '')
    },
    handleInputConfirm(row) {
      const { baseSaleAttrId, inputValue } = row
      if (inputValue.trim() === '') {
        this.$message('不能为空')
        return
      }
      // eslint-disable-next-line prefer-const
      let result = row.spuSaleAttrValueList.every(item => item.saleAttrValueName !== inputValue)
      if (!result) return
      // eslint-disable-next-line prefer-const
      let newSaleAttrValue = { baseSaleAttrId, saleAttrValueName: inputValue }
      row.spuSaleAttrValueList.push(newSaleAttrValue)
      row.inputVisible = false
    },
    async addOrUpdateSpu() {
      this.spu.spuImageList = this.spuImagesList.map((item) => {
        return {
          imgName: item.name,
          imgUrl: (item.response && item.response.data || item.url)
        }
      })
      const result = await this.$API.spu.reqAddOrUpdateSpu(this.spu)
      if (result.code === 200) {
        this.$message({ type: 'success', message: '保存成功' })
        this.$emit('changeScene', { scene: 0, flag: this.spu.id ? '修改' : '添加' })
      }
      Object.assign(this._data, this.$options.data())
    },
    cancel() {
      this.$emit('changeScene', { scene: 0 })
      Object.assign(this._data, this.$options.data())
    }
  }
}
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
