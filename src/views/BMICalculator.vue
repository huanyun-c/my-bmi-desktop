<template>
  <div id="wrapper">
    <el-container>
      <el-header>
        <span class="back-index" title="返回首页" @click="backIndex()">
          <i class="el-icon-back"></i>
        </span>
        <span>BMI计算器</span>
      </el-header>
      <el-main>
        <div class="note-wrapper">
          <img class="ico" width="25" height="25" src="@/assets/note_icon.png" />
          <p class="p1">通过身高体重计算你的BMI值，从而得出你的身体状况评估，高于24你就是超重啦。</p>
          <p class="p2">（*本工具仅适合正常非孕期健康成年人）</p>
        </div>
        <div class="bmi-qa">
          <el-popover
            placement="top-start"
            title="BMI小常识"
            width="200"
            trigger="hover"
            content="身体质量指数（BMI，Body Mass Index）是国际上常用的衡量人体肥胖程度和是否健康的重要标准，主要用于统计分析。
            肥胖程度的判断不能采用体重的绝对值，它天然与身高有关。因此，BMI 通过人体体重和身高两个数值获得相对客观的参数，并用这个参数所处范围衡量身体质量。"
          >
            <span slot="reference">什么是BMI？</span>
          </el-popover>
        </div>
        <div class="form-wrapper">
          <span class="form-title">请输入个人信息</span>
          <div class="form-note">注：由于BMI值短时间波动稳定，每天只会保存一次计算结果，多次计算会覆盖同一天之前的结果。</div>
          <el-form :model="bmiForm" :rules="rules" ref="bmiForm" label-width="100px" class="bmiorm">
            <el-form-item label="身高(cm)：" prop="height">
              <el-input-number
                v-model="bmiForm.height"
                :precision="2"
                :step="0.1"
                size="medium"
                :min="0"
                controls-position="right"
                placeholder="身高, 自动保留2位小数"
              ></el-input-number>
            </el-form-item>
            <el-form-item label="体重(kg)：" prop="weight">
              <el-input-number
                v-model="bmiForm.weight"
                :precision="2"
                :step="0.1"
                size="medium"
                :min="0"
                controls-position="right"
                placeholder="体重, 自动保留2位小数"
              ></el-input-number>
            </el-form-item>
            <el-form-item label prop="isSaveBMI">
              <el-switch v-model="isSaveBMI" active-text="是否保存本次记录"></el-switch>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="submitForm('bmiForm')">计算</el-button>
              <el-button @click="resetForm('bmiForm')">重置</el-button>
            </el-form-item>
          </el-form>
          <div class="line"></div>
          <div class="bmi-note">
            <p>BMI计算公式：体质指数（BMI）= 体重（kg）÷ 身高^2（m）</p>
            <table class="bmi-table" style="border: none;">
              <thead class="bmi-table-head">
                <tr>
                  <th>BMI分类</th>
                  <th>WHO标准</th>
                  <th>亚洲标准</th>
                  <th>中国参考标准</th>
                </tr>
              </thead>
              <tbody class="bmi-table-body">
                <tr>
                  <td>偏瘦</td>
                  <td>{{" < 18.5"}}</td>
                  <td>{{" < 18.5"}}</td>
                  <td>{{" < 18.5"}}</td>
                </tr>
                <tr>
                  <td>正常</td>
                  <td>18.5～24.9</td>
                  <td>18.5～22.9</td>
                  <td>18.5～23.9</td>
                </tr>
                <tr>
                  <td>超重</td>
                  <td>≥25</td>
                  <td>≥23</td>
                  <td>≥24</td>
                </tr>
                <tr>
                  <td>偏胖</td>
                  <td>25.0～29.9</td>
                  <td>23～24.9</td>
                  <td>24～26.9</td>
                </tr>
                <tr>
                  <td>肥胖</td>
                  <td>30.0～34.9</td>
                  <td>25～29.9</td>
                  <td>27～29.9</td>
                </tr>
                <tr>
                  <td>重度肥胖</td>
                  <td>35.0～39.9</td>
                  <td>≥30</td>
                  <td>≥30</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
        <div class="table-wrapper">
          <div style="margin-bottom: 20px;">
            <span class="form-title">我的BMI历史记录：</span>
            <span style="float:right;">
              <el-button-group>
                <el-button
                  type="primary"
                  size="mini"
                  icon="el-icon-download"
                  @click="exportData"
                >导出历史记录</el-button>
                <el-button
                  type="primary"
                  size="mini"
                  icon="el-icon-delete"
                  @click="clearData"
                >清空历史记录</el-button>
              </el-button-group>
            </span>
            <div class="search-warpper">
              <label>开始日期：</label>
              <el-date-picker
                v-model="search.dateRange"
                type="daterange"
                align="right"
                size="small"
                unlink-panels
                range-separator="至"
                start-placeholder="开始日期"
                end-placeholder="结束日期"
                :picker-options="datePickerOptions"
                @change="searchData"
              ></el-date-picker>
            </div>
          </div>

          <table class="bmi-table">
            <thead class="bmi-table-head">
              <colgroup>
                <col width="20%" />
                <col width="20%" />
                <col width="20%" />
                <col width="20%" />
                <col width="20%" />
              </colgroup>
              <tr>
                <th>日期</th>
                <th>身高</th>
                <th>体重</th>
                <th>BMI</th>
                <th>操作</th>
              </tr>
            </thead>
            <tbody class="bmi-table-body">
              <tr v-for="(item, index) in tableData" :key="index">
                <td>{{item.date}}</td>
                <td>{{item.height + "cm"}}</td>
                <td>{{item.weight + "kg"}}</td>
                <td>{{item.bmi}}</td>
                <td>
                  <el-button type="text" @click="handleDelete(item)" size="small">删除</el-button>
                </td>
              </tr>
            </tbody>
          </table>
          <el-pagination background layout="prev, pager, next" :page-size="10" :total="total"></el-pagination>
        </div>
        <div class="chart-wrapper">
          <span class="form-title">体重/BMI趋势图示：</span>
          <ve-line :data="chartData" :settings="chartSettings"></ve-line>
        </div>
      </el-main>
      <el-footer>©2019 - BMI身体指数计算器桌面版 版权所有 © 柳叶刀</el-footer>
    </el-container>
    <el-dialog :visible.sync="dialogVisible" width="30%">
      <div class="dig-body">
        <img class="start-img" src="@/assets/start.gif" alt="scroll" />
        <p>春小云，</p>
        <p>你，又，瘦了！</p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "BMICalculator",
  components: {},
  data() {
    return {
      isSaveBMI: true,
      bmiForm: {
        height: "",
        weight: ""
      },
      rules: {
        height: [{ required: true, message: "请输入身高", trigger: "blur" }],
        weight: [{ required: true, message: "请输入体重", trigger: "blur" }]
      },
      tableData: [],
      total: 0,
      chartData: {
        columns: ["日期", "体重", "BMI"],
        rows: []
      },
      chartSettings: { dimension: ["日期"] },
      search: {
        dateRange: ""
      },
      datePickerOptions: {
        shortcuts: [
          {
            text: "最近一周",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            }
          }
        ]
      },
      dialogVisible: false
    };
  },
  created() {
    this.$bus.on("get-bmi", () => {
      this.initData();
    });
    this.initDatePicker();
  },
  mounted() {
    this.initData();
  },
  methods: {
    initDatePicker() {
      let dateRange = [];
      const end = new Date();
      const start = new Date();
      start.setTime(start.getTime() - 3600 * 1000 * 24 * 10);
      dateRange.push(start, end);
      this.search.dateRange = dateRange;
    },
    initData() {
      let startDate = this.utils.getDate(this.search.dateRange[0]);
      let endDate = this.utils.getDate(this.search.dateRange[1]);
      console.log("initData", startDate, endDate);

      let bmi = localStorage.getItem("bmi");
      if (bmi && bmi != "[]") {
        let bmis = JSON.parse(bmi);
        bmis = bmis.filter(item => {
          return item.date >= startDate && item.date <= endDate;
        });
        this.tableData = bmis;
        this.total = bmis.length;
        for (let i = 0; i < bmis.length; i++) {
          let item = {
            日期: bmis[i].date,
            体重: bmis[i].weight,
            BMI: bmis[i].bmi
          };
          this.chartData.rows.push(item);
        }
      } else {
        this.tableData = [];
        this.chartData.rows = [];
      }
    },
    searchData() {
      let startDate = this.utils.getDate(this.search.dateRange[0]);
      let endDate = this.utils.getDate(this.search.dateRange[1]);
      console.log("searchData", startDate, endDate);
      let bmi = localStorage.getItem("bmi");
      if (bmi) {
        let bmis = JSON.parse(bmi);
        bmis = bmis.filter(item => {
          return item.date >= startDate && item.date <= endDate;
        });
        this.tableData = bmis;
        this.total = bmis.length;
        this.chartData.rows = [];
        for (let i = 0; i < bmis.length; i++) {
          let item = {
            日期: bmis[i].date,
            体重: bmis[i].weight,
            BMI: bmis[i].bmi
          };
          this.chartData.rows.push(item);
        }
      }
    },
    submitForm(formName) {
      let height = this.bmiForm.height;
      let weight = this.bmiForm.weight;
      if (height && weight && height > 0 && weight > 0) {
        this.$refs[formName].validate(valid => {
          if (valid) {
            let bmiValue =
              this.bmiForm.weight / Math.pow(this.bmiForm.height * 0.01, 2);
            this.saveBMI(bmiValue);
          } else {
            return false;
          }
        });
      } else {
        this.$message({
          message: "请输入身高和体重",
          type: "warning"
        });
      }
    },
    saveBMI(bmi) {
      let d = new Date();
      let item = {
        id: d.getTime(),
        date: this.utils.getDate(),
        height: this.bmiForm.height,
        weight: this.bmiForm.weight,
        bmi: bmi.toFixed(1)
      };
      let isSaveBMI = this.isSaveBMI;
      if (isSaveBMI) {
        let bmis = localStorage.getItem("bmi");
        if (bmis) {
          let data = JSON.parse(bmis);
          data = data.filter(bmi => {
            return bmi.date != item.date;
          });
          data.push(item);
          localStorage.setItem("bmi", JSON.stringify(data));
        } else {
          localStorage.setItem("bmi", JSON.stringify([item]));
        }
        this.$bus.emit("get-bmi");
      } else {
        this.tableData.push(item);
      }

      this.dialogVisible = true;
    },
    clearData() {
      this.$confirm(
        "此操作将删除所有历史记录, 数据无法恢复，请谨慎选择。是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      )
        .then(() => {
          localStorage.removeItem("bmi");
          this.$message({
            type: "success",
            message: "清除数据成功!"
          });
          this.initData();
        })
        .catch(e => {
          console.log(e);
        });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    handleDelete(deletedItem) {
      this.$confirm("此操作将永久删该条记录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.deleteBmiItem(deletedItem);
        })
        .catch(() => {});
    },
    deleteBmiItem(deletedItem) {
      let bmi = localStorage.getItem("bmi");
      let bmis = JSON.parse(bmi);
      bmis = bmis.filter(item => {
        return item.date != deletedItem.date;
      });
      localStorage.setItem("bmi", JSON.stringify(bmis));
      let tableData = this.tableData.filter(item => {
        return item.id != deletedItem.id;
      });
      this.tableData = tableData;
      this.$message({
        type: "success",
        message: "删除成功!"
      });
      // this.initData();
      // this.$bus.emit("get-bmi");
    },
    backIndex() {
      this.$router.replace({ path: "/" });
    },

    exportData() {
      let bmi = localStorage.getItem("bmi");
      let bmis = JSON.parse(bmi);
      if (bmi && bmi != "[]") {
      }
      let contentString =
        ["编号", "日期", "身高", "体重", "BMI"].join(",") + "\r\n";
      bmis.forEach(item => {
        let id = item.id;
        let date = item.date;
        let height = item.height + "cm";
        let weight = item.weight + "kg";
        let bmi = item.bmi;
        contentString += `${id},${date},${height},${weight},${bmi}\r\n`;
      });
      this.utils.download(contentString, "BMI数据记录");
    }
  },
  beforeDestroy() {
    this.$bus.off("get-bmi", () => {});
  }
};
</script>

