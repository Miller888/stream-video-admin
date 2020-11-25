<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px">
      <el-form :inline="true" :model="filters">
		<el-form-item>
          <el-button type="primary" @click="handleAdd">新增</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" v-on:click="getPendingMatchList"
            >查询已新增的转播</el-button
          >
          <el-button type="primary" v-on:click="getActiveMatchList"
            >查询正在进行的转播</el-button
          >
        </el-form-item>
      </el-form>
    </el-col>
    <!--列表-->
    <el-table
      :data="matchList"
      highlight-current-row
      v-loading="listLoading"
      @selection-change="selsChange"
      style="width: 100%"
    >
      <el-table-column type="selection" width="55"> </el-table-column>
      <el-table-column type="index" width="60"> </el-table-column>
      <el-table-column prop="match_name" label="赛事名称" width="200" sortable>
      </el-table-column>
      <el-table-column prop="channel_url" label="频道地址" width="280" sortable>
      </el-table-column>
      <el-table-column prop="play_url" label="播放地址" width="330" sortable>
      </el-table-column>
      <el-table-column prop="create_at" label="建立时间" width="220" sortable>
      </el-table-column>
      <el-table-column label="操作" width="150">
        <template scope="scope">
          <el-button v-if="!isActiveList"
            type="primary"
            size="small"
            @click="handlePlay(scope.$index, scope.row)"
            >播放</el-button
          >
          <el-button v-if="isActiveList"
            type="danger"
            size="small"
            @click="handleDel(scope.$index, scope.row)"
            >停止</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <el-pagination
        layout="prev, pager, next"
        @current-change="handleCurrentChange"
        :page-size="20"
        :total="total"
        style="float: right"
      >
      </el-pagination>
    </el-col>
    <!--新增界面-->
    <el-dialog
      title="新增"
      v-model="addFormVisible"
      :close-on-click-modal="false"
    >
      <el-form
        :model="addForm"
        label-width="80px"
        :rules="addFormRules"
        ref="addForm"
      >
        <el-form-item label="赛事名称" prop="name">
          <el-input v-model="addForm.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="直播地址" prop="url">
          <el-input v-model="addForm.url" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="parseLiveUrl">解析</el-button>
        </el-form-item>
        <el-form-item label="画质" prop="quality">
          <el-radio-group v-model="addForm.radio">
            <el-radio
              class="radio"
              v-for="(item, index) in quality"
              :key="`${index}`"
              :label="index"
              >{{ item }}</el-radio
            >
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="addFormVisible = false">取消</el-button>
        <el-button
          type="primary"
          @click.native="addSubmit"
          :loading="addLoading"
          >提交</el-button
        >
      </div>
    </el-dialog>
  </section>
</template>

<script>
import util from "../../common/js/util";
//import NProgress from 'nprogress'
import {
  getUserListPage,
  removeUser,
  batchRemoveUser,
  editUser,
  addUser,
} from "../../api/api";

