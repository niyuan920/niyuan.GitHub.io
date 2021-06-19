参考：https://github.com/imwyw/html-css-js/blob/master/SUMMARY.md

代码示例（登录页面）

    <template>
    <div id="back" >
      <el-form 
      class="login_concent" 
      ref="refLogin" 
      :model="loginForm"  
      label-width="80px"
      :rules="rules"
      >
        <el-form-item label="用户名:" prop="userid">
          <el-input v-model="loginForm.userid"></el-input>
        </el-form-item>
        <el-form-item label="密码：" prop="password">
          <el-input type="password" v-model="loginForm.password"></el-input>
        </el-form-item>

        <el-alert v-show="isError" title="用户名或密码错误" type="error"></el-alert>

        <el-form-item id="buttoned">
          <el-button type="primary" @click="loginHandler">登录</el-button>
          <el-link></el-link>
           <el-link></el-link>
           <el-link></el-link>
          <el-link type="primary" @click="registerHandler">注册！</el-link>
        </el-form-item>
      </el-form>
    </div>
    </template>

    <script>
    export default {
      data() {
        return {
          loginForm: {
            userid: "",
            password: ""
          },
          labelPosition: "top",
          // 表单验证规则
          rules: {
            // 规则必须与data属性对应，el-form-item的prop属性保持一致
            userid: [
              { required: true, message: "用户名不可为空", trigger: "blur" }
            ],
            password: [
              { required: true, message: "密码不可为空", trigger: "blur" },
              { min: 2, message: "密码太短", trigger: "blur" }
            ]
          },
          isError: false
        };
      },
      methods: {
        loginHandler() {
          let sdata = this.$data.loginForm;
               // 提交时需要验证规则
          this.$refs["refLogin"].validate(valid => {
            // 验证通过方可提交
            if (valid) {
              this.axios
                .get(`/Security/Login?uname=${sdata.userid}&pwd=${sdata.password}`)
                .then(res => {
                  // 验证通过
                  if (res.data.status) {
                    // 路由跳转
                    this.$router.push({ name: "Home" });
                    sessionStorage.setItem("Token",res.data.data.Token);
                    this.$cookies.set('UserId',JSON.stringify(res.data.data.userId));
                    this.$cookies.set('UserName',res.data.data.userName);   
                    this.$cookies.set('UserIds',res.data.data.userId);             
                  } else {
                    this.$message.error("用户名或密码错误");
                  }
                })
                .catch(err => {
                  console.error(err);
                });
            } else {
              this.$message.error("信息填写有误！");
            }
          });
        },
        registerHandler() {
          this.$router.push({ name: "Register" });
        }
      }
    };
    </script>

    <style>
    .login_concent {

      width: 400px;
      padding-left: 500px;
      padding-top: 250px;
    }

    #buttoned{
      padding-left: 80px;
    }


    #back{
      background-image:url("../images/111.jpg");
      background-size: auto;
      height: 2800px;

    }

    </style>
