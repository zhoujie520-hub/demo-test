<script setup lang="ts">
import { ref, computed, reactive } from 'vue'
import { ElMessageBox, ElMessage } from 'element-plus'
import EditDialog, { type EditFormData } from './EditDialog.vue'

// 定义数据类型
interface TableItem {
  id: number
  categoryNameZh: string
  categoryNameEn: string
  sortOrder: number
  status: '启用' | '禁用'
}

type DialogMode = 'add' | 'edit'

// 模拟数据 - 更新数据结构
const tableData = ref<TableItem[]>([
  { id: 2356, categoryNameZh: '飞行器', categoryNameEn: 'combat flight control', sortOrder: 1, status: '启用' },
  { id: 2645, categoryNameZh: '飞行器', categoryNameEn: 'combat flight control', sortOrder: 2, status: '启用' },
  { id: 2632, categoryNameZh: '飞行器', categoryNameEn: 'combat flight control', sortOrder: 3, status: '禁用' },
  { id: 2630, categoryNameZh: '飞行器', categoryNameEn: 'combat flight control', sortOrder: 6, status: '启用' },
  { id: 2662, categoryNameZh: '飞行器', categoryNameEn: 'combat flight control', sortOrder: 5, status: '启用' },
  { id: 2699, categoryNameZh: '飞行器', categoryNameEn: 'combat flight control', sortOrder: 4, status: '启用' },
  { id: 2369, categoryNameZh: '飞行器', categoryNameEn: 'combat flight control', sortOrder: 9, status: '启用' },
  { id: 2586, categoryNameZh: '飞行器', categoryNameEn: 'combat flight control', sortOrder: 11, status: '启用' },
])

// 弹窗相关
const dialogVisible = ref(false)
const currentEditData = ref<EditFormData | null>(null)

// 分页相关
const pagination = reactive({
  currentPage: 1,
  pageSize: 2,
  jumpPage: 1,
  pageSizeOptions: ['10', '20', '50', '100'] as const,
})

const total = computed(() => tableData.value.length)

// 分页后的数据
const paginatedData = computed(() => {
  const start = (pagination.currentPage - 1) * pagination.pageSize
  const end = start + pagination.pageSize
  return tableData.value.slice(start, end)
})

// 弹窗模式
const dialogMode = ref<DialogMode>('add')

// 处理新增
// 新增模式：不传 id，表单为空
const handleAdd = () => {
  dialogMode.value = 'add'
  currentEditData.value = null // 没有数据，表示新增
  dialogVisible.value = true
}

// 处理修改
// 编辑模式：传入完整的行数据（包含 id）
const handleEdit = (row: TableItem) => {
  dialogMode.value = 'edit'
  currentEditData.value = { ...row }
  dialogVisible.value = true
}

// 处理删除
const handleDelete = async (row: TableItem) => {
  try {
    await ElMessageBox.confirm(
      `确定要删除"${row.categoryNameZh}"吗？`,
      '删除确认',
      {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }
    )
    
    // 执行删除
    const index = tableData.value.findIndex(item => item.id === row.id)
    if (index > -1) {
      tableData.value.splice(index, 1)
      ElMessage.success('删除成功')
      
      // 如果当前页没有数据了，跳转到上一页
      if (paginatedData.value.length === 0 && pagination.currentPage > 1) {
        pagination.currentPage--
        pagination.jumpPage = pagination.currentPage
      }
    }
  } catch {
    // 用户取消删除
  }
}

// 处理弹窗确认
const handleDialogConfirm = (data: EditFormData) => {
  // 使用映射对象处理不同模式的操作
  const modeHandlers: Record<DialogMode, (data: EditFormData) => void> = {
    edit: (data: EditFormData) => {
      const index = tableData.value.findIndex(item => item.id === data.id)
      if (index > -1) {
        tableData.value[index] = { ...data, id: data.id! }
        ElMessage.success('修改成功')
      } else {
        ElMessage.error('未找到要修改的数据')
      }
    },
    add: (data: EditFormData) => {
      const newId = Math.max(...tableData.value.map(item => item.id), 0) + 1
      tableData.value.push({
        ...data,
        id: newId,
      })
      ElMessage.success('新增成功')
      const totalPages = Math.ceil(total.value / pagination.pageSize)
      pagination.currentPage = totalPages
      pagination.jumpPage = totalPages
    },
  }
  
  // 根据当前模式执行对应操作
  modeHandlers[dialogMode.value](data)
}

// 分页变化
const handleSizeChange = (val: number) => {
  pagination.pageSize = val
  pagination.currentPage = 1
}

const handleCurrentChange = (val: number) => {
  pagination.currentPage = val
  pagination.jumpPage = val
}

// 跳转到指定页
const handleJump = () => {
  if (pagination.jumpPage >= 1 && pagination.jumpPage <= Math.ceil(total.value / pagination.pageSize)) {
    pagination.currentPage = pagination.jumpPage
  }
}
</script>

<template>
  <div class="data-table-container">

    <!--  -->
    <div>测试自动化部署</div>
    <!-- 操作栏 -->
    <div class="table-header">
      <el-button type="primary" @click="handleAdd">新增</el-button>
    </div>

    <!-- 表格 -->
    <el-table
      :data="paginatedData"
      style="width: 100%"
    >
      <el-table-column prop="id" label="ID" width="100" />
      <el-table-column label="主目录分类">
        <template #default="{ row }">
          {{ row.categoryNameZh }},{{ row.categoryNameEn }}
        </template>
      </el-table-column>
      <el-table-column prop="sortOrder" label="购物网站排序" width="150" />
      <el-table-column prop="status" label="状态" width="100">
        <template #default="{ row }">
          <span :class="{ 'status-enabled': row.status === '启用', 'status-disabled': row.status === '禁用' }">
            {{ row.status }}
          </span>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="200">
        <template #default="{ row }">
          <el-button type="primary" size="small" @click="handleEdit(row)">修改</el-button>
          <el-button type="danger" size="small" @click="handleDelete(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-container">
      <div class="pagination-left">
        <el-select v-model="pagination.pageSize" @change="handleSizeChange" style="width: 120px">
          <el-option
            v-for="option in pagination.pageSizeOptions"
            :key="option"
            :label="`${option}条/页`"
            :value="Number(option)"
          />
        </el-select>
      </div>
      
      <div class="pagination-center">
        <el-pagination
          v-model:current-page="pagination.currentPage"
          :page-size="pagination.pageSize"
          :total="total"
          layout="prev, pager, next"
          @current-change="handleCurrentChange"
        />
      </div>

      <div class="pagination-right">
        <span>前往</span>
        <el-input
          v-model.number="pagination.jumpPage"
          type="number"
          :min="1"
          :max="Math.ceil(total / pagination.pageSize)"
          style="width: 60px; margin: 0 8px"
          @keyup.enter="handleJump"
        />
        <span>页</span>
      </div>
    </div>

    <!-- 编辑/新增弹窗 -->
    <EditDialog
      v-model:visible="dialogVisible"
      :form-data="currentEditData"
      :mode="dialogMode"
      @confirm="handleDialogConfirm"
    />
  </div>
</template>

<style scoped>

</style>
