<template>
  <div>
    <p style="color: red;font-size: 12px;">name字段（校验必填，校验3-10个字符</p>
    <p style="color: red;font-size: 12px;">可以给 data 设置特殊：_checked=true 默认选中；_disabled=true 默认禁止勾选，如果自定义了 selectable 方法，则根据该方法结果决定是否允许勾选</p>

    <p>
      <el-button type="success" size="mini" @click="insertEvent">新增一行</el-button>
      <el-button type="danger" size="mini" @click="$refs.editable.removeSelecteds()">删除选中</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.clear()">清空数据</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.clearSelection()">清空用户的选择</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.toggleAllSelection()">选中所有</el-button>
      <el-button type="warning" size="mini" @click="$refs.editable.toggleRowSelection(list[1], true)">设置第二行为选中</el-button>
    </p>

    <el-editable
      ref="editable"
      class="manual-table3"
      size="mini"
      border
      :data.sync="list"
      :edit-rules="validRules"
      :edit-config="{trigger: 'manual', mode: 'row', useDefaultValidTip: true, autoScrollIntoView: true, clearActiveMethod}"
      @clear-active="clearActiveEvent"
      style="width: 100%">
      <el-editable-column type="selection" width="55"></el-editable-column>
      <el-editable-column prop="sex" label="性别" :edit-render="{name: 'ElSelect', options: sexList, optionProps: { value: 'value', label: 'label' }}"></el-editable-column>
      <el-editable-column prop="age" label="年龄" :edit-render="{name: 'ElInputNumber', attrs: {min: 1, max: 200}}"></el-editable-column>
      <el-editable-column prop="name" label="名字" show-overflow-tooltip :edit-render="{name: 'ElInput'}"></el-editable-column>
      <el-editable-column prop="region" label="地区" :edit-render="{name: 'ElCascader', attrs: {options: regionList}}"></el-editable-column>
      <el-editable-column prop="birthdate" label="日期" :edit-render="{name: 'ElDatePicker', attrs: {type: 'date', format: 'yyyy-MM-dd'}}"></el-editable-column>
      <el-editable-column prop="date1" label="选择日期" :edit-render="{name: 'ElDatePicker', attrs: {type: 'datetime', format: 'yyyy-MM-dd hh:mm:ss'}}"></el-editable-column>
      <el-editable-column prop="flag" label="是否启用" :edit-render="{name: 'ElSwitch', type: 'visible'}"></el-editable-column>
      <el-editable-column prop="remark" label="备注" :edit-render="{name: 'ElInput'}"></el-editable-column>
      <el-editable-column label="操作" width="220">
        <template slot-scope="scope">
          <template v-if="$refs.editable.hasActiveRow(scope.row)">
            <el-button size="mini" type="success" @click="saveRowEvent(scope.row)">保存</el-button>
            <el-button size="mini" type="warning" @click="cancelRowEvent(scope.row)">取消</el-button>
            <el-button size="mini" type="warning" @click="$refs.editable.revert(scope.row)">还原</el-button>
          </template>
          <template v-else>
            <el-button size="mini" type="primary" @click="openActiveRowEvent(scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" @click="removeEvent(scope.row)">删除</el-button>
            <el-button size="mini" type="warning" @click="revertEvent(scope.row)">还原</el-button>
          </template>
        </template>
      </el-editable-column>
    </el-editable>
  </div>
</template>

<script>
import XEUtils from 'xe-utils'
import { Message, MessageBox } from 'element-ui'
import listData from '@/common/json/editable/list.json'
import regionData from '@/common/json/editable/region.json'

