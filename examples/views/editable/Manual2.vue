<template>
  <div>
    <p style="color: red;font-size: 12px;">name字段（校验必填，校验3-10个字符；flag（校验必填）</p>

    <p>
      <el-button type="success" size="mini" @click="insertEvent(0)">新增一行</el-button>
      <el-button type="success" size="mini" @click="insertEvent(-1)">在最后新增一行</el-button>
      <el-button type="danger" size="mini" @click="$refs.editable.removeSelecteds()">删除选中</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.revert()">放弃更改</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.clear()">清空数据</el-button>
      <el-button type="warning" size="mini" @click="validEvent">校验</el-button>
      <el-button type="warning" size="mini" @click="submitEvent">校验&保存</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.clearSelection()">清空用户的选择</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.toggleAllSelection()">选中所有</el-button>
    </p>

    <el-editable
      ref="editable"
      class="manual-table2"
      size="mini"
      border
      :data.sync="list"
      :edit-rules="validRules"
      :edit-config="{trigger: 'manual', mode: 'row'}"
      style="width: 100%">
      <el-editable-column type="selection" width="55"></el-editable-column>
      <el-editable-column prop="sex" label="性别" :edit-render="{name: 'ElSelect', options: sexList}"></el-editable-column>
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
            <el-button size="mini" type="primary" @click="$refs.editable.setActiveRow(scope.row)">编辑</el-button>
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
          value: '1'
        },
        {
          label: '女',
          value: '0'
        }
      ],
      regionList: regionData,
      list: XEUtils.clone(listData, true),
      validRules: {
        name: [
          { required: true, message: '请输入名称', trigger: 'change' },
          { min: 3, max: 10, message: '名称长度 3-10 个字符', trigger: 'blur' }
        ],
        flag: [
          { required: true, message: '必须选择启用才能保存', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    insertEvent (index) {
      let row = this.$refs.editable.insertAt({ name: '默认名字1' }, index)
      this.$nextTick(() => this.$refs.editable.setActiveRow(row))
    },
    removeEvent (row) {
      MessageBox.confirm('确定删除该数据?', '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$refs.editable.remove(row)
      }).catch(e => e)
    },
    revertEvent (row) {
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
    },
    saveRowEvent (row) {
      this.$refs.editable.validateRow(row, valid => {
        if (valid) {
          this.$refs.editable.clearActive()
          Message({ message: '保存成功', type: 'success' })
        } else {
          console.log('error row submit!!')
        }
      })
    },
    cancelRowEvent (row) {
      this.$refs.editable.validateRow(row, (valid, validErrs) => {
        if (valid) {
          this.$refs.editable.clearActive()
        } else {
          let message = <p>
            <p>请正确填写以下信息！</p>
            {
              Object.keys(validErrs).map(name => {
                let errors = validErrs[name]
                let msg = `${name}：${errors.map(e => e.message).join(';')}`
                return <p>{msg}</p>
              })
            }
          </p>
          Message({ message, dangerouslyUseHTMLString: true, type: 'error' })
        }
      })
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
