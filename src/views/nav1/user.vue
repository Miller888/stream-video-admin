<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px">
      <el-form :inline="true" :model="filters">
        <!-- <el-form-item>
          <el-input
            v-model="filters.match_name"
            placeholder="赛事名称"
          ></el-input>
        </el-form-item> -->
        <div class="block">
          <span class="demonstration">选择日期范围</span>
          <el-date-picker
            v-model="value2"
            type="daterange"
            align="right"
            unlink-panels
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            :picker-options="pickerOptions"
          >
          </el-date-picker>
        </div>
        <el-form-item>
          <el-button type="primary" v-on:click="getLiveHistory">查询</el-button>
        </el-form-item>
      </el-form>
    </el-col>

    <!--列表-->
    <template>
      <el-table
        :data="matchList"
        highlight-current-row
        v-loading="loading"
        style="width: 100%"
      >
        <el-table-column type="index" width="60"></el-table-column>
        <el-table-column
          prop="match_name"
          label="赛事名称"
          width="120"
          sortable
        >
        </el-table-column>
        <el-table-column
          prop="channel_url"
          label="频道地址"
          width="280"
          sortable
        >
        </el-table-column>
        <el-table-column prop="play_url" label="播放地址" width="330" sortable>
        </el-table-column>
        <el-table-column prop="create_at" label="建立时间" width="220" sortable>
        </el-table-column>
      </el-table>
    </template>
  </section>
</template>
<script>
import { getUserList } from "../../api/api";
//import NProgress from 'nprogress'
export default {
  data() {
    return {
      pickerOptions: {
        shortcuts: [
          {
            text: "最近一周",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            },
          },
        ],
      },
      value1: "",
      value2: "",
      filters: {
        match_name: "",
      },
      matchList: [],
      loading: false,
    };
  },
  methods: {
    //性别显示转换
    formatSex: function (row, column) {
      return row.sex == 1 ? "男" : row.sex == 0 ? "女" : "未知";
    },
    //获取用户列表
    getUser: function () {
      let para = {
        name: this.filters.name,
      };
      this.loading = true;
      //NProgress.start();
      getUserList(para).then((res) => {
        this.users = res.data.users;
        this.loading = false;
        //NProgress.done();
      });
    },
    async getLiveHistory() {
      const res = await fetch(
        `${this.$api}/get_history_stream_info?start=${this.value1}&end=${this.value2}`
      );
      const resJson = await res.json();
      this.matchList = resJson;
    },
  },
  mounted() {
    // this.getUser();
  },
};
</script>

<style scoped>
</style>