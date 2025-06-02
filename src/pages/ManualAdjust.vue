<template>
  <div class="teacher-review">
    <el-card>
      <h2>教师申请审核</h2>
      <el-table :data="applications" style="width: 100%" v-loading="loading" border>
        <el-table-column prop="secId" label="sec ID" width="120" />
        <el-table-column prop="teacher" label="教师 ID" width="140" />
        <el-table-column prop="reason" label="申请理由" />
        <el-table-column label="处理建议">
          <template #default="scope">
            <el-input
              v-model="suggestionMap[scope.row.secId]"
              placeholder="请输入建议"
              size="small"
            />
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200">
          <template #default="scope">
            <el-button type="success" size="small" @click="handleProcess(scope.row.secId, true)">通过</el-button>
            <el-button type="danger" size="small" @click="handleProcess(scope.row.secId, false)">拒绝</el-button>
          </template>
        </el-table-column>
      </el-table>

      <div style="margin-top: 20px; text-align: right">
        <el-pagination
          background
          layout="prev, pager, next, jumper"
          :page-size="10"
          :total="total"
          @current-change="handlePageChange"
          :current-page="currentPage"
        />
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import request from '@/utils/request'

const applications = ref([])
const total = ref(0)
const currentPage = ref(1)
const loading = ref(false)

// 用于存储每条申请的处理建议
const suggestionMap = ref<Record<number, string>>({})

const fetchApplications = async (page = 1) => {
  loading.value = true
  try {
    const res = await request.get('/application/query', {
      params: {
        page,
        size: 10
      }
    })
    applications.value = res.data.items
    total.value = res.data.total
    currentPage.value = page

    // 初始化建议输入框
    suggestionMap.value = {}
    res.data.items.forEach((item: any) => {
      suggestionMap.value[item.secId] = item.suggestion || ''
    })
  } catch (err) {
    ElMessage.error('加载申请记录失败')
  } finally {
    loading.value = false
  }
}

const handlePageChange = (page: number) => {
  fetchApplications(page)
}

const handleProcess = async (secId: number, approved: boolean) => {
  const suggestion = suggestionMap.value[secId]
  if (!suggestion || suggestion.trim() === '') {
    ElMessage.warning('请填写处理建议后再提交审核')
    return
  }

  try {
    await request.post('/application/process', {
      secId,
      suggestion,
      finalDecision: approved
    })
    ElMessage.success(`已${approved ? '通过' : '拒绝'}申请`)
    fetchApplications(currentPage.value)
  } catch (err) {
    ElMessage.error('处理失败，请稍后重试')
  }
}

onMounted(() => {
  fetchApplications()
})
</script>

<style scoped>
.teacher-review {
  padding: 20px;
}

h2 {
  margin-bottom: 20px;
}
</style>