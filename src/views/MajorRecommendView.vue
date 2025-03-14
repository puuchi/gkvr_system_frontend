<template>
  <div class="recommend-container">
    <!-- 步骤导航 -->
    <el-steps :active="activeStep" finish-status="success" class="steps-nav">
      <el-step title="心理测试" />
      <el-step title="测试报告" />
      <el-step title="专业推荐" />
    </el-steps>

    <!-- 心理测试问卷 -->
    <div v-if="activeStep === 0" class="test-section">
      <div class="test-header">
        <h2>职业性格测试</h2>
        <p>请根据您的真实想法选择最符合的选项</p>
      </div>
      
      <el-form :model="testForm" label-position="top" class="test-form">
        <div v-for="(question, index) in questions" :key="index" class="question-item">
          <el-form-item :label="'Q' + (index + 1) + '. ' + question.title">
            <el-radio-group v-model="testForm.answers[index]">
              <el-radio 
                v-for="option in question.options" 
                :key="option.value" 
                :label="option.value"
              >
                {{ option.label }}
              </el-radio>
            </el-radio-group>
          </el-form-item>
        </div>
      </el-form>

      <div class="form-actions">
        <el-button type="primary" @click="submitTest" :disabled="!isTestComplete">
          提交测试
        </el-button>
      </div>
    </div>

    <!-- 测试报告 -->
    <div v-if="activeStep === 1" class="report-section">
      <div class="report-card">
        <h2>心理测试报告</h2>
        <div class="report-content" v-if="testReport">
          <div class="personality-traits">
            <h3>性格特征</h3>
            <el-row :gutter="20">
              <el-col :span="8" v-for="(trait, index) in testReport.traits" :key="index">
                <div class="trait-item">
                  <h4>{{ trait.name }}</h4>
                  <el-progress 
                    :percentage="trait.score" 
                    :color="getProgressColor(trait.score)"
                  />
                </div>
              </el-col>
            </el-row>
          </div>

          <div class="career-interests">
            <h3>职业倾向</h3>
            <div class="interest-tags">
              <el-tag
                v-for="(interest, index) in testReport.interests"
                :key="index"
                :type="getTagType(index)"
                class="interest-tag"
              >
                {{ interest }}
              </el-tag>
            </div>
          </div>

          <div class="analysis">
            <h3>分析报告</h3>
            <p>{{ testReport.analysis }}</p>
          </div>
        </div>

        <div class="report-actions">
          <el-button @click="activeStep = 0">重新测试</el-button>
          <el-button type="primary" @click="getMajorRecommendations">
            查看专业推荐
          </el-button>
        </div>
      </div>
    </div>

    <!-- 专业推荐列表 -->
    <div v-if="activeStep === 2" class="major-section">
      <div class="major-header">
        <h2>推荐专业列表</h2>
        <p>基于您的心理测试结果，为您推荐以下专业</p>
      </div>

      <div class="major-list">
        <el-card v-for="major in recommendList" :key="major.id" class="major-card">
          <template #header>
            <div class="major-header">
              <h3>{{ major.name }}</h3>
              <el-tag :type="getMatchRateType(major.matchRate)">
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
import { ref, reactive, computed } from 'vue'
import {ElLoading, ElMessage} from 'element-plus'
import request from '@/utils/request'

// 当前步骤
const activeStep = ref(0)

// 问卷题目
const questions = [
  {
    title: '当面对新环境时，您更倾向于：',
    options: [
      { label: '主动与他人交流，建立关系', value: 'A' },
      { label: '先观察环境，慢慢适应', value: 'B' },
      { label: '保持中立，视情况而定', value: 'C' }
    ]
  },
  {
    title: '在团队合作中，您更喜欢：',
    options: [
      { label: '担任领导者，统筹规划', value: 'A' },
      { label: '专注于具体任务执行', value: 'B' },
      { label: '协调沟通，维护团队氛围', value: 'C' }
    ]
  },
  {
    title: '遇到问题时，您通常会：',
    options: [
      { label: '仔细分析原因，寻找最优解', value: 'A' },
      { label: '寻求他人建议和帮助', value: 'B' },
      { label: '凭直觉快速做出决定', value: 'C' }
    ]
  },
  {
    title: '您更喜欢的工作环境是：',
    options: [
      { label: '创新活力，充满挑战', value: 'A' },
      { label: '稳定有序，规范明确', value: 'B' },
      { label: '灵活自由，富有创意', value: 'C' }
    ]
  },
  {
    title: '在闲暇时间，您更愿意：',
    options: [
      { label: '阅读专业书籍，提升技能', value: 'A' },
      { label: '参加社交活动，扩展人脉', value: 'B' },
      { label: '从事艺术创作或体育运动', value: 'C' }
    ]
  }
]

// 测试表单
const testForm = reactive({
  answers: []
})

// 测试报告
const testReport = ref(null)

// 推荐专业列表
const recommendList = ref([])
const currentPage = ref(1)
const total = ref(0)

// 判断测试是否完成
const isTestComplete = computed(() => {
  return testForm.answers.length === questions.length && 
         !testForm.answers.includes(undefined)
})

