<template>
  <div>
    <head-top></head-top>
    <el-container>
      <el-main>
        <el-row>
          <el-col class="searchInp" :span="24" width="100%">
            <div class="block" style="float:left">
              <span class="demonstration">请选择日期</span>

              <el-date-picker
                v-model="day"
                type="date"
                placeholder="选择日期"
                value-format="yyyy-MM-dd"
                @change="handelcheckday"
              ></el-date-picker>
              <el-button type="primary" size="small" @click="searchday()">查询</el-button>
              <!-- 自定义组件 获取时间 -->
              <get-time :time="date"></get-time>
            </div>

            <el-table :data="tableData" stripe style="width: 80%;margin:0 auto" border>
              <el-table-column prop="ip" label="ip" align="center"></el-table-column>
              <el-table-column prop="num" label="访问量" align="center"></el-table-column>
              <el-table-column prop="time" label="访问时间" align="center"></el-table-column>
            </el-table>
            <div class="blockpage">
              <el-pagination
                @current-change="handleCurrentChange"
                @size-change="handleSizeChange"
                background
                layout="total,prev,pager,next,sizes"
                :current-page="currentPage"
                :page-sizes="[10, 50, 100]"
                :page-size="10"
                :total="total"
              ></el-pagination>
            </div>
          </el-col>
        </el-row>
        <div id="main" style="width: 1200px;height:600px;"></div>
      </el-main>
    </el-container>
    <!-- 优酷视频播放 -->
    <!-- <iframe
      height="498"
      width="510"
      src="http://player.youku.com/embed/XNDMyMTk4NDA2NA=="
      frameborder="0"
      allowfullscreen
    ></iframe>-->
  </div>
</template>

<script>
import headTop from "@/components/header";
import { quillRedefine } from "vue-quill-editor-upload";
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
import { setTimeout } from "timers";
import { totalip, Dtotalip } from "@/api/new";
import { setCookie, getCookie } from "@/api/axios";
import echarts from "echarts";
// 导入自定义时间组件
import getTime from "@/components/toolsApi/getTime";

export default {
  inject: ["reload"], //注入刷新方法
  components: { headTop, quillRedefine, getTime },
  data() {
    return {
      day: "",
      tableData: [],
      pageSize: 10,
      title: "",
      total: null,
      currentPage: 1,
      date: "",
      saveDate: ""
    };
  },

  mounted() {
    this.getiplist();
  },

  methods: {
    getiplist() {
      let data = { pageSize: 10, currentPage: this.currentPage };
      totalip().then(res => {
        // const len = res.data.items.length;
        let nowDay = res.data.items[0].time;
        // this.day = nowDay;
        this.saveDate = nowDay;

        this.date = "今日";
        this.tableData = res.data.items;
        this.total = res.data.totalNum;
      });
    },

    /**
     * 点击查询
     */
    searchday() {
      this.date = this.day == this.saveDate ? "今日" : this.day;
      this.currentPage = 1;
      if (this.day == "") {
        return this.$message.error("请选择日期");
      }
      let data = { currentPage: 1, pageSize: this.pageSize, time: this.day };
      Dtotalip(data).then(res => {
        if (res.data == "") {
          return this.$message.error("暂无数据");
        }
        this.tableData = res.data.items;
        this.total = res.data.totalNum;
      });
    },
    handleCurrentChange(val) {
      // 分页
      this.currentPage = val;
      let data = { currentPage: val, pageSize: this.pageSize, time: this.day };

      // 请求分页数据
      Dtotalip(data).then(res => {
        this.tableData = res.data.items;
        this.total = res.data.totalNum;
      });
    },
    handleSizeChange(val) {
      this.pageSize = val;
      let data = {
        currentPage: this.currentPage,
        pageSize: this.pageSize,
        time: this.day
      };
      Dtotalip(data).then(res => {
        this.tableData = res.data.items;
        this.total = res.data.totalNum;
      });
    },

    /**
     * 日期选择期选择日期
     */
    handelcheckday(e) {}
  }
};
</script>
<style scoped>
.searchInp {
  line-height: 70px;
}
.demonstration {
  margin: 0 10px;
}
.block {
  margin-left: 150px;
}
.blockpage {
  padding: 10px 0;
  text-align: center;
}
</style>