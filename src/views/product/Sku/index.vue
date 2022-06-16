/* eslint-disable require-atomic-updates */
<template>
  <div>
    <el-card>
      <el-table
        style="width: 100%"
        border
        :data="skuList"
      >
        <el-table-column type="index" label="序号" width="80" align="center" />
        <el-table-column prop="skuName" label="名称" width="width" />
        <el-table-column prop="skuDesc" label="描述" width="width" />
        <el-table-column prop="prop" label="默认图片" width="110" align="center">
          <template slot-scope="{row}">
            <img :src="row.skuDefaultImg" alt="" style="width:80px;height:80px">
          </template>
        </el-table-column>
        <el-table-column prop="weight" label="重量(KG)" width="110" />
        <el-table-column prop="price" label="价格(元)" width="110" />
        <el-table-column prop="prop" label="操作" width="width">
          <template slot-scope="{row}">
            <HintButton v-if="row.isSale==0" title="下架状态" type="success" size="mini" icon="el-icon-bottom" @click="onSale(row)" />
            <HintButton v-else title="上架状态" type="info" size="mini" icon="el-icon-top" @click="cancelSale(row)" />
            <HintButton title="编辑" type="primary" size="mini" icon="el-icon-edit" @click="edit" />
            <HintButton title="查看详情" type="info" size="mini" icon="el-icon-info" @click="showSkuInfo(row)" />
            <HintButton title="删除" type="danger" size="mini" icon="el-icon-delete" />
          </template>
        </el-table-column>
      </el-table>
      <!-- @size-change="handleSizeChange"
        @current-change="handleCurrentChange" -->
      <el-pagination
        style="text-align:center"
        :current-page="page"
        :page-sizes="[3,5,10]"
        :page-size="limit"
        layout="prev, pager, next, jumper,->,sizes,total"
        :total="total"
        @current-change="getSkuList"
        @size-change="handleSizeChange"
      />
      <el-drawer
        :visible.sync="drawer"
        size="50%"
        :show-close="false"
      >
        <el-row>
          <el-col :span="5">名称</el-col>
          <el-col :span="16">{{ skuInfo.skuName }}</el-col>
        </el-row>
        <el-row>
          <el-col :span="5">描述</el-col>
          <el-col :span="16">{{ skuInfo.skuDesc }}</el-col>
        </el-row>
        <el-row>
          <el-col :span="5">价格</el-col>
          <el-col :span="16">{{ skuInfo.price }}</el-col>
        </el-row>
        <el-row>
          <el-col :span="5">平台属性</el-col>
          <el-col :span="16">
            <el-tag v-for="sku in skuInfo.skuAttrValueList" :key="sku.id" style="margin-right:5px" type="success">{{ sku.attrId }}--{{ sku.valueId }}</el-tag>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="5">商品图片</el-col>
          <el-col :span="16">
            <el-carousel height="400px">
              <el-carousel-item v-for="item in skuInfo.skuImageList" :key="item.id">
                <img style="width:400px;height:400px" :src="item.imgUrl" alt="">
              </el-carousel-item>
            </el-carousel>
          </el-col>
        </el-row>
      </el-drawer>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Sku',
  data() {
    return {
      page: 1,
      limit: 3,
      skuList: [],
      total: 0,
      skuInfo: [],
      drawer: false
    }
  },
  mounted() {
    this.getSkuList()
  },
  methods: {
    async getSkuList(pages = 1) {
      this.page = pages
      const result = await this.$API.sku.reqSkuList(this.page, this.limit)
      if (result.code === 200) {
        this.skuList = result.data.records
        this.total = result.data.total
      }
    },
    handleSizeChange(pagS) {
      this.limit = pagS
      this.getSkuList()
    },
    async onSale(row) {
      const result = await this.$API.sku.reqSale(row.id)
      if (result.code === 200) {
        // eslint-disable-next-line require-atomic-updates
        row.isSale = 1
        this.$message({ type: 'success', message: '上架成功' })
      }
    },
    async cancelSale(row) {
      const result = await this.$API.sku.reqSale(row.id)
      if (result.code === 200) {
        this.$message({ type: 'success', message: '下架成功' })
        // eslint-disable-next-line require-atomic-updates
        row.isSale = 0
      }
    },
    edit() {
      this.$message('正在开发中!')
    },
    async showSkuInfo(row) {
      const result = await this.$API.sku.reqSkuById(row.id)
      if (result.code === 200) {
        this.skuInfo = result.data
      }
      this.drawer = true
    }
  }
}
</script>

<style>
  .el-carousel{
    width: 400px;
  }
  .el-carousel__item h3 {
    color: #475669;
    font-size: 14px;
    opacity: 0.75;
    line-height: 150px;
    margin: 0;
  }

  .el-carousel__item:nth-child(2n) {
     background-color: #99a9bf;
  }

  .el-carousel__item:nth-child(2n+1) {
     background-color: #d3dce6;
  }
  .el-carousel__button{
    width: 8px;
    height: 8px;
    background-color: red;
    border-radius: 50%;
  }
</style>

<style scoped>
  .el-row .el-col-5{
    font-size: 18px;
    text-align: right;
    font-weight: bold;
  }
  .el-col{
    margin: 10px 10px;
  }
</style>
