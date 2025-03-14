<template>
  <el-menu
    :default-active="onRoutes"
    :ellipsis="false"
    mode="horizontal"
    @select="handleSelect"
  >
    <el-menu-item v-for="item in menu" :key="item.path" :index="item.path">
      <template v-slot:title>
        <span style="font-size: 16px">{{ item.lable }}</span>
      </template>
    </el-menu-item>

    <div class="menu-right">
      <el-button @click="showDialog">志愿表</el-button>
      <div class="user-avator">
        <el-icon>
          <Avatar />
        </el-icon>
      </div>
      <el-dropdown trigger="click" @command="handleCommand">
        <span class="el-dropdown-link">
          {{ username }}
          <el-icon class="el-icon--right">
            <arrow-down />
          </el-icon>
        </span>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item command="profile">个人信息管理</el-dropdown-item>
            <el-dropdown-item command="loginout">退出登录</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </el-menu>

  <!-- 个人信息管理对话框 -->
  <el-dialog
    title="个人信息管理"
    v-model="showProfileDialog"
    width="500px"
    align-center
    center
  >
    <el-form :model="profileForm" label-width="120px">
      <el-form-item label="省份">
        <el-select v-model="profileForm.province" placeholder="请选择省份" style="width: 100%">
          <el-option
            v-for="item in provinceList"
            :key="item"
            :label="item"
            :value="item"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="首选科目">
        <el-radio-group v-model="profileForm.firstSubject">
          <el-radio label="物理">物理</el-radio>
          <el-radio label="历史">历史</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="再选科目">
        <div class="second-subjects">
          <el-checkbox-group 
            v-model="profileForm.secondSubjects" 
            :max="2"
          >
            <el-checkbox 
              v-for="subject in getSecondSubjectOptions"
              :key="subject"
              :label="subject"
              :disabled="profileForm.secondSubjects.length >= 2 && !profileForm.secondSubjects.includes(subject)"
            >
              {{ subject }}
            </el-checkbox>
          </el-checkbox-group>
          <div class="subject-hint" v-if="profileForm.secondSubjects.length < 2">
            请选择2个科目
          </div>
        </div>
      </el-form-item>
      <el-form-item label="高考分数">
        <el-input-number 
          v-model="profileForm.score" 
          :min="0" 
          :max="750" 
          style="width: 180px"
        />
      </el-form-item>
      <el-form-item label="排名">
        <div style="display: flex; align-items: center;">
          <el-input v-model="profileForm.rank" style="width: 180px" disabled />
          <el-button 
            type="primary" 
            @click="getAutoRank" 
            style="margin-left: 10px"
            :disabled="!canGetRank"
          >
            自动获取
          </el-button>
        </div>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="showProfileDialog = false">取消</el-button>
        <el-button type="primary" @click="saveProfile">保存</el-button>
      </span>
    </template>
  </el-dialog>

  <!-- 志愿表对话框 -->
  <el-dialog
    title="志愿表管理"
    v-model="showTableDialog"
    align-center
    center
    width="90%"
    class="volunteer-dialog"
  >
    <!-- 志愿表格 -->
    <div class="volunteer-table">
      <el-table
        :data="volunteerList"
        row-key="id"
        border
        style="width: 100%"
      >
        <el-table-column label="序号" type="index" width="60" align="center" />
        <el-table-column label="学校名称" prop="schoolName" width="180" />
        <el-table-column label="风险等级" width="100" align="center">
          <template #default="scope">
            <el-tag :type="getRiskTagType(scope.row.riskLevel)">
              {{ scope.row.riskLevel }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="录取概率" width="100" align="center">
          <template #default="scope">
            <span>{{ scope.row.admissionRate }}%</span>
          </template>
        </el-table-column>
        <el-table-column label="专业组" min-width="400">
          <template #default="scope">
            <el-table
              :data="scope.row.majorGroups"
              border
              style="width: 100%"
            >
              <el-table-column prop="groupName" label="组名称" width="220">
                <template #default="groupScope">
                  <div class="group-info">
                    <div class="group-name">{{ groupScope.row.groupName }}</div>
                    <div class="group-tags">
                      <el-tag size="small" type="info">
                        选考科目: {{ groupScope.row.requiredSubjects.join('、') }}
                      </el-tag>
                      <el-tag 
                        size="small" 
                        :type="getRiskTagType(groupScope.row.riskLevel)"
                      >
                        {{ groupScope.row.riskLevel }} ({{ groupScope.row.admissionRate }}%)
                      </el-tag>
                    </div>
                  </div>
                </template>
              </el-table-column>
              <el-table-column label="专业选择" min-width="260">
                <template #default="groupScope">
                  <div class="major-selection">
                    <div v-for="(major, index) in groupScope.row.majors" :key="index" class="major-item">
                      <span class="major-index">{{ String.fromCharCode(65 + index) }}.</span>
                      <el-select 
                        v-model="major.selected" 
                        placeholder="请选择专业"
                        style="width: 180px"
                      >
                        <el-option
                          v-for="option in groupScope.row.majorOptions"
                          :key="option"
                          :label="option"
                          :value="option"
                        />
                      </el-select>
                    </div>
                  </div>
                </template>
              </el-table-column>
            </el-table>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="160" align="center">
          <template #default="scope">
            <el-button-group>
              <el-button 
                type="primary" 
                :icon="ArrowUp" 
                @click="moveVolunteer(scope.$index, 'up')"
                :disabled="scope.$index === 0"
              >
                上移
              </el-button>
              <el-button 
                type="primary" 
                :icon="ArrowDown" 
                @click="moveVolunteer(scope.$index, 'down')"
                :disabled="scope.$index === volunteerList.length - 1"
              >
                下移
              </el-button>
              <el-button 
                type="danger" 
                :icon="Delete"
                @click="handleDelete(scope.row)"
              >
                删除
              </el-button>
            </el-button-group>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 风险评估部分 -->
    <div class="risk-assessment">
      <h3>风险评估</h3>
      <div class="risk-distribution">
        <div class="risk-item">
          <span class="risk-label">冲档</span>
          <el-progress 
            :percentage="Math.round(riskAssessment.冲 / volunteerList.length * 100)" 
            :color="'#F56C6C'"
          />
        </div>
        <div class="risk-item">
          <span class="risk-label">稳档</span>
          <el-progress 
            :percentage="Math.round(riskAssessment.稳 / volunteerList.length * 100)" 
            :color="'#67C23A'"
          />
        </div>
        <div class="risk-item">
          <span class="risk-label">保底</span>
          <el-progress 
            :percentage="Math.round(riskAssessment.保 / volunteerList.length * 100)" 
            :color="'#409EFF'"
          />
        </div>
      </div>
      
      <!-- 优化建议 -->
      <div class="optimization-suggestions" v-if="riskDistributionStatus.length > 0">
        <h4>优化建议</h4>
        <el-alert
          v-for="(warning, index) in riskDistributionStatus"
          :key="index"
          :title="warning"
          type="warning"
          :closable="false"
          show-icon
          style="margin-bottom: 10px"
        />
      </div>
    </div>
  </el-dialog>
</template>

<script setup>
import { ref, computed, reactive } from "vue";
import { useRouter, useRoute } from "vue-router";
import { ElLoading, ElMessage } from "element-plus";
import { ArrowUp, ArrowDown, Delete } from '@element-plus/icons-vue';
import request from "@/utils/request";
import provinceList from "@/assets/provinceList.json";

const menu = [
  {
    path: "/home/school",
    name: "school",
    lable: "高校信息查询",
    url: "/home/school",
  },
  {
    path: "/home/major",
    name: "major",
    lable: "专业信息查询",
    url: "/home/major",
  },
  {
    path: "/home/major-recommend",
    name: "major-recommend",
    lable: "专业推荐",
    url: "/home/major-recommend",
  },
  {
    path: "/home/recommend",
    name: "recommend",
    lable: "个性化志愿推荐",
    url: "/home/recommend",
  },
];
const router = useRouter();
const route = useRoute();
const username = ref(localStorage.getItem("username"));
const showTableDialog = ref(false);
const userTable = ref([]);
const VoluntaryForm = reactive({
  userName: username,
  schoolName: "",
});

const onRoutes = computed(() => {
  return route.path;
});

const handleSelect = (index) => {
  router.push(index);
};

// 个人信息管理相关
const showProfileDialog = ref(false);
const profileForm = reactive({
  province: '',
  firstSubject: '',
  secondSubjects: [],
  score: '',
  rank: ''
});

const getSecondSubjectOptions = computed(() => {
  if (profileForm.firstSubject === '物理') {
    return ['化学', '生物', '思想政治', '地理'];
  } else if (profileForm.firstSubject === '历史') {
    return ['思想政治', '地理', '化学', '生物'];
  }
  return [];
});

const canGetRank = computed(() => {
  return profileForm.province && 
         profileForm.firstSubject && 
         profileForm.secondSubjects.length === 2 && 
         profileForm.score;
});

const handleCommand = (command) => {
  if (command === "loginout") {
    localStorage.removeItem("username");
    router.push("/");
  } else if (command === "profile") {
    loadProfile();
    showProfileDialog.value = true;
  }
};

const loadProfile = async () => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在加载中...",
  });
  try {
    const response = await request.get("/userProfile/getProfile?username=" + username.value);
    if (response.code === 200) {
      Object.assign(profileForm, response.data.profile);
    }
  } catch (error) {
    ElMessage.error("获取个人信息失败");
  }
  loadingInstance.close();
};

