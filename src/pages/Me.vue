<template>
  <el-main>
    <div class="me">
      <el-row>
        <el-col>
          <img class="me-head-img" alt="图片无法加载" src="https://cube.elemecdn.com/6/94/4d3ea53c084bad6931a56d5158a48jpeg.jpeg"/>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" class="info-row">
        <el-col class="avatar-box">
          <el-tooltip class="item" effect="dark" content="修改头像" placement="bottom">
            <el-button class="avatar-btn" @click="avatarDialogVisible=true">
              <img :src="avatarUrl" class="avatar-img" alt="图片无法加载"/>
            </el-button>
          </el-tooltip>
        </el-col>
      </el-row>
      <div style="margin-top: -50px">
        <el-row type="flex" justify="space-between">
          <el-col :span="24">
            <h1 style="font-size: 36px">{{this.$store.getters.getCurUser.nick}} 的个人主页 </h1>
          </el-col>
        </el-row>
        <el-row style="height: 150px">
          <el-col :span="14">
            <el-descriptions class="margin-top" title="个人信息" :column="9" direction="vertical">
              <el-descriptions-item label="id" span="1">{{this.$store.getters.getCurUser.uid}}</el-descriptions-item>
              <el-descriptions-item label="邮箱" span="2">{{this.$store.getters.getCurUser.mail}}</el-descriptions-item>
              <el-descriptions-item label="学校" span="2" v-if="$store.getters.getCurUser.type !== 0">{{this.school}}</el-descriptions-item>
              <el-descriptions-item label="学院" span="3" v-if="$store.getters.getCurUser.type !== 0">{{this.college}}</el-descriptions-item>
              <el-descriptions-item label="类型" span="1">
                <el-tag size="medium" type="danger" v-if="$store.getters.getCurUser.type === 0">管理员</el-tag>
                <el-tag size="medium" type="success" v-if="$store.getters.getCurUser.type === 1">教师</el-tag>
                <el-tag size="medium" v-if="$store.getters.getCurUser.type === 2">学生</el-tag>
              </el-descriptions-item>
            </el-descriptions>
          </el-col>
          <el-col :span="10">
            <el-row>
              <el-col>
                <h4 style="margin-top: 0; margin-bottom: 3%">注册统计</h4>
              </el-col>
            </el-row>
            <el-row type="flex" justify="space-around">
              <el-col>
                <div id="tagStatistic" style="width: 100%; height: 150px"></div>
              </el-col>
            </el-row>
          </el-col>
        </el-row>
        <el-row>
          <el-col>
            <span style="font-size: 24px; margin: 0 0; font-weight: bold">兴趣预测 </span>
            <interest-dialog></interest-dialog>
          </el-col>
        </el-row>
        <el-row>
          <el-col>
            <div id="intentionBarChart" style="height: 300px; width: 100%"></div>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
            <MainRecommendTitle title="我注册的"></MainRecommendTitle>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
            <MyRegisters></MyRegisters>
          </el-col>
        </el-row>
      </div>
    </div>
    <el-dialog :visible.sync="avatarDialogVisible" title="修改头像" width="25%">
      <div class="avatar-dialog">
        <el-upload
            class="avatar-uploader"
            action="/"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
            :http-request="avatarUpload">
          <img v-if="imageUrl" :src="imageUrl" class="avatar" alt="图片加载错误"/>
          <div v-else class="avatar-uploader-icon">
            <i class="el-icon-plus"></i>
          </div>
        </el-upload>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
      </div>
    </el-dialog>
  </el-main>
</template>

<script>
import MainRecommendTitle from "@/components/MainRecommendTitle";
import MyRegisters from "@/components/MyRegisters";
import InterestDialog from "@/components/InterestDialog";

