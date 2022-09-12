<template>
  <div>
    <!-- 添加按钮 -->
    <el-button type="primary" @click="dialogVisible = true"
      >添加新用户</el-button
    >
    <!-- 用户的表格 -->
    <el-table :data="userList" stripe border>
      <!-- 这是索引列的固定写法 -->
      <el-table-column type="index" label="#"></el-table-column>
      <!-- 渲染用户名称 -->
      <el-table-column label="姓名" prop="name"></el-table-column>
      <el-table-column label="年龄" prop="age"></el-table-column>
      <el-table-column label="头衔" prop="position"></el-table-column>
      <el-table-column label="创建时间">
        <template v-slot:default="scope">
          {{ scope.row.addtime | dataFormat }}
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="{ row }">
          <router-link :to="'/users/' + row.id">详情</router-link>&nbsp;
          <a href="#" @click.prevent="onRemove(row.id)">删除</a>
        </template>
      </el-table-column>
    </el-table>
    <!-- 添加用户的对话框 -->
    <el-dialog
      title="添加新用户"
      :visible.sync="dialogVisible"
      width="50%"
      @close="onDialogClosed"
    >
      <!-- 添加用户的表单 -->
      <el-form
        :model="form"
        label-width="80px"
        :rules="fromRules"
        ref="myaddForm"
      >
        <!-- 采集用户的姓名 -->
        <el-form-item label="用户姓名" prop="name">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="用户年龄" prop="age">
          <el-input v-model.number="form.age"></el-input>
        </el-form-item>
        <el-form-item label="用户头衔" prop="position">
          <el-input v-model="form.position"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="onAddNewUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "UserList",
  data() {
    // 声明校验年龄的函数
    let checkAge = (rule, value, callback) => {
      if (!Number.isInteger(value)) {
        return callback(new Error("请填写整数"));
      }
      if (value > 100 || value < 1) {
        return callback(new Error("年龄必须在1到100之间"));
      }
      callback();
    };
    return {
      userList: [],
      // 控制添加对话框的显示与隐藏
      dialogVisible: false,
      // 要采集的用户的信息对象
      form: {
        name: "",
        age: "",
        position: "",
      },
      // 表单的验证规则对象
      fromRules: {
        name: [
          { required: true, message: "姓名是必填项", trigger: "blur" },
          { min: 2, max: 7, message: "长度在2到7个字符", trigger: "blur" },
        ],
        age: [
          { required: true, message: "密码是必填项", trigger: "blur" },
          { validator: checkAge, trigger: "blur" },
        ],
        position: [
          { required: true, message: "头衔是必填项", trigger: "blur" },
          { min: 1, max: 10, message: "长度在1到10个字符", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    // 请求用户列表的数据
    async getUserList() {
      const { data: res } = await this.$http.get("/api/users");
      if (res.status !== 0) return console.log("用户列表数据获取失败");
      this.userList = res.data;
    },
    // 监听对话框关闭的事件
    onDialogClosed() {
      // 拿到From组件的引用，调用resetFields函数，即可重置表单
      this.$refs.myaddForm.resetFields();
    },
    // 添加用户事件
    onAddNewUser() {
      this.$refs.myaddForm.validate(async (valid) => {
        if (!valid) return;
        // 执行添加的操作
        const { data: res } = await this.$http.post("/api/users", this.form);
        if (res.status !== 0) return this.$message.error("添加用户失败");
        this.$message.success("添加用户成功");
        this.dialogVisible = false;
        this.getUserList();
      });
    },
    // 点击了删除的链接
    async onRemove(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该用户, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") return this.$message.info("取消了删除");
      const { data: res } = await this.$http.delete("/api/users/" + id);
      if (res.status !== 0) return this.$message.error("删除失败");
      this.$message.success("删除成功");
      this.getUserList();
    },
  },
  created() {
    this.getUserList();
  },
};
</script>

<style lang="less" scoped>
.el-table {
  margin-top: 15px;
}
</style>