export default {
  data() {
    return {
      filters: {
        name: "",
	  },
	  isActiveList:false,
      matchList: [],
      users: [],
      total: 0,
      page: 1,
      listLoading: false,
      sels: [], //列表选中列
      editFormVisible: false, //编辑界面是否显示
      editLoading: false,
      editFormRules: {
        name: [{ required: true, message: "请输入赛事名称", trigger: "blur" }],
      },
      radio: 0,
      addFormVisible: false, //新增界面是否显示
      addLoading: false,
      addFormRules: {
        name: [{ required: true, message: "请输入赛事名称", trigger: "blur" }],
        url: [{ required: true, message: "请输入直播地址", trigger: "blur" }],
      },
      //新增界面数据
      addForm: {
        radio: 0,
        name: "",
        url: "",
      },
      quality: [],
    };
  },
  methods: {
    handleCurrentChange(val) {
      this.page = val;
      this.getUsers();
    },
    async getPendingMatchList() {
      this.listLoading = true;
      const res = await fetch(`${this.$api}/get_pending_tasks`);
      const resJson = await res.json();
      this.matchList = resJson;
      console.log(this.matchList);
	  this.listLoading = false;
	  this.isActiveList = false;
    },
    async getActiveMatchList() {
      this.listLoading = true;
      const res = await fetch(`${this.$api}/get_active_tasks`);
      const resJson = await res.json();
      this.matchList = resJson;
      console.log(this.matchList);
	  this.listLoading = false;
	  this.isActiveList = true;
    },
    async parseLiveUrl() {
      const url = this.addForm.url;
      const res = await fetch(`${this.$api}/parse?channel_url=${url}`);
      const text = await res.json();
      this.quality = text;
    },
    async playLive(params) {
      const { id, channel_url, md5_match_name, quality } = params;
      const res = await fetch(
        `${this.$api}/start?channel_url=${channel_url}&md5_match_name=${md5_match_name}&quality=${quality}&id=${id}`
      );
      const resJson = res.json();
      return resJson;
    },
    async stopLive(params) {
      const { task_id } = params;
      const res = await fetch(`${this.$api}/stop?id=${task_id}`);
      const resJson = res.json();
      return resJson.code;
    },
    //获取用户列表
    getUsers() {
      let para = {
        page: this.page,
        name: this.filters.name,
      };
      this.listLoading = true;
      //NProgress.start();
      getUserListPage(para).then((res) => {
        this.total = res.data.total;
        this.users = res.data.users;
        this.listLoading = false;
        //NProgress.done();
      });
    },
    //删除
    handleDel: function (index, row) {
      this.$confirm("确认停止该转播吗?", "提示", {
        type: "warning",
      })
        .then(() => {
          this.listLoading = true;
          this.stopLive(row);
          this.listLoading = false;
          this.$message({
            message: "停止成功",
            type: "success",
		  });
		  this.getActiveMatchList();
          //   this.listLoading = true;
          //   //NProgress.start();
          //   let para = { id: row.id };
          //   removeUser(para).then((res) => {
          //     this.listLoading = false;
          //     //NProgress.done();

          //     this.getUsers();
        })
        .catch(() => {});
    },
    handlePlay: function (index, row) {
      this.$confirm("确认开始播放该转播吗?", "提示", {
        type: "warning",
      })
        .then(() => {
          this.listLoading = true;
          //NProgress.start();
          this.playLive(row);
          this.listLoading = false;
          //NProgress.done();
          this.$message({
            message: "播放成功",
            type: "success",
          });
        })
        .catch(() => {});
    },
    //显示编辑界面
    handleEdit: function (index, row) {
      this.editFormVisible = true;
      this.editForm = Object.assign({}, row);
    },
    //显示新增界面
    handleAdd: function () {
      this.addFormVisible = true;
      this.addForm = {
        radio: 0,
        name: "",
        url: "",
        age: 0,
        birth: "",
        addr: "",
        position: 0,
      };
    },
    //新增
    addSubmit: function () {
      this.$refs.addForm.validate((valid) => {
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {}).then(async () => {
            this.addLoading = true;
            //NProgress.start();
            let para = Object.assign({}, this.addForm);
            this.addLoading = false;
            //NProgress.done();
            const url = encodeURI(para.url);
            const res = await fetch(
              `${this.$api}/create?channel_url=${
                para.url
              }&match_name=${para.name}&quality=${this.quality[para.radio]}`
            );
            const resJson = await res.json();
            if (resJson.code == "ok") {
              this.$message({
                message: "提交成功",
                type: "success",
              });
              this.$refs["addForm"].resetFields();
              this.quality = [];
              this.addFormVisible = false;
            } else {
              this.$message({
                message: `提交错误 错误代码: ${resJson.code}`,
                type: "warning",
              });
              this.$refs["addForm"].resetFields();
              this.quality = [];
              this.addFormVisible = false;
            }
          });
        }
      });
    },
    selsChange: function (sels) {
      this.sels = sels;
    },
    //批量删除
    batchRemove: function () {
      var ids = this.sels.map((item) => item.id).toString();
      this.$confirm("确认删除选中记录吗？", "提示", {
        type: "warning",
      })
        .then(() => {
          this.listLoading = true;
          //NProgress.start();
          let para = { ids: ids };
          batchRemoveUser(para).then((res) => {
            this.listLoading = false;
            //NProgress.done();
            this.$message({
              message: "删除成功",
              type: "success",
            });
            this.getUsers();
          });
        })
        .catch(() => {});
    },
  },
  mounted() {
    // this.getUsers();
  },
};
</script>

<style scoped>
</style>