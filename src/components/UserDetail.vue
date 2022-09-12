<template>
  <div>
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>用户详情</span>
        <el-button
          style="float: right; padding: 3px 0"
          type="text"
          @click="goBack"
          >返回</el-button
        >
      </div>
      <div class="text item">
        <p>姓名：{{ userInfo.name }}</p>
        <p>年龄：{{ userInfo.age }}</p>
        <p>头衔：{{ userInfo.position }}</p>
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: "UserDetail",
  data() {
    return {
      userInfo: [],
    };
  },
  props: ["id"],
  created() {
    this.getUserInfo();
  },
  methods: {
    async getUserInfo() {
      const { data: res } = await this.$http.get("/api/users/" + this.id);
      if (res.status !== 0) return this.$message.error("获取用户详情失败");
      this.userInfo = res.data;
    },
    goBack() {
      this.$router.go(-1);
    },
  },
};
</script>

<style lang="less" scoped></style>