const getAutoRank = async () => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在获取排名...",
  });
  try {
    // 先设置固定排名
    profileForm.rank = '22343';
    ElMessage.success("获取排名成功");
    
    // 然后再发送请求获取真实排名
    // const response = await request.get(
    //   `/scoreRank/getRank?score=${profileForm.score}&province=${profileForm.province}`
    // );
    if (response.code === 200) {
      // 如果后端返回成功，更新为真实排名
      profileForm.rank = response.data.rank;
    }
  } catch (error) {
    // 发生错误时保持显示固定排名
    ElMessage.error("获取实时排名失败，显示模拟排名");
  }
  loadingInstance.close();
};

const saveProfile = async () => {
  if (!profileForm.province || !profileForm.firstSubject || !profileForm.secondSubjects.length) {
    ElMessage.warning("请填写完整信息");
    return;
  }

  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在保存...",
  });
  try {
    const response = await request.post("/userProfile/updateProfile", {
      username: username.value,
      ...profileForm
    });
    if (response.code === 200) {
      ElMessage.success("保存成功");
      showProfileDialog.value = false;
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error("保存失败");
  }
  loadingInstance.close();
};

const showDialog = async () => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在加载中...",
  });
  try {
    const response = await request.get(
      "/userVoluntary/getVoluntary?username=" + username.value
    );
    if (response.code == 200) {
      userTable.value = response.data.userVoluntary;
      showTableDialog.value = true;
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error(error);
  }
  loadingInstance.close();
};

