<template>
  <el-form label-width="60px" label-position="left">
    <div class="rank-box">
      <el-form-item label="省份">
        <el-select
            v-model="province"
            filterable
            clearable
            placeholder="请选择省份"
            size="large"
            style="width: 240px"
        >
          <el-option
              v-for="item in provinceList"
              :key="item"
              :label="item"
              :value="item"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="喜欢的专业" label-width="100px" style="margin-left: 2vw">
        <el-select
            v-model="fondMajor"
            filterable
            clearable
            placeholder="请选择专业"
            size="large"
            style="width: 240px"
            multiple
        >
          <el-option
              v-for="item in majorTypeList"
              :key="item"
              :label="item"
              :value="item"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="不喜欢的专业" label-width="100px" style="margin-left: 2vw">
        <el-select
            v-model="dislikedMajor"
            placeholder="请选择专业"
            filterable
            clearable
            size="large"
            style="width: 240px"
            multiple
        >
          <el-option
              v-for="item in majorTypeList"
              :key="item"
              :label="item"
              :value="item"
          />
        </el-select>
      </el-form-item>
      <el-button
        type="primary"
        style="margin-left: 4vw"
        @click="getRecommendList"
      >
        <el-icon>
          <Search />
        </el-icon>
        推荐
      </el-button>
    </div>
    <el-form-item label="风险">
      <el-radio-group v-model="risk">
        <el-radio-button label="全部" value="全部" />
        <el-radio-button label="可冲击" value="可冲击" />
        <el-radio-button label="较稳妥" value="较稳妥" />
        <el-radio-button label="可保底" value="可保底" />
      </el-radio-group>
    </el-form-item>
  </el-form>
  <div class="school-list">
    <ul>
      <li v-for="(school, index) in schoolList" :key="school.schoolId">
        <el-card
          class="schoolCard"
          shadow="hover"
          @click="goToDetail(school.schoolId)"
        >
          <div class="school-info">
            <div class="school-image">
              <el-image
                :src="
                  'https://static-data.gaokao.cn/upload/logo/' +
                  school.schoolId +
                  '.jpg'
                "
                style="width: 100px; height: 100px"
                alt="school-logo"
              />
            </div>
            <div class="school-detail">
              <p>{{ school.schoolName }}</p>
              <span>&nbsp;&nbsp;最低位次：{{ school.rank2022 }}</span>
              <span>&nbsp;&nbsp;预测投档线：{{ averageScores[index] }}</span>
              <span>&nbsp;&nbsp;录取概率：&nbsp;&nbsp;
