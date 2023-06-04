<template>
  <div class="monitor">
    <el-container>
      <el-header height="30px">
        <div
          class="title"
          style="float: left; font-size: 30px; font-weight: 700"
        >
          统计&分析 -- {{ monitorPaper.title }}
        </div>
      </el-header>
      <el-divider></el-divider>
      <el-main>
        <div class="analyzeTables" :model="monitorPaper">
          <div style="text-align: left; margin: 5px 0">
            <span style="font-size: 20px; font-weight: 450"
              >问卷状态：{{
                { INIT: '编辑中', START: '已发放', STOP: '已回收' }[
                  monitorPaper.status
                ]
              }}</span
            >
          </div>
          <div style="text-align: left; margin: 5px 0">
            <span
              style="font-size: 20px; font-weight: 450"
              v-if="monitorPaper.endTime != null"
            >
            发放时段：{{ monitorPaper.startTime }} 到 {{ monitorPaper.endTime }}
            </span>
            <span style="font-size: 20px; font-weight: 450" v-else>
              发放时段：人工操作
            </span>
          </div>
          <div style="text-align: left">
            <el-button
              v-if="monitorPaper.status === 'START'"
              type="danger"
              @click="endCollection()"
              >结束收集</el-button
            >
            <el-button
              v-if="monitorPaper.status === 'STOP' || monitorPaper.status === 'INIT'"
              type="warning"
              @click="showResetModal = true"
            >重新发放</el-button>
            <el-dialog title="设定收集时间" :visible.sync="showResetModal">
              <div style="height: 50px; text-align: left; line-height: 50px">
                <el-radio v-model="radio" label="1">手动结束收集</el-radio>
                <el-divider direction="vertical" />
                <el-radio v-model="radio" label="2">开始时间-结束时间</el-radio>
              </div>
              <div
                style="height: 30px; text-align: left; line-height: 30px; margin-bottom: 40px"
                v-if="datepicker"
              >
                <el-date-picker
                  v-model="date"
                  type="daterange"
                  range-separator="至"
                  start-placeholder="开始日期"
                  end-placeholder="结束日期"
                  v-if="datepicker"
                >
                </el-date-picker>
              </div>
              <div slot="footer" class="dialog-footer">
                <el-button @click="showResetModal = false">取 消</el-button>
                <el-button type="primary" @click="resetCollection()"
                  >确 定</el-button
                >
              </div>
            </el-dialog>
          </div>
          <div style="text-align: left; margin-top: 10px;">
            <el-button type="primary" @click="answersVisible = true">查看填写记录</el-button>
          </div>
          <div style="text-align: left; margin-top: 10px;">
            <el-button type="warning" @click="exportExcel">导出Excel结果</el-button>
          </div>
          <el-dialog top="10vh" width="70%" :visible.sync="answersVisible">
            <span slot="title" style="font-size: 25px; font-weight: 450">问卷填写记录</span>
            <el-table :data="answers.rows" max-height="600px" id="export-table">
              <el-table-column property="date" label="填写时间" width="200"></el-table-column>
              <el-table-column v-for="(ques, j) in monitorPaper.questionStatistics" :key="j" :property="'q'+(j+1)" :label="'第'+(j+1)+'题'+ques.title" min-width="300"></el-table-column>
            </el-table>
          </el-dialog>
          <div
            v-for="(question, index) in monitorPaper.questionStatistics"
            :key="question.id"
          >
            <h1 class="qusetionTitle" style="font-size: 20px; font-weight: 700">
              第{{ index + 1 }}题--{{ question.title }}
            </h1>

            <el-tag style="color:rgb(37,124,115);margin-bottom:10px;margin-right:10px;float:left"
              >填写总数：{{ question.filledInNum }}</el-tag
            >
            <div v-if="question.type === 1">
            <div
                style="margin-bottom:10px;margin-right:10px;float:left"
            >
              <el-button @click="openDialog(question.id)" type="success" float="right" size="small">饼状图</el-button>

              <el-dialog title="饼状图" :visible.sync="circleTableVisible" @close="handleDialogClose">
                <div id="main" style="width: 100%; height: 300px;"></div>
                <div slot="footer" class="dialog-footer">
                  <el-button type="primary" @click="circleTableVisible = false">确 定</el-button>
                </div>
              </el-dialog>
              </div>
            <div
                style="margin-bottom:10px;float:left"
            >
              <el-button @click="openDialog2(question.id)" type="success" float="right" size="small">柱状图</el-button>

              <el-dialog title="柱状图" :visible.sync="columnTableVisible" @close="handleDialogClose">
                <div id="main2" style="width: 100%; height: 300px;"></div>
                <div slot="footer" class="dialog-footer">
                  <el-button type="primary" @click="columnTableVisible = false">确 定</el-button>
                </div>
              </el-dialog>
            </div>
            </div>
            <el-table
              v-if="question.type === 1 || question.type == 2"
              :data="question.optionStatistics"
              border
              stripe
              style="width:100%; margin-bottom: 10px"
            >
              <el-table-column
                prop="sequence"
                label="选项"
                sortable
              ></el-table-column>
              <el-table-column prop="content" label="选项描述">
                <template slot-scope="scope">
                  <span style="center">{{
                    scope.row[scope.column.property]
                  }}</span>
                </template>
              </el-table-column>
              <el-table-column
                prop="selectedNum"
                label="选择人数"
                sortable
              ></el-table-column>
              <el-table-column prop="percent" label="比例"> </el-table-column>
            </el-table>

            <el-table
              v-if="question.type === 3"
              :data="question.answerVOList"
              border
              stripe
              style="width: 100%;margin-bottom: 40px"
            >
              <el-table-column
                label="序号"
                type="index"
                width="100"
              ></el-table-column>
              <el-table-column
                label="答案文本"
                prop="answerContent"
              ></el-table-column>
            </el-table>
          </div>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'