<style scoped lang="scss">
#wrapper {
  display: block;
}

.el-header {
  -webkit-app-region: drag;
  height: 40px !important;
  text-align: center;
  font-size: 22px;
  line-height: 40px;
  color: white;
  background: #5f87d8;
  position: fixed;
  width: 100%;
  z-index: 100;
  padding: 0 5px;
  .back-index {
    -webkit-app-region: no-drag;
    float: left;
    cursor: pointer;
  }
}
.el-main {
  margin-top: 50px;
}
.note-wrapper {
  background: #fff;
  background-size: 25px auto;
  padding: 10px 15px 5px 0;
  position: relative;
}

.p1 {
  display: inline-block;
  vertical-align: text-bottom;
  color: #409eff;
  text-indent: 0.5em;
}

.p2 {
  color: #f74444;
  font-size: 14px;
  margin: 5px 0 0 0;
  display: inline;
  vertical-align: text-bottom;
}

.bmi-qa {
  float: right;
  font-size: 14px;
  color: #409eff;
  cursor: pointer;
}

.form-wrapper {
  margin-top: 30px;
  padding: 10px;
  background: #fff;
  border-radius: 6px;
  border: 1px solid #ededed;
  position: relative;
}

form {
  margin-top: 20px;
}

.form-title {
  font-size: 16px;
  display: inline-block;
  color: #409eff;
}

