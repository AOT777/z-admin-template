<template>
  <div>
    <el-form ref="form" label-width="80px">
      <el-form-item label="SPU名称">
        {{ spu.spuName }}
      </el-form-item>
      <el-form-item label="SKU名称">
        <el-input v-model="skuInfo.skuName" placeholder="SKU名称" />
      </el-form-item>
      <el-form-item label="价格(元)">
        <el-input v-model="skuInfo.price" type="number" placeholder="价格(元)" />
      </el-form-item>
      <el-form-item label="重量(千克)">
        <el-input v-model="skuInfo.weight" placeholder="重量(千克)" />
      </el-form-item>
      <el-form-item label="规格描述">
        <el-input v-model="skuInfo.skuDesc" type="textarea" rows="4" placeholder="规格描述" />
      </el-form-item>
      <el-form-item label="平台属性">
        <el-form :inline="true" label-width="80px">
          <el-form-item v-for="attr in attrInfoList" :key="attr.id" :label="attr.attrName">
            <el-select v-model="attr.attrIdAndValue" placeholder="请选择">
              <el-option v-for="attrValue in attr.attrValueList" :key="attrValue.id" :label="attrValue.valueName" :value="`${attr.id}:${attrValue.id}`" />
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-form :inline="true" label-width="80px">
          <el-form-item v-for="spuSaleAttr in spuSaleAttrList" :key="spuSaleAttr.id" :label="spuSaleAttr.saleAttrName">
            <el-select v-model="spuSaleAttr.attrIdAndValue" placeholder="请选择">
              <el-option v-for="spuSaleAttrValue in spuSaleAttr.spuSaleAttrValueList" :key="spuSaleAttrValue.id" :label="spuSaleAttrValue.saleAttrValueName" :value="`${spuSaleAttr.id}:${spuSaleAttrValue.id}`" />
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="图片列表">
        <el-table
          style="width: 100%"
          border
          :data="spuImagesList"
          @selection-change="handleSelectionChange"
        >
          <el-table-column
            type="selection"
            width="80px"
            align="center"
          />
          <el-table-column
            prop="prop"
            label="图片"
            width="width"
          >
            <template slot-scope="{row}">
              <img :src="row.imgUrl" style="width:80px;height:80px">
            </template>
          </el-table-column>
          <el-table-column
            prop="imgName"
            label="名称"
            width="width"
          />
          <el-table-column
            prop="prop"
            label="操作"
            width="width"
          >
            <template slot-scope="{row}">
              <el-button v-if="row.isDefault===0" type="primary" size="mini" @click="changeDefault(row)">设为默认</el-button>
              <el-button v-else type="success" size="mini">默认</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="saveSku">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: 'SkuForm',
  data() {
    return {
      spuImagesList: [],
      spuSaleAttrList: [],
      attrInfoList: [],
      skuInfo: {
        // 第一类父组件给的
        category3Id: 0,
        spuId: 0,
        tmId: 0,
        // 第二类简单绑定
        price: 0,
        weight: '',
        skuName: '',
        skuDesc: '',
        // 第三类需要手写的
        // 平台属性
        skuAttrValueList: [
          /*  {
            attrId: 0,
            valueId: 0,
          }, */
        ],
        skuDefaultImg: '',
        skuImageList: [
          /* {
            imgName: "string",
            imgUrl: "string",
            isDefault: "string",
            spuImgId: 0,
          }, */
        ],
        // 销售属性
        skuSaleAttrValueList: [
          /* {
            saleAttrId: 0,
            saleAttrValueId: 0,
          }, */
        ]
      },
      spu: {},
      imgList: []
    }
  },
  methods: {
    async getSkuData(row, category1Id, category2Id, category3Id) {
      this.skuInfo.category3Id = category3Id
      this.skuInfo.spuId = row.id
      this.skuInfo.tmId = row.tmId
      this.spu = row
      // 1
      const spuImagesListResult = await this.$API.spu.reqSpuImagesList(row.id)
      if (spuImagesListResult.code === 200) {
        // eslint-disable-next-line prefer-const
        let list = spuImagesListResult.data
        // 返回数据缺少一个，自己补上
        list.forEach(item => {
          item.isDefault = 0
        })
        this.spuImagesList = list
      }
      // 2
      const spuSaleAttrListResult = await this.$API.spu.reqSpuSaleAttrLists(row.id)
      if (spuSaleAttrListResult.code === 200) {
        this.spuSaleAttrList = spuSaleAttrListResult.data
      }
      // 3
      const attrInfoListResult = await this.$API.spu.reqAttrInfoList(category1Id, category2Id, category3Id)
      if (attrInfoListResult.code === 200) {
        this.attrInfoList = attrInfoListResult.data
      }
    },
    // 选中的图片
    handleSelectionChange(selection) {
      this.imgList = selection
    },
    // 改变默认图片
    changeDefault(row) {
      // 排他
      this.spuImagesList.forEach(item => {
        item.isDefault = 0
      })
      row.isDefault = 1
      this.skuInfo.skuDefaultImg = row.imgUrl
    },
    cancel() {
      this.$emit('changeScene2', 0)
      Object.assign(this._data, this.$options.data())
    },
    async saveSku() {
      const { attrInfoList, spuSaleAttrList, skuInfo, imgList } = this
      // 1平台属性
      skuInfo.skuAttrValueList = attrInfoList.reduce((prev, item) => {
        if (item.attrIdAndValue) {
          const [attrId, valueId] = item.attrIdAndValue.split(':')
          prev.push({ attrId, valueId })
        }
        return prev
      }, [])
      // 2销售属性
      skuInfo.skuSaleAttrValueList = spuSaleAttrList.reduce((prev, item) => {
        if (item.attrIdAndValue) {
          const [saleAttrId, saleAttrValueId] = item.attrIdAndValue.split(':')
          prev.push({ saleAttrId, saleAttrValueId })
        }
        return prev
      }, [])
      // 3图片数据
      skuInfo.skuImageList = imgList.map(item => {
        return {
          imgName: item.imgName,
          imgUrl: item.imgUrl,
          isDefault: item.isDefault,
          spuImgId: item.id
        }
      })
      // 4请求
      const result = await this.$API.spu.reqAddSku(skuInfo)
      if (result.code === 200) {
        this.$message({ type: 'success', message: '保存成功' })
        this.$emit('changeScene2', 0)
        Object.assign(this._data, this.$options.data())
      }
    }
  }
}
</script>

<style>

</style>
