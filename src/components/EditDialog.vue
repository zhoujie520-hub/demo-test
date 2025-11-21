<script setup lang="ts">
import { ref, computed, useAttrs, watch } from 'vue'

// 定义数据类型
export interface EditFormData {
  id?: number
  categoryNameZh: string
  categoryNameEn: string
  sortOrder: number
  status: '启用' | '禁用'
}

type DialogMode = 'add' | 'edit'

interface Props {
  /** 弹窗显示状态 */
  visible: boolean
  /** 表单初始数据 */
  formData?: EditFormData | null
  /** 对话框模式：'add' 为新增，'edit' 为编辑 */
  mode: DialogMode
}

const props = withDefaults(defineProps<Props>(), {
  visible: false,
  formData: null,
  mode: 'add',
})

const emit = defineEmits<{
  /** 更新显示状态 */
  'update:visible': [value: boolean]
  /** 确认提交，返回表单数据 */
  'confirm': [data: EditFormData]
  /** 取消操作 */
  'cancel': []
}>()

// ========== 属性透传 ==========
// 获取未在 props 中声明的属性（$attrs）
// 这些属性会自动传递给 el-dialog，比如：
// - append-to-body: 是否插入至 body 元素上
// - lock-scroll: 是否在 Dialog 出现时将 body 滚动锁定
// - modal-class: 遮罩层自定义类名
// - custom-class: Dialog 的自定义类名
// - 等等其他 el-dialog 支持的属性
// 注意：显式设置的属性（如 :model-value、:title）会覆盖 attrs 中的同名属性
const attrs = useAttrs()

// ========== 组件内部状态 ==========
// 表单数据（组件内部管理，不直接修改 props）
const form = ref<EditFormData>({
  categoryNameZh: '',
  categoryNameEn: '',
  sortOrder: 1,
  status: '启用',
})

// 表单引用
const formRef = ref()

// ========== 配置常量 ==========
const maxLength = 50

// 表单验证规则
const rules = {
  categoryNameZh: [
    { required: true, message: '请输入主目录分类名称（中文）', trigger: 'blur' },
    { max: maxLength, message: `最多输入${maxLength}个字符`, trigger: 'blur' },
  ],
  categoryNameEn: [
    { required: true, message: '请输入主目录分类名称（英文）', trigger: 'blur' },
    { max: maxLength, message: `最多输入${maxLength}个字符`, trigger: 'blur' },
  ],
  sortOrder: [
    { required: true, message: '请输入购物网站排序', trigger: 'blur' },
    { type: 'number', min: 1, message: '排序值必须大于0', trigger: 'blur' },
  ],
  status: [
    { required: true, message: '请选择状态', trigger: 'change' },
  ],
} as const

// ========== 计算属性 ==========
// 字符计数
const zhCharCount = computed(() => form.value.categoryNameZh.length)
const enCharCount = computed(() => form.value.categoryNameEn.length)

// 判断是否为编辑模式
const isEditMode = computed(() => props.mode === 'edit')

// 对话框标题
const dialogTitle = computed(() => isEditMode.value ? '修改' : '新增')

// ========== 方法 ==========
/**
 * 初始化表单数据
 * 根据 formData 是否存在 id 来判断是新增还是编辑
 */
const initForm = () => {
  if (isEditMode.value && props.formData) {
    // 编辑模式：填充现有数据
    form.value = {
      id: props.formData.id,
      categoryNameZh: props.formData.categoryNameZh || '',
      categoryNameEn: props.formData.categoryNameEn || '',
      sortOrder: props.formData.sortOrder || 1,
      status: props.formData.status || '启用',
    }
  } else {
    // 新增模式：重置为默认值
    form.value = {
      categoryNameZh: '',
      categoryNameEn: '',
      sortOrder: 1,
      status: '启用',
    }
  }
}

/**
 * 重置表单
 * 使用 Element Plus 的 resetFields() 方法重置表单到初始值
 */
const resetForm = () => {
  formRef.value?.resetFields()
}

/**
 * 关闭弹窗
 */
const handleClose = () => {
  emit('update:visible', false)
  // 重置表单
  resetForm()
}

/**
 * 弹窗打开时（动画完成后）
 */
const handleOpened = () => {
  // 清除验证状态
  formRef.value?.clearValidate()
}

// 监听 visible 变化，在弹窗打开前就初始化数据
watch(
  () => props.visible,
  (newVal) => {
    if (newVal) {
      // 弹窗打开前立即初始化数据，避免用户看到数据填充过程
      initForm()
    }
  }
)

/**
 * 确认提交
 */
const handleConfirm = async () => {
  if (!formRef.value) return
  
  try {
    // 表单验证
    await formRef.value.validate()
    // 发送确认事件，传递表单数据（深拷贝，避免外部修改）
    emit('confirm', { ...form.value })
    // 关闭弹窗（会自动重置表单）
    handleClose()
  } catch (error) {
    // 验证失败，不关闭弹窗
    console.log('表单验证失败', error)
  }
}

/**
 * 取消操作
 */
const handleCancel = () => {
  emit('cancel')
  handleClose()
}

</script>

<template>
  <el-dialog
    :model-value="visible"
    :title="dialogTitle"
    width="600px"
    :close-on-click-modal="false"
    v-bind="attrs"
    @update:model-value="handleClose"
    @opened="handleOpened"
  >
    <el-form
      ref="formRef"
      :model="form"
      :rules="rules"
      label-width="180px"
      label-position="left"
    >
      <el-form-item label="主目录分类名称 (中文)*" prop="categoryNameZh">
        <div class="input-wrapper">
          <el-input
            v-model="form.categoryNameZh"
            :maxlength="maxLength"
            placeholder="请输入主目录分类名称（中文）"
            show-word-limit
          />
          <div class="char-count">{{ zhCharCount }}/{{ maxLength }}</div>
        </div>
      </el-form-item>

      <el-form-item label="主目录分类名称 (英文)*" prop="categoryNameEn">
        <div class="input-wrapper">
          <el-input
            v-model="form.categoryNameEn"
            :maxlength="maxLength"
            placeholder="请输入主目录分类名称（英文）"
            show-word-limit
          />
          <div class="char-count">{{ enCharCount }}/{{ maxLength }}</div>
        </div>
      </el-form-item>

      <el-form-item label="购物网站排序*" prop="sortOrder">
        <el-input-number
          v-model="form.sortOrder"
          :min="1"
          :max="9999"
          controls-position="right"
          style="width: 100%"
        />
      </el-form-item>

      <el-form-item label="状态*" prop="status">
        <el-radio-group v-model="form.status">
          <el-radio label="启用">启用</el-radio>
          <el-radio label="禁用">禁用</el-radio>
        </el-radio-group>
      </el-form-item>
    </el-form>

    <template #footer>
      <div class="dialog-footer">
        <el-button @click="handleCancel">取消</el-button>
        <el-button type="primary" @click="handleConfirm">确定</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<style scoped>

</style>