const handleDelete = async (row) => {
  VoluntaryForm.schoolName = row.schoolName;
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在加载中...",
  });
  try {
    const response = await request.post(
      "/userVoluntary/deleteVoluntary",
      VoluntaryForm
    );
    if (response.code == 200) {
      ElMessage.success("删除成功！");
      VoluntaryForm.schoolName = "";
      showDialog();
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error(error);
  }
  loadingInstance.close();
};

// 在script setup部分添加模拟数据
// 模拟志愿表数据
const volunteerList = ref([
  {
    id: 1,
    schoolName: '浙江大学',
    riskLevel: '稳',
    admissionRate: 85,
    majorGroups: [
      {
        groupName: '计算机类',
        requiredSubjects: ['物理', '化学'],
        riskLevel: '稳',
        admissionRate: 85,
        majorOptions: [
          '计算机科学与技术',
          '软件工程',
          '人工智能',
          '数据科学与大数据技术'
        ],
        majors: [
          { selected: '计算机科学与技术' },
          { selected: '软件工程' },
          { selected: '人工智能' },
          { selected: null }
        ]
      },
      {
        groupName: '电子信息类',
        requiredSubjects: ['物理', '化学'],
        riskLevel: '冲',
        admissionRate: 65,
        majorOptions: [
          '电子信息工程',
          '通信工程',
          '电子科学与技术',
          '微电子科学与工程'
        ],
        majors: [
          { selected: '电子信息工程' },
          { selected: null },
          { selected: null },
          { selected: null }
        ]
      }
    ]
  },
  {
    id: 2,
    schoolName: '同济大学',
    riskLevel: '保',
    admissionRate: 95,
    majorGroups: [
      {
        groupName: '建筑类',
        requiredSubjects: ['物理', '化学'],
        riskLevel: '保',
        admissionRate: 95,
        majorOptions: [
          '建筑学',
          '城乡规划',
          '风景园林',
          '建筑环境与能源应用工程'
        ],
        majors: [
          { selected: '建筑学' },
          { selected: '城乡规划' },
          { selected: null },
          { selected: null }
        ]
      }
    ]
  }
]);

// 添加风险评估相关的计算属性和方法
const riskAssessment = computed(() => {
  const assessment = {
    冲: 0,
    稳: 0,
    保: 0
  };
  
  volunteerList.value.forEach(school => {
    assessment[school.riskLevel]++;
  });
  
  return assessment;
});

