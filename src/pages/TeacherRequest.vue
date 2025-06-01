<template>
  <div class="teacher-request">
    <el-card shadow="hover" style="max-width: 600px; margin: 0 auto;">
      <h2>发起课程调整申请</h2>

      <el-form :model="form" :rules="rules" ref="formRef" label-width="120px">
        <el-form-item label="sec ID" prop="secId">
          <el-input-number v-model="form.secId" :min="1" placeholder="输入secID" />
        </el-form-item>

        <el-form-item label="教师ID" prop="teacher">
          <el-input v-model="form.teacher" placeholder="请输入教师姓名" />
        </el-form-item>

        <el-form-item label="申请原因" prop="reason">
          <el-input
            type="textarea"
            v-model="form.reason"
            placeholder="请输入课程调整申请原因(可包含理由、希望调整地点、时段等)"
            rows="4"
          />
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="handleSubmit">提交申请</el-button>
          <el-button @click="resetForm">重置</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { ElMessage, FormInstance } from 'element-plus';
import request from '@/utils/request'; // 请根据你的路径修改

// 表单数据模型
const form = ref({
  secId: null as number | null,
  reason: '',
  teacher: ''
});

// 表单验证规则
const rules = {
  secId: [{ required: true, message: '请输入课程章节 ID', trigger: 'blur' }],
  reason: [{ required: true, message: '请输入申请原因', trigger: 'blur' }],
  teacher: [{ required: true, message: '请输入教师姓名', trigger: 'blur' }]
};

const formRef = ref<FormInstance>();

// 提交表单
const handleSubmit = async () => {
  if (!formRef.value) return;
  await formRef.value.validate(async (valid) => {
    if (valid) {
      try {
        const res = await request.post('/application/add', form.value);
        ElMessage.success('申请提交成功');
        resetForm();
      } catch (err: any) {
        ElMessage.error(err?.response?.data?.message || '提交失败');
      }
    }
  });
};

const resetForm = () => {
  form.value = {
    secId: null,
    reason: '',
    teacher: ''
  };
};
</script>

<style scoped>
.teacher-request {
  padding: 32px;
}
</style>