<template>
  <div>
    <el-card style="margin:20px 0">
      <Category :show="scene!=0" @getCategoryId="getCategoryId" />
    </el-card>
    <el-card>
      <!-- first part -->
      <div v-show="scene==0">
        <el-button :disabled="!category3Id" type="primary" icon="el-icon-plus" @click="addSpu">添加SPU</el-button>
        <el-table
          style="width: 100%"
          :data="spuList"
          border
        >
          <el-table-column
            type="index"
            label="序号"
            width="80px"
            align="center"
          />
          <el-table-column
            prop="spuName"
            label="spu名称"
            width="250px"
          />
          <el-table-column
            prop="description"
            label="spu描述"
            width="width"
          />
          <el-table-column
            prop="prop"
            label="操作"
            width="width"
          >
            <template slot-scope="{row}">
              <HintButton title="添加sku" type="success" size="mini" icon="el-icon-plus" @click="addSku(row)" />
              <HintButton title="修改spu" type="warning" size="mini" icon="el-icon-edit" @click="updateSpu(row)" />
              <HintButton title="查看当前spu全部sku列表" type="info" size="mini" icon="el-icon-info" @click="handleShowSku(row)" />
              <el-popconfirm
                :title="`确定删除${row.spuName}吗？`"
                @onConfirm="deleteSpu(row)"
              >
                <HintButton slot="reference" title="删除spu" type="danger" size="mini" icon="el-icon-delete" />
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <!-- @size-change="handleSizeChange"
          @current-change="handleCurrentChange" -->
        <el-pagination
          style="margin-top:5px;text-align:center"
          :current-page="page"
          :page-count="5"
          :page-sizes="[3,5,10]"
          :page-size="limit"
          layout="prev, pager, next, jumper,->,sizes,total"
          :total="total"
          @current-change="getSpuList"
          @size-change="handleSizeChange"
        />
      </div>
      <!-- second part添加或修改spu -->
      <SpuForm v-show="scene==1" ref="spu" @changeScene="changeScene" />
      <!-- third part添加sku -->
      <SkuForm v-show="scene==2" ref="sku" @changeScene2="changeScene2" />
    </el-card>
    <!-- 查看当前spu全部sku列表 -->
    <el-dialog :title="`${spu.spuName}的sku列表`" :visible.sync="dialogTableVisible" :before-close="closeDialog">
      <el-table
        v-loading="loading"
        :data="skuList"
        style="width: 100%"
        border
      >
        <el-table-column
          prop="skuName"
          label="名称"
          width="width"
        />
        <el-table-column
          prop="price"
          label="价格"
          width="width"
        />
        <el-table-column
          prop="weight"
          label="重量"
          width="width"
        />
        <el-table-column
          prop="prop"
          label="图片"
          width="width"
        >
          <template slot-scope="{row}">
            <img :src="row.skuDefaultImg" alt="" style="width:80px;height:80px">
          </template>
        </el-table-column>

      </el-table>
    </el-dialog>
  </div>
</template>

<script>
import SpuForm from './SpuForm'
import SkuForm from './SkuForm'
export default {
  name: 'Spu',
  components: { SpuForm, SkuForm },
  data() {
    return {
      category1Id: '',
      category2Id: '',
      category3Id: '',
      showTable: false,
      page: 1,
      limit: 3,
      total: 0,
      spuList: [],
      scene: 0,
      dialogTableVisible: false,
      spu: {},
      skuList: [],
      loading: true
    }
  },
  methods: {
    getCategoryId({ categoryId, level }) {
      if (level === 1) {
        this.category1Id = categoryId
        this.category2Id = ''
        this.category3Id = ''
        this.spuList = []
      } else if (level === 2) {
        this.category2Id = categoryId
        this.category3Id = ''
        this.spuList = []
      } else {
        this.category3Id = categoryId
        this.getSpuList()
      }
    },
    async getSpuList(pages = 1) {
      this.page = pages
      const { page, limit, category3Id } = this
      const result = await this.$API.spu.reqSpuList(page, limit, category3Id)
      if (result.code === 200) {
        this.spuList = result.data.records
        this.total = result.data.total
      }
    },
    handleSizeChange(pags) {
      this.limit = pags
      this.getSpuList()
    },
    addSpu() {
      this.scene = 1
      this.$refs.spu.addSpuData(this.category3Id)
    },
    updateSpu(row) {
      this.scene = 1
      this.$refs.spu.initSpuData(row)
    },
    changeScene({ scene, flag }) {
      this.scene = scene
      // eslint-disable-next-line eqeqeq
      if (flag == '修改') {
        this.getSpuList(this.page)
      } else {
        this.getSpuList()
      }
    },
    async deleteSpu(row) {
      const result = await this.$API.spu.reqDeleteSpu(row.id)
      if (result.code === 200) {
        this.$message({ type: 'success', message: '保存成功' })
        this.getSpuList(this.spuList.length > 1 ? this.page : this.page - 1)
      }
    },
    addSku(row) {
      this.scene = 2
      const { category1Id, category2Id, category3Id } = this
      this.$refs.sku.getSkuData(row, category1Id, category2Id, category3Id)
    },
    changeScene2(scene) {
      this.scene = scene
    },
    async handleShowSku(row) {
      this.spu = row
      this.dialogTableVisible = true
      const result = await this.$API.spu.reqSkuList(row.id)
      if (result.code === 200) {
        this.skuList = result.data
        this.loading = false
      }
    },
    closeDialog(done) {
      this.loading = true
      this.skuList = []
      done()
    }
  }
}
</script>

<style scoped>

</style>
