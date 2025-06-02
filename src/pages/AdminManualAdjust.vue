<template>
  <div class="admin-adjust">
    <el-card>
      <div class="search-bar">
        <el-input v-model="filters.courseTitle" placeholder="课程名称" style="width: 200px; margin-right: 10px" />
        <el-input-number v-model="filters.year" :min="2000" placeholder="学年" style="margin-right: 10px" />
        <el-select v-model="filters.semester" placeholder="学期" style="width: 120px; margin-right: 10px">
          <el-option label="Spring" value="Spring" />
          <el-option label="Summer" value="Summer" />
          <el-option label="Fall" value="Fall" />
          <el-option label="Winter" value="Winter" />
        </el-select>
        <el-button type="primary" @click="fetchData">搜索</el-button>
      </div>

      <el-table :data="tableData" style="margin-top: 20px" border>
        <el-table-column prop="secId" label="sec ID" width="80" />
        <el-table-column prop="courseTitle" label="课程名称" />
        <el-table-column prop="teacherId" label="教师ID" />
        <el-table-column prop="year" label="学年" width="80" />
        <el-table-column prop="semester" label="学期" width="100" />
        <el-table-column prop="classroomId" label="教室ID">
          <template #default="scope">
            <el-input v-model="scope.row.classroomId" size="small" />
          </template>
        </el-table-column>
        <el-table-column prop="timeSlotIds" label="时间段列表">
          <template #default="scope">
            <el-input v-model="scope.row.timeSlotIdsString" size="small" />
          </template>
        </el-table-column>
        <el-table-column label="操作" width="100">
          <template #default="scope">
            <el-button type="success" size="small" @click="submitUpdate(scope.row)">保存</el-button>
          </template>
        </el-table-column>
      </el-table>

      <div class="pagination">
        <el-pagination
          background
          layout="prev, pager, next"
          :total="total"
          :page-size="10"
          @current-change="handlePageChange"
        />
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import { fetchAdminSections, modifySection } from '@/api/schedule'

const filters = reactive({
  courseTitle: '',
  semester: '',
  year: new Date().getFullYear(),
})

const tableData = ref<any[]>([])
const page = ref(1)
const total = ref(0)

const fetchData = async () => {
  try {
    const res = await fetchAdminSections({
      ...filters,
      page: page.value,
      pageSize: 10,
    })
    tableData.value = res.data.records.map((item: any) => ({
      ...item,
      timeSlotIdsString: item.timeSlotIds.join(','),
    }))
    total.value = res.data.total
  } catch (err) {
    ElMessage.error('加载数据失败')
  }
}

const handlePageChange = (p: number) => {
  page.value = p
  fetchData()
}

const submitUpdate = async (row: any) => {
  const parsedSlots = row.timeSlotIdsString
    .split(',')
    .map((s: string) => parseInt(s.trim()))
    .filter((n: number) => !isNaN(n))

  try {
    await modifySection({
      sectionId: row.secId,
      classroomId: parseInt(row.classroomId),
      timeSlotIds: parsedSlots,
    })
    ElMessage.success('保存成功')
  } catch (err) {
    ElMessage.error('保存失败')
  }
}

onMounted(fetchData)
</script>

<style scoped>
.admin-adjust {
  padding: 20px;
}

.search-bar {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
}

.pagination {
  margin-top: 20px;
  text-align: right;
}
</style>