// 在 script setup 部分添加模拟数据
// 模拟测试报告数据
const mockTestReport = {
  testId: 'test123',
  traits: [
    { name: '领导力', score: 85 },
    { name: '创造力', score: 75 },
    { name: '分析能力', score: 90 },
    { name: '沟通能力', score: 70 },
    { name: '执行力', score: 80 },
    { name: '抗压能力', score: 65 }
  ],
  interests: ['技术研发', '项目管理', '数据分析', '创新设计'],
  analysis: '根据测试结果显示，您具有较强的分析能力和领导力，同时在创新思维方面也表现出色。您适合在需要逻辑思维和决策能力的领域发展。在团队中，您倾向于担任领导角色，善于统筹规划。建议您选择能够发挥这些优势的专业方向。'
}

// 模拟专业推荐数据
const mockMajorList = [
  {
    id: 1,
    name: '计算机科学与技术',
    matchRate: 95,
    category: '工学',
    careers: '软件开发、系统架构师、技术管理等',
    courses: '数据结构、算法设计、计算机网络、操作系统等',
    reason: '您的逻辑分析能力强，对技术领域感兴趣，适合从事计算机相关工作。'
  },
  {
    id: 2,
    name: '数据科学与大数据技术',
    matchRate: 90,
    category: '理学',
    careers: '数据分析师、数据工程师、商业智能分析等',
    courses: '统计学、机器学习、数据挖掘、数据可视化等',
    reason: '您具备较强的分析能力和逻辑思维，适合在数据分析领域发展。'
  },
  {
    id: 3,
    name: '工商管理',
    matchRate: 85,
    category: '管理学',
    careers: '企业管理、创业、咨询顾问等',
    courses: '管理学原理、市场营销、财务管理、组织行为学等',
    reason: '您的领导力突出，具有良好的决策能力，适合从事管理工作。'
  },
  {
    id: 4,
    name: '人工智能',
    matchRate: 82,
    category: '工学',
    careers: 'AI算法工程师、机器学习工程师、智能系统开发等',
    courses: '机器学习、深度学习、计算机视觉、自然语言处理等',
    reason: '您对技术创新有浓厚兴趣，同时具备较强的学习能力。'
  },
  {
    id: 5,
    name: '金融工程',
    matchRate: 78,
    category: '经济学',
    careers: '金融分析师、量化交易员、风险管理师等',
    courses: '金融数学、计量经济学、投资学、风险管理等',
    reason: '您的数理分析能力强，对金融市场感兴趣。'
  }
]

// 修改提交测试方法
const submitTest = async () => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: '正在分析测试结果...'
  })

  try {
    // 模拟后端请求延迟
    await new Promise(resolve => setTimeout(resolve, 1000))
    
    // 使用模拟数据
    testReport.value = mockTestReport
    activeStep.value = 1
    
  } catch (error) {
    ElMessage.error('提交失败，请稍后重试')
  }

  loadingInstance.close()
}

// 修改获取专业推荐方法
const getMajorRecommendations = async () => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: '正在生成专业推荐...'
  })

  try {
    // 模拟后端请求延迟
    await new Promise(resolve => setTimeout(resolve, 1500))
    
    // 使用模拟数据
    recommendList.value = mockMajorList
    total.value = mockMajorList.length
    activeStep.value = 2
    
  } catch (error) {
    ElMessage.error('获取推荐失败，请稍后重试')
  }

  loadingInstance.close()
}

// 处理分页
const handlePageChange = (page) => {
  currentPage.value = page
  getMajorRecommendations()
}

// 获取进度条颜色
const getProgressColor = (score) => {
  if (score >= 80) return '#67C23A'
  if (score >= 60) return '#E6A23C'
  return '#F56C6C'
}

// 获取标签类型
const getTagType = (index) => {
  const types = ['', 'success', 'warning', 'danger']
  return types[index % types.length]
}

// 获取匹配度标签类型
const getMatchRateType = (rate) => {
  if (rate >= 80) return 'success'
  if (rate >= 60) return 'warning'
  return 'info'
}
</script>

<style scoped>
.recommend-container {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.steps-nav {
  margin-bottom: 40px;
}

.test-section {
  max-width: 800px;
  margin: 0 auto;
}

.test-header {
  text-align: center;
  margin-bottom: 30px;
}

.test-header h2 {
  color: #303133;
  margin-bottom: 10px;
}

.test-header p {
  color: #606266;
  font-size: 14px;
}

.question-item {
  margin-bottom: 30px;
  padding: 20px;
  background: #f5f7fa;
  border-radius: 8px;
}

.form-actions {
  text-align: center;
  margin-top: 30px;
}

.report-section {
  max-width: 900px;
  margin: 0 auto;
}

.report-card {
  background: #fff;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.report-card h2 {
  text-align: center;
  margin-bottom: 30px;
  color: #303133;
}

.personality-traits {
  margin-bottom: 30px;
}

.trait-item {
  margin-bottom: 20px;
}

.trait-item h4 {
  margin: 0 0 10px;
  color: #606266;
}

.career-interests {
  margin-bottom: 30px;
}

.interest-tags {
  margin-top: 15px;
}

.interest-tag {
  margin: 0 10px 10px 0;
  padding: 6px 12px;
}

.analysis {
  margin-bottom: 30px;
}

.analysis p {
  line-height: 1.6;
  color: #606266;
}

.report-actions {
  text-align: center;
  margin-top: 30px;
}

.major-section {
  margin-top: 20px;
}

.major-header {
  text-align: center;
  margin-bottom: 30px;
}

.major-header h2 {
  color: #303133;
  margin-bottom: 10px;
}

.major-header p {
  color: #606266;
  font-size: 14px;
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

.major-card :deep(.el-card__header) {
  padding: 15px 20px;
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
</style> 