export default {
  data () {
    return {
      sexList: [
        {
          label: '男',
          spell: 'nan',
          value: '1',
          val: 'x'
        },
        {
          label: '女',
          spell: 'nv',
          value: '0',
          val: 'o'
        }
      ],
      regionList: regionData,
      list: [],
      isClearActiveFlag: true,
      validRules: {
        name: [
          { required: true, message: '请输入名称', trigger: 'change' },
          { min: 3, max: 10, message: '名称长度 3-10 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    let list = XEUtils.clone(listData, true).map((item, index) => {
      item._checked = index < 2
      item._disabled = index === 3
      return item
    })
    this.list = list
  },
  methods: {
    insertEvent () {
      let row = this.$refs.editable.insert({ age: 26 })
      this.$nextTick(() => this.$refs.editable.setActiveRow(row))
    },
    isRowOperate (row) {
      let activeInfo = this.$refs.editable.getActiveRow()
      return activeInfo ? activeInfo.row === row : true
    },
    clearActiveMethod ({ row }) {
      return this.isClearActiveFlag
    },
    openActiveRowEvent (row) {
      let activeInfo = this.$refs.editable.getActiveRow()
      // 如果当前行正在编辑中，禁止编辑其他行
      if (activeInfo) {
        if (activeInfo.row === row || !this.$refs.editable.checkValid().error) {
          if (activeInfo.isUpdate) {
            this.isClearActiveFlag = false
            MessageBox.confirm('检测到未保存的内容，是否在离开前保存修改?', '温馨提示', {
              closeOnClickModal: false,
              distinguishCancelAndClose: true,
              confirmButtonText: '保存',
              cancelButtonText: '放弃修改',
              type: 'warning'
            }).then(() => {
              this.$refs.editable.setActiveRow(row)
              this.updateRowEvent(activeInfo.row)
            }).catch(action => {
              if (action === 'cancel') {
                this.$refs.editable.revert(activeInfo.row)
                this.$refs.editable.setActiveRow(row)
                Message({ message: '放弃修改并离开当前行', type: 'warning' })
              } else {
                Message({ message: '停留在当前行编辑', type: 'info' })
              }
            }).then(() => {
              this.isClearActiveFlag = true
            })
          } else {
            this.$refs.editable.setActiveRow(row)
          }
        }
      } else {
        this.$refs.editable.setActiveRow(row)
      }
    },
    removeEvent (row) {
      if (this.isRowOperate(row)) {
        MessageBox.confirm('确定删除该数据?', '温馨提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$refs.editable.remove(row)
        }).catch(e => e)
      }
    },
    revertEvent (row) {
      if (this.isRowOperate(row)) {
        if (this.$refs.editable.hasRowChange(row)) {
          MessageBox.confirm('确定还原该行数据?', '温馨提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$refs.editable.revert(row)
            Message({ message: '数据还原成功！', type: 'success' })
          }).catch(e => e)
        } else {
          Message({ message: '数据未改动！', type: 'info' })
        }
      }
    },
    clearActiveEvent (row, event) {
      this.updateRowEvent(row)
    },
    updateRowEvent (row) {
      this.$refs.editable.reloadRow(row)
      Message({ message: '保存成功', type: 'success' })
    },
    saveRowEvent (row) {
      this.$refs.editable.validateRow(row, valid => {
        if (valid) {
          this.$refs.editable.clearActive()
          this.$refs.editable.reloadRow(row)
          Message({ message: '保存成功', type: 'success' })
        } else {
          console.log('error row submit!!')
        }
      })
    },
    cancelRowEvent (row) {
      let activeInfo = this.$refs.editable.getActiveRow()
      if (activeInfo && activeInfo.isUpdate) {
        MessageBox.confirm('检测到未保存的内容，确定放弃修改?', '温馨提示', {
          closeOnClickModal: false,
          confirmButtonText: '放弃更改',
          cancelButtonText: '返回',
          type: 'warning'
        }).then(action => {
          if (action === 'confirm') {
            this.$refs.editable.clearActive()
            this.$refs.editable.revert(row)
          } else {
            this.$refs.editable.setActiveRow(row, false)
          }
        }).catch(e => e)
      } else {
        this.$refs.editable.clearActive()
      }
    },
    validEvent () {
      this.$refs.editable.validate().then(valid => {
        Message({ message: '校验通过', type: 'success' })
      }).catch(valid => {
        console.log('error submit!!')
      })
    },
    submitEvent () {
      this.$refs.editable.validate().then(valid => {
        alert('提交通过')
      }).catch(valid => {
        console.log('error submit!!')
      })
    }
  }
}
</script>
