<template>
  <div id="app">
    <el-container>
      <el-header>
        <div class="site-title-bar">
          <span>
            <el-link class="site-title" href="/" :underline="false">
              每日知乎
            </el-link>
          </span>
          <el-date-picker
            v-model="datePicker"
            type="date"
            placeholder="选择收录日期"
            @change="pickdate"
            :picker-options="pickerOptions"
          ></el-date-picker>
        </div>
      </el-header>
      <el-main v-loading="isLoading" element-loading-text="Loading">
        日报日期:{{ newsDate }}
        <el-tooltip
          effect="dark"
          content="这个时间是日报日期,上面的是发布日期"
          placement="bottom"
          ><i class="el-icon-info"></i>
        </el-tooltip>
        <el-table
          class="main-table"
          @row-click="openNews"
          :data="stories"
          stripe
          ><el-table-column prop="hint" label="栏目"> </el-table-column>
          <el-table-column prop="title" label="标题" min-width="200px">
          </el-table-column>
        </el-table>
      </el-main>
      <el-footer
        >以上所有数据来自于<el-link
          type="primary"
          href="//zhihu.com"
          target="_blank"
          rel="noopener noreferrer"
          >知乎</el-link
        >,本站只负责数据整理<br />
        图标来源<el-link
          type="primary"
          href="//www.iconfont.cn/"
          target="_blank"
          rel="noopener noreferrer"
          >iconfont</el-link
        >,基于<el-link
          type="primary"
          href="//github.com/Vuejs/Vue"
          target="_blank"
          rel="noopener noreferrer"
          >Vue</el-link
        >&<el-link
          type="primary"
          href="//github.com/ElemeFE/element"
          target="_blank"
          rel="noopener noreferrer"
          >Element</el-link
        >构建<br />
        <el-link
          type="primary"
          href="//github.com/iezuni/zhihudaily-vue"
          target="_blank"
          rel="noopener noreferrer"
          >github</el-link
        ></el-footer
      >
    </el-container>
  </div>
</template>

<script>
import { decode } from "./base64";
import axios from "axios";
const apiPath = process.env.VUE_APP_apiUrl;
export default {
  data() {
    return {
      pickerOptions: {
        disabledDate(time) {
          return time.getTime() > Date.now() || 1368842400000 > time.getTime();
        },
        shortcuts: [
          {
            text: "今天",
            onClick(picker) {
              picker.$emit("pick", new Date());
            },
          },
          {
            text: "昨天",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24);
              picker.$emit("pick", date);
            },
          },
          {
            text: "一周前",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", date);
            },
          },
          {
            text: "52周前",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24 * 7 * 52);
              picker.$emit("pick", date);
            },
          },
        ],
      },
      datePicker: "",
      newsDate: "",
      isLoading: false,
      stories: [{ hint: "没有数据", title: "请在上面选择一个日期" }],
    };
  },
  methods: {
    openNews(newsItem) {
      let newsLink = document.createElement("a");
      if (newsItem.url) newsLink.href = newsItem.url;
      else return 0;
      newsLink.target = "_blank";
      newsLink.rel = "noopener noreferrer";
      newsLink.click();
    },
    pickdate(time) {
      this.getNews(time);
    },
    async getNews(time) {
      if (this.isLoading) {
        this.$message({
          showClose: true,
          message: "请等待上一次请求完成",
          type: "warning",
          duration: 1500,
        });
        return false;
      }
      let _date = this.formatTime(time);
      this.stories = [];
      this.isLoading = true;
      let data = await axios({
        method: "post",
        url: apiPath,
        data: "date=" + _date,
      });
      for (const story of data.data.data.stories) {
        this.stories.push({
          hint: decode(story.hint),
          title: decode(story.title),
          url: story.url || "",
        });
      }
      this.newsDate = data.data.data.date;
      this.isLoading = false;
      this.$message({
        showClose: true,
        message: "加载完成",
        type: "success",
        duration: 1500,
      });
      return 0;
    },
    formatTime(date) {
      const formatNumber = (a) => {
        let n = a.toString();
        return n[1] ? n : "0" + n;
      };
      const year = date.getFullYear();
      const month = date.getMonth() + 1;
      const day = date.getDate();
      return [year, month, day].map(formatNumber).join("");
    },
  },
  mounted: async function () {
    await this.getNews(new Date());
  },
};
</script>

<style>
body {
  background-color: #eaedf7;
  margin: 0;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  text-align: center;
}
.el-header {
  z-index: 10;
  background-color: #eaf4fc;
  box-shadow: #b5b5b5ad 0px 1px 20px 1px;
  text-align: center;
}
.el-footer {
  margin: 0 auto;
  color: #636363;
  font-size: 10px;
}
.main-table {
  width: 100%;
  margin: 20px auto;
  color: #333;
}
.site-title {
  font-size: 24px;
  float: left;
}
.site-title-bar {
  margin: 0 auto;
  max-width: 800px;
}
@media screen and (min-width: 800px) {
  .main-table {
    border: 1px solid #3e3e3e14;
    max-width: 800px !important;
  }
}
</style>