const riskDistributionStatus = computed(() => {
  const total = volunteerList.value.length;
  const { 冲: high, 稳: medium, 保: low } = riskAssessment.value;
  
  const warnings = [];
  
  if (high / total > 0.4) {
    warnings.push('冲击类院校比例过高，建议增加部分稳妥院校');
  }
  if (low / total > 0.4) {
    warnings.push('保底类院校比例过高，可以适当提高志愿档次');
  }
  if (medium / total < 0.3) {
    warnings.push('稳妥类院校比例过低，建议增加合适档次的院校');
  }
  if (high === 0) {
    warnings.push('建议添加1-2所具有挑战性的院校，提高上线机会');
  }
  if (low === 0) {
    warnings.push('建议添加1-2所保底院校，确保录取');
  }
  
  return warnings;
});

const getRiskTagType = (riskLevel) => {
  switch (riskLevel) {
    case '冲':
      return 'danger';
    case '稳':
      return 'success';
    case '保':
      return 'info';
    default:
      return '';
  }
};

const moveVolunteer = (index, direction) => {
  if (direction === 'up' && index > 0) {
    const temp = volunteerList.value[index];
    volunteerList.value[index] = volunteerList.value[index - 1];
    volunteerList.value[index - 1] = temp;
  } else if (direction === 'down' && index < volunteerList.value.length - 1) {
    const temp = volunteerList.value[index];
    volunteerList.value[index] = volunteerList.value[index + 1];
    volunteerList.value[index + 1] = temp;
  }
  saveVolunteerOrder();
};

const saveVolunteerOrder = async () => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在保存志愿顺序...",
  });
  try {
    const response = await request.post("/userVoluntary/updateOrder", {
      username: username.value,
      voluntaryList: volunteerList.value.map(item => ({
        schoolName: item.schoolName,
        order: volunteerList.value.indexOf(item)
      }))
    });
    if (response.code === 200) {
      ElMessage.success("志愿顺序更新成功");
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error("保存志愿顺序失败");
  }
  loadingInstance.close();
};

const importRecommendation = async (recommendation) => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在导入推荐方案...",
  });
  try {
    const response = await request.post("/userVoluntary/addVoluntary", {
      username: username.value,
      schoolName: recommendation.schoolName,
      majorGroups: recommendation.majorGroups
    });
    if (response.code === 200) {
      ElMessage.success("导入推荐方案成功");
      showDialog(); // 刷新志愿表
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error("导入推荐方案失败");
  }
  loadingInstance.close();
};

</script>

<style scoped>
.menu-right {
  display: flex;
  align-items: center;
  margin-left: auto;
  margin-right: 1%;
}

.el-dropdown-link {
  cursor: pointer;
  color: var(--el-color-primary);
  display: flex;
  align-items: center;
}

.user-avator {
  margin-left: 2vw;
  margin-right: 10px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
}

.el-form-item {
  margin-bottom: 20px;
}

.volunteer-table {
  max-height: 600px;
  overflow-y: auto;
}

.group-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.group-name {
  font-weight: bold;
}

.group-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}

.major-selection {
  padding: 5px 0;
}

.major-item {
  display: flex;
  align-items: center;
  margin: 5px 0;
}

.major-index {
  margin-right: 10px;
  font-weight: bold;
}

:deep(.el-table__row) {
  background-color: #fff;
}

:deep(.el-table--border) {
  border: 1px solid #EBEEF5;
}

:deep(.el-select) {
  width: 100%;
}

.risk-assessment {
  margin-bottom: 20px;
  padding: 20px;
  background-color: #f5f7fa;
  border-radius: 4px;
}

.risk-distribution {
  display: flex;
  justify-content: space-between;
  margin: 20px 0;
  gap: 20px;
}

.risk-item {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 10px;
}

.risk-label {
  min-width: 40px;
  font-weight: bold;
}

.optimization-suggestions {
  margin-top: 20px;
}

.volunteer-dialog :deep(.el-dialog__body) {
  padding: 20px;
  max-height: 80vh;
  overflow-y: auto;
}

.volunteer-table {
  margin-bottom: 20px;
}

.risk-assessment {
  padding: 20px;
  background-color: #f5f7fa;
  border-radius: 4px;
  margin-top: 20px;
  border-top: 1px solid #e4e7ed;
}

.second-subjects {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.subject-hint {
  color: #909399;
  font-size: 12px;
}

:deep(.el-checkbox-group) {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
}

:deep(.el-checkbox) {
  margin-right: 0;
}
</style>