<!--                <el-icon-->
<!--                  size="20px"-->
<!--                  :color="-->
<!--                    upLineRateList[index] < 60-->
<!--                      ? '#FF0000'-->
<!--                      : upLineRateList[index] >= 80-->
<!--                      ? '#21c33c'-->
<!--                      : '#409eff'-->
<!--                  "-->
<!--                >-->
<!--                  {{-->
<!--                    upLineRateList[index] == 0 ? "<25" : upLineRateList[index]-->
<!--                  }}%-->
<!--                </el-icon>-->
                <el-icon
                    size="20px"
                    :color="
                    school.maxProbability < 60
                      ? '#FF0000'
                      : school.maxProbability >= 80
                      ? '#21c33c'
                      : '#409eff'
                  ">
                  {{
                    school.maxProbability == 0 ? "<25" :school.maxProbability
                  }}%
                </el-icon>
              </span>
            </div>
          </div>

          <div class="major-group-info">
            <ul>
              <li class="major-group-item radius" v-for="(majorGroup, index) in school.majorGroupList" :key="majorGroup.id">
                <el-row>
                  <el-col :span="6">
                    <span> 专业组&nbsp;{{majorGroup.id}}</span>
                  </el-col>
                  <el-col :span="6">
                    <span v-for="(subjectRequirement, index) in majorGroup.subjectRequirements" :key="subjectRequirement">&nbsp;&nbsp;
                      {{ subjectRequirement }}
                    </span>
                  </el-col>
                  <el-col :span="6">
                    <span>&nbsp;
                      <el-icon
                          size="20px"
                          :color="
                          majorGroup.probability < 60
                            ? '#FF0000'
                            : majorGroup.probability >= 80
                            ? '#21c33c'
                            : '#409eff'
                        "
                      >
                         {{majorGroup.probability}}%
                      </el-icon>
                    </span>
                  </el-col>
                  <el-col :span="6">
                    <el-button
                        size="large"
                        style="margin: auto 0"
                        @click.stop="handleCommit(school.schoolId)"
                    >
                      +志愿表
                    </el-button>
                  </el-col>
                </el-row>

              </li>
            </ul>
          </div>
        </el-card>
      </li>
    </ul>
  </div>
  <el-pagination
    class="pagination"
    background
    v-model:current-page="pageNum"
    :total="total"
    layout="prev, pager, next, jumper"
    @current-change="handleCurrentChange"
  />
  <el-dialog
    title="专业表"
    v-model="showTableDialog"
    align-center
    center
    style="height: 70%"
  >
    <el-table
      ref="multipleTable"
      :data="schoolMajors"
      height="55vh"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="30px" />
      <el-table-column prop="majorName" label="专业名称" />
      <el-table-column prop="batch" label="录取批次" width="80px" />
      <el-table-column prop="level2" label="学科类别" width="80px" />
      <el-table-column prop="level3" label="专业类别" width="80px" />
      <el-table-column prop="min" label="最低分" width="66px" />
      <el-table-column prop="minSection" label="最低位次" width="80px" />
    </el-table>
    <div class="button-container">
      <el-button @click="clearSelection()">清除所有</el-button>
      <el-button type="primary" @click="commitVoluntary()">提交</el-button>
    </div></el-dialog
  >
</template>

<script setup>
import { onMounted, ref, reactive, watch } from "vue";
import { ElLoading, ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import request from "../utils/request.js";
import provinceList from "@/assets/provinceList.json";
import majorTypeList from "@/assets/majorTypeList.json";

const subject = ref("1");
const fondMajor = ref("1");
const dislikedMajor = ref("1");
const province = ref("福建");
const userScore = ref(600);
const userRank = ref("");
const risk = ref("全部");
const schoolList = ref([]);
const averageScores = ref([]);
const upLineRateList = ref([]);
const pageNum = ref(1);
const total = ref(0);
const schoolMajors = ref([]);
const showTableDialog = ref(false);
const multipleTable = ref(null);
const VoluntaryForm = reactive({
  userName: localStorage.getItem("username"),
  schoolName: "",
  majorA: "",
  majorB: "",
  majorC: "",
  majorD: "",
  majorE: "",
  majorF: "",
});
const router = useRouter();

watch(risk, () => {
  pageNum.value = 1;
  getRecommendList();
});

const goToDetail = (id) => {
  router.push({ name: "schoolDetail", params: { id: id } });
};

function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

const getRecommendList = async () => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在加载中...",
  });
  try {
    const response = await request.get(
      "/scoreRank/getReco?page=" +
        pageNum.value +
        "&score=" +
        userScore.value +
        "&risk=" +
        risk.value
    );
    if (response.code == 200) {
      schoolList.value = response.data.schools;

      for (let i = 0; i < schoolList.value.length; i++) {
        let school = schoolList.value[i];
        school.majorGroupList = [];
        let num = getRandomInt(1,3);
        school.maxProbability = -1;
        for (let j = 0; j < num; j++) {
          let item = {
            id: 200 + getRandomInt(1,9),
                subjectRequirements: [
                  '物','生'
              ],
              probability: getRandomInt(50,90)
          }
          school.majorGroupList.push(item);

          if (item.probability > school.maxProbability) {
            school.maxProbability = item.probability;
          }
        }
      }

      averageScores.value = response.data.averageScores;
      upLineRateList.value = response.data.upLineRateList;
      total.value = response.data.total;
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error(error);
  }
  loadingInstance.close();
};