import * as echarts from "echarts"
import {reviewPaperAPI} from "@/api/paper/paper";
import XLSX from "xlsx";
import FileSaver from 'file-saver';

export default {
  name: 'Monitor',
  data() {
    return {
      showResetModal: false,
      // datepicker: false,
      answersVisible: false,
      radio: '1',
      date: '',
      columnTableVisible: false,
      circleTableVisible: false,
      dialogFormVisible: false,
      chart:null,
      res2:null
    }
  },
  async mounted() {
    const paperId = this.$route.params.paperId
    let res = await this.getFullPaperStatistic(paperId)
    this.res2 = await reviewPaperAPI(paperId)
    if (!res) {
      this.$message.error('问卷统计加载失败，请询问管理员')
      return
    }
    res = await this.getAnswersRow(paperId)
    console.log(this.answers)
  },
  // 导出当前页

  computed: {
    ...mapGetters(['monitorPaper', 'answers']),
    datepicker() {
      return this.radio === '2'
    },
  },
  methods: {
    exportExcel() {
      this.$confirm("是否确认导出当前问卷所有数据?", "警告", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then((response) => {
        /* const wb = XLSX.utils.table_to_book(
            document.querySelector("answers")
        );*/
        this.$once("hook:updated", () => {
          // 获取表格元素
          const table = document.querySelector("#export-table")
          // 将表格元素转化为工作簿
          const wb = XLSX.utils.table_to_book(table)
          const wbout = XLSX.write(wb, {
            bookType: "xlsx",
            bookSST: true,
            type: "array",
          });
          try {
            FileSaver.saveAs(
                new Blob([wbout], {type: "application/octet-stream"}),
                "问卷数据.xlsx"
            );
          } catch (e) {
            if (typeof console !== "undefined") console.log(e, wbout);
          }
        });

      });
    },
    ...mapActions([
      'getFullPaperStatistic',
      'terminateCollection',
      'restartCollection',
      'getAnswersRow'
    ]),
    endCollection() {
      this.$confirm('确定结束此问卷收集？', '结束收集', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.terminateCollection(this.monitorPaper).then((res) => {
            this.$message({
              type: 'warning',
              message: '问卷收集结束！'
            })
          })
        })
        .catch(() => {})
    },
    resetCollection() {
      const date = this.datepicker ? this.date : null
      console.log(`date: ${date}`)
      this.restartCollection(date).then((res) => {
        if (res) {
          this.$message.success('重新发放成功')
          console.log(this.monitorPaper)
        } else {
          this.$message.error('发放失败')
        }
      })
      this.showResetModal = false
    },
    openDialog(param) {
      this.circleTableVisible = true;
      this.$nextTick(() => {
        this.renderChart(param);
      });
    },
    watch: {
      //观察option的变化
      echarts1_option: {
        handler(newVal, oldVal) {
          if (this.chart) {
            if (newVal) {
              this.chart.setOption(newVal);
            } else {
              this.chart.setOption(oldVal);
            }
          } else {
            this.init();
          }
        },
        deep: true //对象内部属性的监听，关键。
      }
    },
    renderChart(param) {
      //const chartContainer = this.$refs.chartContainer;
      //let dom = this.$refs.charts[0];
      //let dom = document.getElementById("chart");// 获取图表容器元素
      //this.chart = echarts.init(dom); // 初始化图表实例
      //不要用refs this.chart = echarts.init(this.$refs.chartContainer);
      this.chart = echarts.init(document.getElementById('main'));

      // 将百分比数据转换为 echarts 饼图所需的格式
      let question, option
      const paperId = this.$route.params.paperId
      let optionArray = []; // 定义一个空数组，用来存储所有的选项
      if (this.res2 && this.res2.data.success) {
        const paperStatistic = this.res2.data.content
        for (question of paperStatistic.questionStatistics) {
          if (question.type === 3) {
            continue
          }
          const total = question.filledInNum
          for (option of question.optionStatistics) {
            option.percent = (Math.floor(option.selectedNum / total * 1000) / 10);
            if(option.questionId === param) {
              optionArray.push(option);
            }
          }
        }

        console.log("test1");
        console.log(option.content);
        console.log(option.percent);
        //var data = null;
        //data.name = option.content;
        //data.value = option.percent;
        let data = optionArray.map(o => ({value: o.percent, name: o.content}));
        console.log("check");
        console.log(data);


        // 配置图表选项
        const options = {
          series: [
            {
              name: '百分比',
              type: 'pie',
              radius: '50%',
              data: data,
            },
          ],
        };
        console.log(options)
        // 使用配置项显示图表
        this.chart.setOption(options,true);
      }
    },
    openDialog2(param) {
      this.columnTableVisible = true;
      this.$nextTick(() => {
        this.renderChart2(param);
      });
    },
    renderChart2(param) {
      this.chart = echarts.init(document.getElementById('main2'));

      // 将百分比数据转换为 echarts 饼图所需的格式
      let question, option
      const paperId = this.$route.params.paperId
      let optionArray = []; // 定义一个空数组，用来存储所有的选项
      if (this.res2 && this.res2.data.success) {
        const paperStatistic = this.res2.data.content
        for (question of paperStatistic.questionStatistics) {
          if (question.type === 3) {
            continue
          }
          const total = question.filledInNum
          for (option of question.optionStatistics) {
            option.percent = (Math.floor(option.selectedNum / total * 1000) / 10);
            if(option.questionId === param) {
              optionArray.push(option);
            }
          }
        }

        console.log("test1");
        console.log(option.content);
        console.log(option.percent);
        //var data = null;
        //data.name = option.content;
        //data.value = option.percent;
        let data = optionArray.map(o => o.percent); // 把对象数组转换成数值数组
        let names = optionArray.map(o => o.content); // 把对象数组转换成名称数组

        console.log("check");
        console.log(data);

        // 配置图表选项
        const options = {
          xAxis: { // 添加 x 轴属性
            type: 'category', // 类目轴
            data: names // x 轴数据
          },
          yAxis: { // 添加 y 轴属性
            type: 'value' // 数值轴
          },
          series: [
            {
              name: '百分比',
              type: 'bar', // 柱状图类型
              data: data, // 柱状图数据
              barWidth: '50%' // 柱子宽度
            },
          ],
        };

        // 使用配置项显示图表
        this.chart.setOption(options);
      }
    },
      handleDialogClose()
      {
        if (this.chart) {
          this.chart.dispose(); // 销毁图表实例
          this.chart = null;
        }
      }
    }
}
</script>

<style>
.questionTitle {
  margin-left: 5px;
  display: inline-block;
}
</style>
