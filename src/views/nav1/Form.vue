<template>
  <el-form
    ref="form"
    :model="form"
    label-width="80px"
    @submit.prevent="onSubmit"
    :rules="rules"
    style="margin:20px;width:60%;min-width:600px;"
  >
    <el-form-item label="文章标题" prop="title">
      <el-input v-model="form.title" required="true"></el-input>
    </el-form-item>
    <el-form-item label="作者" prop="author">
      <el-input v-model="form.author"></el-input>
    </el-form-item>
    <el-form-item label="文章正文" prop="content">
      <el-input type="textarea" v-model="form.content" :autosize="{ minRows: 5}"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="onSubmit">立即创建</el-button>
      <el-button @click.native.prevent>取消</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        title: "",
        author: "",
        content: ""
      },
      rules: {
        title: [{ required: true, message: "请输入", trigger: "blur" }],
        author: [{ required: true, message: "请输入", trigger: "blur" }],
        content: [{ required: true, message: "请输入", trigger: "blur" }]
      }
    };
  },
  methods: {
    onSubmit() {
      let para = {
        title: this.form.title,
        author: this.form.author,
        content: this.form.content
      };
      console.log(para);
      this.axios
        .post("http://localhost:8080/news/insert", {
          title: para.title,
          author: para.author,
          content: para.content
        })
        .then(res => {
          console.log(res);
          if (res.data.meta.code == 1000) {
            this.$message({
              message: "添加成功",
              type: "success"
            });
            this.$router.push({ path: "/table" });
          }
        });
    }
  }
};
</script>