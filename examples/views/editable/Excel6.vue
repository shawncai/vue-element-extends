<template>
  <div>
    <p style="color: red;font-size: 12px;">自定义设置动态列</p>
    <p style="color: red;font-size: 12px;">A字段（校验数值）B字段（校验汉字）C字段（校验字母）D字段（校验整数）E字段（校验小数）</p>
    <p style="color: red;font-size: 12px;">上下左右方向键切换列、Tab 键切换列、选中后可直接输入值覆盖旧值</p>

    <p>
      <el-button size="mini" @click="$refs.editable.insertAt(null, -1)">新增</el-button>
      <el-button size="mini" @click="$refs.editable.clearFilter()">清空筛选条件</el-button>
      <el-button size="mini" @click="$refs.editable.clearSort()">清空排序条件</el-button>
      <el-button size="mini" @click="getAllEvent">获取所有</el-button>
      <el-button size="mini" @click="getUpdateEvent">获取改动</el-button>
      <el-button size="mini" @click="getResultEvent">获取有值数据</el-button>
      <el-button size="mini" @click="exportCsvEvent">导出数据</el-button>
    </p>

    <el-editable
      ref="editable"
      class="excel-table6"
      border
      size="customSize"
      :edit-rules="validRules"
      :edit-config="{trigger: 'dblclick', showIcon: false, showStatus: false, isTabKey: true, isArrowKey: true}"
      style="width: 100%" >
      <el-editable-column type="index" align="center" width="50">
        <template slot="header">
          <i class="el-icon-setting" @click="dialogVisible = true"></i>
        </template>
      </el-editable-column>
      <template v-for="(item, index) in columnConfigs">
        <template v-if="item.customShow">
          <el-editable-column :key="index" v-bind="item" header-align="center" min-width="60" show-overflow-tooltip></el-editable-column>
        </template>
      </template>
    </el-editable>

    <el-dialog title="自定义列" :visible.sync="dialogVisible" width="540px" append-to-body @open="openCustomEvent">
      <el-transfer
        v-model="selectColumns"
        :data="columnConfigs"
        :titles="['隐藏列', '显示列']"
        :props="{key: 'prop', label: 'label'}"></el-transfer>
        <span slot="footer" class="dialog-footer">
          <el-button @click="resetCustomEvent">重 置</el-button>
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveCustomEvent">保 存</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
import XEUtils from 'xe-utils'
import { Message, MessageBox } from 'element-ui'

