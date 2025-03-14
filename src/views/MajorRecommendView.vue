<template>
  <div class="recommend-container">
    <div class="input-section">
      <el-form :model="userForm" label-width="120px">
        <el-form-item label="高考分数">
          <el-input-number 
            v-model="userForm.score" 
            :min="0" 
            :max="750"
            placeholder="请输入高考分数"
            style="width: 180px"
          />
        </el-form-item>
        <el-form-item label="兴趣方向">
          <el-select
            v-model="userForm.interests"
            multiple
            placeholder="请选择您的兴趣方向"
            style="width: 100%"
          >
            <el-option
              v-for="item in interestOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="特长科目">
          <el-select
            v-model="userForm.strengths"
            multiple
            placeholder="请选择您的特长科目"
            style="width: 100%"
          >
            <el-option
              v-for="item in strengthOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getRecommendations">获取推荐</el-button>
        </el-form-item>
      </el-form>
    </div>

    <div class="result-section" v-if="recommendList.length > 0">
      <h2>推荐专业列表</h2>
      <div class="major-list">
        <el-card v-for="major in recommendList" :key="major.id" class="major-card">
          <template #header>
            <div class="major-header">
              <h3>{{ major.name }}</h3>
              <el-tag :type="major.matchRate >= 80 ? 'success' : major.matchRate >= 60 ? 'warning' : 'info'">
                匹配度: {{ major.matchRate }}%
              </el-tag>
            </div>
          </template>
          <div class="major-content">
            <p><strong>专业类别：</strong>{{ major.category }}</p>
            <p><strong>就业方向：</strong>{{ major.careers }}</p>
            <p><strong>主要课程：</strong>{{ major.courses }}</p>
            <p><strong>推荐理由：</strong>{{ major.reason }}</p>
          </div>
        </el-card>
      </div>
      <el-pagination
        v-if="total > 10"
        class="pagination"
        background
        v-model:current-page="currentPage"
        :page-size="10"
        :total="total"
        layout="prev, pager, next"
        @current-change="handlePageChange"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { ElMessage } from 'element-plus'
import request from '@/utils/request'

const userForm = reactive({
  score: '',
  interests: [],
  strengths: []
})

const interestOptions = [
  { value: 'technology', label: '科技创新' },
  { value: 'art', label: '艺术设计' },
  { value: 'business', label: '商业经济' },
  { value: 'medicine', label: '医疗卫生' },
  { value: 'education', label: '教育服务' },
  { value: 'law', label: '法律政治' },
  { value: 'agriculture', label: '农业生产' },
  { value: 'environment', label: '环境保护' }
]

const strengthOptions = [
  { value: 'math', label: '数学' },
  { value: 'physics', label: '物理' },
  { value: 'chemistry', label: '化学' },
  { value: 'biology', label: '生物' },
  { value: 'chinese', label: '语文' },
  { value: 'english', label: '英语' },
  { value: 'politics', label: '政治' },
  { value: 'history', label: '历史' },
  { value: 'geography', label: '地理' }
]

const recommendList = ref([])
const currentPage = ref(1)
const total = ref(0)

const getRecommendations = async () => {
  if (!userForm.score) {
    ElMessage.warning('请输入高考分数')
    return
  }
  if (userForm.interests.length === 0) {
    ElMessage.warning('请至少选择一个兴趣方向')
    return
  }
  if (userForm.strengths.length === 0) {
    ElMessage.warning('请至少选择一个特长科目')
    return
  }

  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: '正在加载中...'
  })

  try {
    const response = await request.post('/majorRecommend/getRecommendations', {
      score: userForm.score,
      interests: userForm.interests,
      strengths: userForm.strengths,
      page: currentPage.value
    })

    if (response.code === 200) {
      recommendList.value = response.data.majors
      total.value = response.data.total
      if (recommendList.value.length === 0) {
        ElMessage.info('暂无匹配的专业推荐')
      }
    } else {
      ElMessage.error(response.message)
    }
  } catch (error) {
    ElMessage.error('获取推荐失败，请稍后重试')
  }

  loadingInstance.close()
}

const handlePageChange = (page) => {
  currentPage.value = page
  getRecommendations()
}
</script>

<style scoped>
.recommend-container {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.input-section {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.result-section {
  margin-top: 30px;
}

.major-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

.major-card {
  border-radius: 8px;
  transition: all 0.3s ease;
}

.major-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.major-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.major-header h3 {
  margin: 0;
  font-size: 18px;
  color: #303133;
}

.major-content {
  font-size: 14px;
  color: #606266;
}

.major-content p {
  margin: 8px 0;
  line-height: 1.6;
}

.pagination {
  margin-top: 30px;
  text-align: center;
}

h2 {
  color: #303133;
  margin-bottom: 20px;
  text-align: center;
}
</style> 