export default {
  name: "Me",
  components: {InterestDialog, MainRecommendTitle, MyRegisters},
  async created() {
    if (this.$store.getters.getCurUser.avatar !== '') this.avatarUrl = 'http://localhost:8000/media/' + this.$store.getters.getCurUser.avatar
    else this.avatarUrl = 'http://localhost:8000/media/web/img/avatar/default.jpg';
    if (this.$store.getters.getCurUser.type !== 0) {
      this.getSchoolCollege()
    }
  },
  mounted() {
    this.drawHobbiesPie();
    this.drawIntentionBar();
  },
  data() {
    return {
      imageUrl: '',
      avatarUrl: 'http://localhost:8000/media/web/img/avatar/default.jpg',
      avatarDialogVisible: false,
      hobbies: [],
      hobbiesName: [],
      school: '',
      college: '',
      interestDialogVisible: false
    }
  },
  methods: {
    getSchoolCollege() {
      this.$axios({
        url: 'users/get_school_college',
        params: {
          uid: this.$store.getters.getCurUser.uid
        },
        method: 'get'
      }).then((res)=>{
        this.school = res.data.school;
        this.college = res.data.college;
      })
    },
    avatarUpload(params) {
      console.log(params)
      if (this.$store.getters.getToken !== '') {
        let upUrl = 'users/update_avatar/' + this.$store.getters.getCurUser.uid
        let form = new FormData();
        form.append("avatar", params.file);
        this.$axios({
          url: upUrl,
          method: 'POST',
          data: form
        }).then((res) => {
          this.avatarUrl = 'http://localhost:8000/media/' + res.data.avatar;
          this.avatarDialogVisible = false;
          // 更新当前用户内容
          let cur_user = this.$store.getters.getCurUser
          cur_user.avatar = res.data.avatar;
          this.$store.commit('setCurUser', cur_user)
        }).catch((err) => {
          console.log(err);
          this.$message.error(err)
        })
      }
    },
    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw);
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg';
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!');
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!');
      }
      return isJPG && isLt2M;
    },
    async drawHobbiesPie() {
      let hobbiesName = [], hobbies = [];
      await this.$axios({
        url: 'users/get_hobbies',
        params: {
          uid: this.$store.getters.getCurUser.uid
        },
        method: 'get'
      }).then((res) => {
        for (let i in res.data.hobbies) {
          hobbiesName.push(i);
          hobbies.push({value: res.data.hobbies[i], name: i})
        }
      }).catch((err)=>{
        console.log(err)
      })
      let option = {
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)'
        },
        legend: {
          orient: 'horizontal',
          top: -5,
          left: 0,
          data: hobbiesName
        },
        series: [
          {
            name: '注册次数',
            type: 'pie',
            radius: ['50%', '70%'],
            avoidLabelOverlap: false,
            label: {
              show: false,
              position: 'center'
            },
            emphasis: {
              label: {
                show: true,
                fontSize: '15',
                fontWeight: 'bold'
              }
            },
            labelLine: {
              show: false
            },
            data: hobbies
          }
        ]
      };
      let tagPieChart = this.$echarts.init(document.getElementById("tagStatistic"));
      tagPieChart.setOption(option);
    },
    async drawIntentionBar() {
      let intentionList = [], typeName = [];
      await this.$axios({
        url: 'class/intention',
        params: {
          uid: this.$store.getters.getCurUser.uid
        },
        method: 'get'
      }).then((res) => {
        intentionList = res.data.intention;
        for (let i in intentionList) {
          intentionList[i] = intentionList[i].toFixed(2);
          if (intentionList[i] > 30) {
            intentionList[i] = {
              value: intentionList[i],
              itemStyle: {
                color: '#FCD900'
              }
            }
          }
          else if (intentionList[i] > 10) {
            intentionList[i] = {
              value: intentionList[i],
              itemStyle: {
                color: '#E8630A'
              }
            }
          }
        }
        typeName = res.data.typeName;
      }).catch((err) => {
        console.log(err)
      });
      let option = {
        tooltip: {
          trigger: 'item',
          formatter: '{b}: {c}%'
        },
        xAxis: {
          data: typeName,
          axisLabel: {
            interval: 0,
            formatter: function (value) {
              return value.split('').join('\n')
            }
          }
        },
        yAxis: {},
        series: [
          {
            type: 'bar',
            data: intentionList,
            barWidth: '50%'
          }
        ]
      };
      let intentionBarChart = this.$echarts.init(document.getElementById("intentionBarChart"));
      intentionBarChart.setOption(option);
    }
  }
}
</script>
<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>

<style scoped>
.me {
  margin-left: 7%;
  margin-right: 7%;
}
.me-head-img {
  width: 100%;
  height: 200px;
  object-fit: cover;
}
.info-row {
  transform: translate(0,-50%);
}
.avatar-btn {
  background: none;
  border: none;
  padding: 0;
}
.avatar-btn:hover {
  background: none;
  border: none;
  padding: 0;
}
.avatar-img {
  object-fit: cover;
  border-radius: 50%;
  border: 2px solid #f5f7fa;
  width: 8rem;
  height: 8rem;
}
.avatar-box {
  width: 10%;
  display: flex;
  justify-content: center;
}
.avatar-dialog {
  display: flex;
  justify-content: center;
}
.avatar-col {
  margin: auto;
}
</style>