const getRank = async () => {
  if (userScore.value == "") {
    ElMessage.error("请输入分数！");
    return;
  }
  if (userScore.value < 0 || userScore.value > 750) {
    ElMessage.error("请输入正确的分数！");
    return;
  }
  if (userScore.value < 100 || userScore.value > 695) {
    ElMessage.error("无数据");
    return;
  }
  try {
    const response = await request.get(
      "/scoreRank/getRank?score=" + userScore.value
    );
    if (response.code == 200) {
      userRank.value = response.data.scoreRank.rankRange;
      ElMessage.success("查询排名成功！");
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error(error);
  }
};

const handleCurrentChange = (newPage) => {
  pageNum.value = newPage;
  getRecommendList();
};

const handleCommit = async (schoolId) => {
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在加载中...",
  });
  try {
    const response = await request.get(
      "/schoolInfoDetail?schoolId=" + schoolId
    );
    if (response.code == 200) {
      schoolMajors.value = response.data.majorScore;
      VoluntaryForm.schoolName = response.data.schoolInfo.schoolName;
      showTableDialog.value = true;
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error(error);
  }
  loadingInstance.close();
};

const clearSelection = () => {
  multipleTable.value.clearSelection();
};

const handleSelectionChange = (val) => {
  if (val.length > 6) {
    ElMessage.warning("最多选择六个专业！");
    return;
  }
  VoluntaryForm.majorA =
    val.map((item) => item.majorName).length > 0
      ? val.map((item) => item.majorName)[0]
      : "";
  VoluntaryForm.majorB =
    val.map((item) => item.majorName).length > 1
      ? val.map((item) => item.majorName)[1]
      : "";
  VoluntaryForm.majorC =
    val.map((item) => item.majorName).length > 2
      ? val.map((item) => item.majorName)[2]
      : "";
  VoluntaryForm.majorD =
    val.map((item) => item.majorName).length > 3
      ? val.map((item) => item.majorName)[3]
      : "";
  VoluntaryForm.majorE =
    val.map((item) => item.majorName).length > 4
      ? val.map((item) => item.majorName)[4]
      : "";
  VoluntaryForm.majorF =
    val.map((item) => item.majorName).length > 5
      ? val.map((item) => item.majorName)[5]
      : "";
};

const commitVoluntary = async () => {
  if (VoluntaryForm.majorA == "") {
    ElMessage.warning("最少选择一门专业！");
    return;
  }
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在加载中...",
  });
  try {
    const response = await request.post(
      "/userVoluntary/addVoluntary",
      VoluntaryForm
    );
    if (response.code == 200) {
      ElMessage.success("添加成功！");
      VoluntaryForm.schoolName = "";
      VoluntaryForm.majorA = "";
      VoluntaryForm.majorB = "";
      VoluntaryForm.majorC = "";
      VoluntaryForm.majorD = "";
      VoluntaryForm.majorE = "";
      VoluntaryForm.majorF = "";
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error(error);
  }
  loadingInstance.close();
  showTableDialog.value = false;
};

onMounted(getRank(), getRecommendList());
</script>

<style scoped>
.rank-box {
  display: flex;
}

ul {
  justify-content: center;
  padding-inline-start: 0;
}

li {
  list-style-type: none;
}

.schoolCard {
  margin: 20px auto;
  //height: 140px;
  border-radius: 10px;
}

.schoolCard:hover {
  box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.2);
  background-color: rgb(250, 250, 250);
  cursor: pointer;
}

.school-info {
  display: flex;
}

.school-detail {
  text-align: center;
  width: 100%;
}

.major-group-info {
  text-align: center;
  width: 100%;
}

p {
  font-size: large;
}

.pagination {
  justify-content: center;
}

.button-container {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
}

.major-group-item {
  border-top: 1px solid var(--el-border-color);
  border-radius: 4px;
  margin-top: 10px;
  padding: 10px;
  height: 50px;
  line-height: 50px;
}
</style>