export default {
  data () {
    let columns = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z']
    const checkD = (rule, value, callback) => {
      if (!value || XEUtils.isInteger(Number(value))) {
        callback()
      } else {
        callback(new Error('请输入整数'))
      }
    }
    const checkE = (rule, value, callback) => {
      if (!value || XEUtils.isFloat(Number(value))) {
        callback()
      } else {
        callback(new Error('请输入小数'))
      }
    }
    return {
      dialogVisible: false,
      selectColumns: [],
      list: Array.from(new Array(15), (v, i) => {
        let rest = {}
        columns.forEach((name, index) => {
          rest[name.toLowerCase()] = `${name}${i}_${index}`
        })
        return rest
      }),
      columnConfigs: columns.map((name, index) => {
        let defaultShow = index < 20
        let column = {
          customDefault: defaultShow,
          customShow: defaultShow,
          prop: name.toLowerCase(),
          label: name,
          minWidth: '80',
          sortable: true,
          editRender: { name: 'ElInput' }
        }
        switch (name) {
          case 'A':
            column.filters = [{ text: '大于10', value: 10 }, { text: '大于50', value: 50 }, { text: '大于100', value: 100 }]
            column.filterMethod = (value, row, column) => Number(row[column.property] || 0) > value
            break
          case 'C':
            column.filters = [{ text: 'a开头', value: 'a' }, { text: 'b开头', value: 'b' }, { text: 'c开头', value: 'c' }]
            column.filterMethod = (value, row, column) => (row[column.property] || '').substring(0, 1) === value
            break
          case 'D':
            column.filters = [{ text: '大于0', value: 0 }, { text: '大于20', value: 20 }, { text: '大于200', value: 200 }]
            column.filterMethod = (value, row, column) => Number(row[column.property] || 0) > value
            break
          case 'E':
            column.filters = [{ text: '大于2.5', value: 2.5 }, { text: '大于7.8', value: 7.8 }, { text: '大于9.5', value: 9.5 }]
            column.filterMethod = (value, row, column) => Number(row[column.property] || 0) > value
            break
        }
        return column
      }),
      validRules: {
        a: [
          { type: 'number', message: '必须输入数字', trigger: 'change' }
        ],
        b: [
          { pattern: /^[\u4e00-\u9fa5]{1,5}$/, message: '校验1-5个汉字', trigger: 'change' }
        ],
        c: [
          { pattern: /^[a-zA-Z]{1,5}$/, message: '校验1-5个字母', trigger: 'blur' }
        ],
        d: [
          { validator: checkD, trigger: 'blur' }
        ],
        e: [
          { validator: checkE, trigger: 'change' }
        ]
      }
    }
  },
  created () {
    this.$nextTick(() => {
      this.$refs.editable.reload(this.list)
    })
  },
  methods: {
    exportCsvEvent () {
      this.$refs.editable.exportCsv({
        filename: 'Excel5数据.csv',
        columnFilterMethod: (column, columnIndex) => !['index'].includes(column.type)
      })
    },
    getAllEvent () {
      let rest = this.$refs.editable.getRecords()
      MessageBox({ message: JSON.stringify(rest), title: `获取所有数据(${rest.length}条)` }).catch(e => e)
    },
    getUpdateEvent () {
      let rest = this.$refs.editable.getUpdateRecords()
      MessageBox({ message: JSON.stringify(rest), title: `获取已修改数据(${rest.length}条)` }).catch(e => e)
    },
    getResultEvent () {
      let rest = this.$refs.editable.getRecords().filter(item => Object.keys(item).some(key => item[key]))
      MessageBox({ message: JSON.stringify(rest), title: `获取有值数据(${rest.length}条)` }).catch(e => e)
    },
    openCustomEvent () {
      this.selectColumns = this.columnConfigs.filter(column => column.customShow).map(column => column.prop)
    },
    resetCustomEvent () {
      this.selectColumns = this.columnConfigs.filter(column => column.customDefault).map(column => column.prop)
    },
    saveCustomEvent () {
      if (!this.selectColumns.length) {
        return Message({
          type: 'error',
          message: '请至少选择一列！'
        })
      }
      this.dialogVisible = false
      this.columnConfigs.forEach(column => {
        column.customShow = this.selectColumns.includes(column.prop)
      })
    }
  }
}
</script>

<style>
.excel-table6.el-table--customSize .editable-column {
  height: 30px;
}
.excel-table6 .el-table__body .el-table__row>td {
  cursor: cell;
}
.excel-table6 .el-table__header th,
.excel-table6 .el-table__body .el-table__row>td:first-child,
.excel-table6 .el-table__body .el-table__row:hover>td:first-child {
  background-color: #f5f5f5;
}
.excel-table6 .el-table__body .el-table__row>td:first-child {
  cursor: default;
}
.excel-table6 .el-table__body .el-table__row:hover>td {
  background-color: inherit;
}
.excel-table6 .el-table__body .el-table__row>td.editable-col_checked {
  border: 1px solid #217346;
}
.excel-table6 .el-table__body .el-table__row>td .cell {
  width: 100% !important;
  line-height: 30px;
  padding: 0 3px;
}
.excel-table6 .el-table__body .el-table__row>td.editable-col_checked .cell {
  padding: 0 2px;
  line-height: 28px;
}
.excel-table6 .el-table__body .el-table__row>td.editable-col_active .cell {
  padding: 0;
}
.excel-table6 .el-table__body .el-table__row>td .cell,
.excel-table6 .el-table__body .el-table__row>td .cell .el-input,
.excel-table6 .el-table__body .el-table__row>td .cell .el-input__inner {
  height: 100%;
}
.excel-table6 .el-table__body .el-table__row>td .cell .el-input__inner {
  border-radius: 0;
  padding: 0 2px;
  line-height: 30px;
  border-color: #217346;
}
.custom-wrapper {
  height: 200px;
  overflow: auto;
}
</style>
