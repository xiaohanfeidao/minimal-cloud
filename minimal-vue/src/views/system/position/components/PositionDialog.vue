<template>
  <div>
    <!-- 添加或修改岗位对话框 -->
    <el-dialog v-el-drag-dialog :title="dialog.title" :visible.sync="dialog.show" width="500px" @close="closeDialog">
      <el-form ref="form" :model="dialog.form" :rules="rules" label-width="80px">
        <el-form-item label="部门">
          <treeselect
            v-model="dialog.form.treeId"
            :options="leftTreeData"
            :normalizer="normalizerLeftTreeData"
            :show-count="true"
            placeholder="请选择部门"
            @input="changeLeftTreeNode"
          />
        </el-form-item>
        <el-form-item label="岗位编码" prop="number">
          <el-input v-model="dialog.form.number" placeholder="请输入岗位编码" />
        </el-form-item>
        <el-form-item label="岗位名称" prop="name">
          <el-input v-model="dialog.form.name" placeholder="请输入岗位名称" />
        </el-form-item>
        <el-form-item label="岗位状态" prop="enabled">
          <el-radio-group v-model="dialog.form.enabled">
            <el-radio
              v-for="dict in dialog.dictValues['enabled']"
              :key="dict.key"
              :label="dict.key"
            >{{dict.value}}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="dialog.form.remark" type="textarea" placeholder="请输入内容" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="closeDialog">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import Treeselect from '@riophae/vue-treeselect' // 树形结构选择器 Select
import elDragDialog from '@/directive/el-drag-dialog' // 可拖拽Dialog
import { insertPosition, updatePosition, leftTreeData } from '@/api/system/position' // 岗位api
export default {
  name: 'PositionDialog', // 组件名称
  components: {
    // 注册树形结构选择器
    Treeselect
  },
  directives: { elDragDialog }, // 注册可拖拽Dialog directive
  props: {
    dialog: Object // 从父组件接收的参数
  },
  data() {
    return {
      // 左树数据
      leftTreeData: {},
      // 表单校验
      rules: {
        name: [
          { required: true, message: '岗位名称不能为空', trigger: 'blur' }
        ],
        number: [
          { required: true, message: '岗位编码不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  // 初始化组件时执行
  created() {
    this.getLeftTreeData() // 获取左树数据
  },
  methods: {
    /**
     * 获得左树数据
     */
    getLeftTreeData() {
      leftTreeData().then(res => {
        this.leftTreeData = res
      })
    },
    /**
     * 转换左树数据
     * @param node
     */
    normalizerLeftTreeData(node) {
      return {
        id: node.id,
        label: node.name,
        children: node.children
      }
    },
    /**
     * 当左树节点更新时操作
     * @param value
     */
    changeLeftTreeNode(value) {

    },
    /**
     * 关闭弹出框
     */
    closeDialog() {
      this.dialog.form = this.deepClone(this.dialog.defaultForm)
      this.resetForm('form')
      this.dialog.show = false
    },
    /**
     * 表单提交
     */
    submitForm() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          const model = this.dialog.form
          if (model.id) {
            updatePosition(model.id, this.filterModel(model)).then(res => {
              this.$notify({
                title: '成功',
                message: '修改岗位成功',
                type: 'success'
              })
              this.closeDialog()
              this.$emit('refresh')
            })
          } else {
            insertPosition(this.filterModel(model)).then(res => {
              this.$notify({
                title: '成功',
                message: '新增岗位成功',
                type: 'success'
              })
              this.closeDialog()
              this.$emit('refresh')
            })
          }
        } else {
          console.log('error submit!!')
          return false
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
