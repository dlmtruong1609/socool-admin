<template>
  <a-layout id="components-layout-demo-custom-trigger">
    <a-layout-sider v-model="collapsed" :trigger="null" collapsible>
      <div class="logo" />
      <a-menu theme="dark" mode="inline" :default-selected-keys="['1']">
        <a-menu-item key="1">
          <a-icon type="unordered-list" />
          <span>Wishes</span>
        </a-menu-item>
      </a-menu>
    </a-layout-sider>
    <a-layout>
      <a-layout-header style="background: #fff; padding: 0; display: flex;">
        <a-icon
          class="trigger"
          :type="collapsed ? 'menu-unfold' : 'menu-fold'"
          @click="() => (collapsed = !collapsed)"
        />

        <a-dropdown
          style="margin-left: auto; display: block; margin-right: 1rem;"
        >
          <a class="ant-dropdown-link" @click="e => e.preventDefault()">
            Setting <a-icon type="down" />
          </a>
          <a-menu slot="overlay">
            <a-menu-item>
              <a @click="showModal">Đổi mật khẩu</a>
            </a-menu-item>
            <a-menu-item>
              <a href="javascript:;" @click="onLogout">Đăng xuất</a>
            </a-menu-item>
          </a-menu>
        </a-dropdown>
      </a-layout-header>
      <a-layout-content
        :style="{
          margin: '24px 16px',
          padding: '24px',
          background: '#fff',
          minHeight: '280px'
        }"
      >
        <nuxt />
      </a-layout-content>
    </a-layout>
    <a-modal
      title="Change Password"
      :visible="visible"
      :confirm-loading="confirmLoading"
      @ok="handleOk"
      @cancel="handleCancel"
    >
      <a-form
        id="components-form-demo-normal-login"
        :form="form"
        class="login-form"
        @submit="handleOk"
      >
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
          <a-input
            v-decorator="[
              'newPassword',
              {
                rules: [
                  { required: true, message: 'Please input your New Password!' }
                ]
              }
            ]"
            type="password"
            placeholder="New Password"
          >
            <a-icon
              slot="prefix"
              type="lock"
              style="color: rgba(0, 0, 0, 0.25)"
            />
          </a-input>
        </a-form-item>
      </a-form>
    </a-modal>
  </a-layout>
</template>
<script>
import Cookies from "js-cookie";
import axios from "axios";
export default {
  beforeCreate() {
    this.form = this.$form.createForm(this, { name: "change_password" });
  },
  data() {
    return {
      collapsed: false,
      ModalText: "Change password",
      visible: false,
      confirmLoading: false
    };
  },

  methods: {
    onLogout() {
      Cookies.remove("token");
      this.$router.push("/");
    },

    showModal() {
      this.visible = true;
    },

    handleOk(e) {
      this.ModalText = "Loading...";
      this.confirmLoading = true;
      this.form.validateFields(async (err, values) => {
        if (!err) {
          try {
            const res = await axios({
              method: "put",
              url: "https://socool-api.herokuapp.com/password",
              headers: {
                "Content-Type": "application/json",
                Authorization: `Bearer ${Cookies.get("token")}`
              },
              data: {
                ...values,
                identifier: Cookies.get("username")
              }
            });

            this.$notification.open({
              message: "Successful change password",
              description: "Successful change password",
              icon: <a-icon type="smile" style="color: #108ee9" />
            });

            this.visible = false;
            this.confirmLoading = false;
          } catch (error) {
            this.confirmLoading = false;
            this.$notification.open({
              message: "Error",
              description: error.response.data?.data[0]?.messages[0]?.message,
              icon: <a-icon type="smile" style="color: red" />
            });
          }
        }
      });
    },

    handleCancel(e) {
      this.visible = false;
    }
  }
};
</script>
<style>
#components-layout-demo-custom-trigger .trigger {
  font-size: 18px;
  line-height: 64px;
  padding: 0 24px;
  cursor: pointer;
  transition: color 0.3s;
}

#components-layout-demo-custom-trigger .trigger:hover {
  color: #1890ff;
}

#components-layout-demo-custom-trigger .logo {
  height: 32px;
  background: rgba(255, 255, 255, 0.2);
  margin: 16px;
}
</style>