.form-note {
  color: rgb(169, 169, 169);
  font-size: 13px;
  width: 50%;
}

.el-form-item {
  width: 340px;
}

.line {
  border-right: 1px solid #dddddd;
  position: absolute;
  left: 50%;
  height: 370px;
  top: 0;
}

.bmi-note {
  position: absolute;
  top: 0;
  right: 0;
  width: 50%;
  text-align: center;

  p {
    font-size: 16px;
    padding: 10px;
    color: #409eff;
    margin-top: 20px;
    margin-bottom: 20px;
  }
}

.table-wrapper {
  margin-top: 40px;
}

.bmi-table {
  width: 100%;
  border-bottom: 1px solid #ebebeb;
  font-size: 14px;
}

.bmi-table-head {
  color: #909399;
  font-weight: 500;
  background-color: #f5f5f5;
}

.bmi-table-body {
  text-align: center;
  color: #606266;
}
.search-warpper {
  display: inline-block;
  font-size: 14px;
  margin-left: 10%;
}
.el-pagination {
  text-align: center;
  margin-top: 15px;
}
.chart-wrapper {
  margin-top: 20px;
}
.el-footer {
  height: 40px !important;
  text-align: center;
  font-size: 16px;
  line-height: 40px;
  color: white;
  background: #5f87d8;
  width: 100%;
}
.dig-body {
  text-align: center;
  font-size: 20px;
  img {
    width: 180px;
    height: 180px;
  }
}
.el-dialog__footer {
  text-align: center !important;
}
</style>
