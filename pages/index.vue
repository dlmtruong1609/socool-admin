<template>
  <a-row
    type="flex"
    justify="space-around"
    align="middle"
    style="height: 100vh"
  >
    <a-col :span="12">
      <a-alert
        v-if="errMessage"
        message="Error"
        :description="errMessage"
        type="error"
        show-icon
      />
      <h1>Login</h1>
      <a-form
        id="components-form-demo-normal-login"
        :form="form"
        class="login-form"
        @submit="handleSubmit"
      >
        <a-form-item>
          <a-input
            v-decorator="[
              'identifier',
              {
                rules: [
                  { required: true, message: 'Please input your username!' }
                ]
              }
            ]"
            placeholder="Username"
          >
            <a-icon
              slot="prefix"
              type="user"
              style="color: rgba(0, 0, 0, 0.25)"
            />
          </a-input>
        </a-form-item>
        <a-form-item>
          <a-input
            v-decorator="[
              'password',
              {
                rules: [
                  { required: true, message: 'Please input your Password!' }
                ]
              }
            ]"
            type="password"
            placeholder="Password"
          >
            <a-icon
              slot="prefix"
              type="lock"
              style="color: rgba(0, 0, 0, 0.25)"
            />
          </a-input>
        </a-form-item>
        <a-form-item>
          <a-button :loading="busy" type="primary" html-type="submit" class="login-form-button">
            Log in
          </a-button>
        </a-form-item>
      </a-form>
    </a-col>
  </a-row>
</template>

<script>
import axios from "axios";
import Cookies from "js-cookie";
export default {
  beforeCreate() {
    this.form = this.$form.createForm(this, { name: "normal_login" });
  },
  data() {
    return {
      errMessage: null,
      busy: false,
    };
  },

  fetch() {
    if (Cookies.get("token")) {
      this.$router.push("/admin/wishes");
    }
  },

  methods: {
    handleSubmit(e) {
      e.preventDefault();
      this.form.validateFields(async (err, values) => {
        if (!err) {
          this.busy = true
          try {
            const res = await axios({
              method: "post",
              url: "https://socool-api.herokuapp.com/auth/local",
              headers: {
                "Content-Type": "application/json"
              },
              data: values
            });
            Cookies.set("token", res.data.jwt);
            Cookies.set("username", values.identifier);
            this.$router.push("/admin/wishes");

            this.$notification.open({
              message: "Successful login",
              description: "Successful login",
              icon: <a-icon type="smile" style="color: #108ee9" />
            });

            this.busy = false
          } catch (error) {
            this.busy = false
            this.errMessage =
              error.response.data?.data[0]?.messages[0]?.message;
          }
        }
      });
    }
  }
};
</script>
<style>
#components-form-demo-normal-login .login-form {
  max-width: 300px;
}
#components-form-demo-normal-login .login-form-forgot {
  float: right;
}
#components-form-demo-normal-login .login-form-button {
  width: 100%;
}
</style>
