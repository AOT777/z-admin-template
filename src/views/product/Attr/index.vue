<template>
  <div>
    <el-card style="margin: 20px 0">
      <Category :show="!showTable" @getCategoryId="getCategoryId" />
    </el-card>
    <el-card>
      <div v-show="showTable">
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addAttr"
        >添加属性</el-button>
        <el-table :data="attrList" style="width: 100%" border>
          <el-table-column
            type="index"
            label="序号"
            width="80px"
            align="center"
          />
          <el-table-column prop="attrName" label="属性名称" width="200px" />
          <el-table-column prop="prop" label="属性值列表" width="width">
            <template slot-scope="{ row }">
              <el-tag
                v-for="tag in row.attrValueList"
                :key="tag.id"
                style="margin-right: 10px"
                type="success"
              >{{ tag.valueName }}</el-tag>
            </template></el-table-column>
          <el-table-column prop="prop" label="操作" width="200px">
            <template slot-scope="{row,$index}">
              <el-button
                type="warning"
                size="mini"
                icon="el-icon-edit"
                @click="updateAttr(row,$index)"
              />
              <el-button type="danger" size="mini" icon="el-icon-delete" />
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div v-show="!showTable">
        <el-form :inline="true" label-width="80px" :model="attrinfo">
          <el-form-item label="属性名称">
            <el-input v-model="attrinfo.attrName" placeholder="请输入属性名" />
          </el-form-item>
        </el-form>
        <el-button
          style="margin-left: 10px"
          type="primary"
          icon="el-icon-plus"
          :disabled="!attrinfo.attrName"
          @click="addAttrValue"
        >添加属性值</el-button>
        <el-table style="width: 100%; margin: 20px 0 20px 10px" border :data="attrinfo.attrValueList">
          <el-table-column
            type="index"
            label="序号"
            width="80px"
            align="center"
          />
          <el-table-column prop="prop" label="属性值名称" width="width">
            <template slot-scope="{row,$index}">
              <el-input v-if="row.flag" :ref="$index" v-model="row.valueName" placeholder="请输入属性值名称" size="mini" @blur="tolook(row)" @keyup.native.enter="tolook(row)" />
              <span v-else style="display:block" @click="toEdit(row,$index)">{{ row.valueName }}</span>
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{row,$index}">
              <el-popconfirm
                :title="`确认删除${row.valueName}吗?`"
                @onConfirm="deleteAttrvalue($index)"
              >
                <el-button slot="reference" type="danger" size="mini" icon="el-icon-delete" />
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-button style="margin-left: 10px" type="primary" :disabled="attrinfo.attrValueList.length<1" @click="addOrUpdateAttr">保存</el-button>
        <el-button @click="showTable=true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Attr',
  data() {
    return {
      category1Id: '',
      category2Id: '',
      category3Id: '',
      attrList: [],
      showTable: true,
      attrinfo: {
        attrName: '', // 属性名
        attrValueList: [
          // 属性名中属性值，因为属性值可以是多个，因此需要的是数组
        ],
        categoryId: 0, // category3Id
        categoryLevel: 3
      }
    }
  },
  methods: {
    getCategoryId({ categoryId, level }) {
      if (level === 1) {
        this.category1Id = categoryId
        this.category2Id = ''
        this.category3Id = ''
        this.attrList = []
      } else if (level === 2) {
        this.category2Id = categoryId
        this.category3Id = ''
        this.attrList = []
      } else {
        this.category3Id = categoryId
        this.getAttrList()
      }
    },
    async getAttrList() {
      const { category1Id, category2Id, category3Id } = this
      const result = await this.$API.atr.reqAttrList(
        category1Id,
        category2Id,
        category3Id
      )
      if (result.code === 200) {
        this.attrList = result.data
      }
    },
    addAttrValue() {
      this.attrinfo.attrValueList.push({
        attrId: this.attrinfo.id,
        valueName: '',
        flag: true
      })
      this.$nextTick(() => {
        this.$refs[this.attrinfo.attrValueList.length - 1].focus()
      })
    },
    addAttr() {
      this.showTable = false
      this.attrinfo = {
        attrName: '',
        attrValueList: [
        ],
        categoryId: this.category3Id,
        categoryLevel: 3
      }
    },
    updateAttr(row, index) {
      this.showTable = false
      // console.log(row)
      this.attrinfo = Object.assign(this.attrinfo, row)
      this.attrinfo.attrValueList.forEach(item => {
        this.$set(item, 'flag', true)
      })
    },
    tolook(row) {
      if (row.valueName.trim() === '') {
        this.$message('属性值不能为空')
        return
      }
      const isRepat = this.attrinfo.attrValueList.some(item => {
        // 不能和自己比
        if (row !== item) {
          return row.valueName === item.valueName
        }
      })
      if (isRepat) return
      row.flag = false
    },
    toEdit(row, index) {
      row.flag = true
      // input刚刚创建不能直接就获取,得等它渲染完毕后再去获取
      this.$nextTick(() => {
        this.$refs[index].focus()
      })
    },
    deleteAttrvalue(index) {
      this.attrinfo.attrValueList.splice(index, 1)
    },
    async addOrUpdateAttr() {
      this.attrinfo.attrValueList = this.attrinfo.attrValueList.filter(item => {
        if (item.valueName !== '') {
          delete item.flag
          return true
        }
      })
      console.log(this.attrinfo)
      try {
        await this.$API.atr.reqAddOrUpdateAttr(this.attrinfo)
        this.showTable = true
        this.$message({ type: 'success', message: '保存成功' })
        this.getAttrList()
      } catch (error) {
        this.$message('保存失败')
      }
    }
  }
}
</script>

<style scoped>
</style>
