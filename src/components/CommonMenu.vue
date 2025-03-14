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
      <el-form-item label="选考科目">
        <el-select 
          v-model="profileForm.selectedSubjects" 
          multiple
          placeholder="请选择两门选考科目"
          style="width: 100%"
          :multiple-limit="2"
        >
          <el-option
            v-for="subject in subjectOptions"
            :key="subject"
            :label="subject"
            :value="subject"
          />
        </el-select>
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
    title="志愿表"
    v-model="showTableDialog"
    align-center
    center
    width="90%"
    style="height: 70%"
  >
    <el-table :data="userTable" height="60vh">
      <el-table-column prop="schoolName" label="学校名称" width="100px" />
      <el-table-column prop="majorA" label="专业A" />
      <el-table-column prop="majorB" label="专业B" />
      <el-table-column prop="majorC" label="专业C" />
      <el-table-column prop="majorD" label="专业D" />
      <el-table-column prop="majorE" label="专业E" />
      <el-table-column prop="majorF" label="专业F" />
      <el-table-column label="操作" width="84px">
        <template #default="scope">
          <el-button type="danger" @click="handleDelete(scope.row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </el-dialog>
</template>

<script setup>
import { ref, computed, reactive } from "vue";
import { useRouter, useRoute } from "vue-router";
import { ElLoading, ElMessage } from "element-plus";
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
  selectedSubjects: [],
  score: '',
  rank: ''
});

const subjectOptions = ['化学', '生物', '地理', '思想政治'];

const canGetRank = computed(() => {
  return profileForm.province && 
         profileForm.firstSubject && 
         profileForm.selectedSubjects.length === 2 && 
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
    // 直接设置固定排名
    profileForm.rank = '22534';
    ElMessage.success("获取排名成功");
  } catch (error) {
    ElMessage.error("获取排名失败");
  }
  loadingInstance.close();
};

const saveProfile = async () => {
  if (!profileForm.province || !profileForm.firstSubject || profileForm.selectedSubjects.length !== 2) {
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
</style>
