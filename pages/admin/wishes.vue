<template>
  <div>
    <a-button @click="showModal()" type="primary">
      Create
    </a-button>
    <a-table :columns="columns" :data-source="data">
      <span slot="media" slot-scope="text, record">
        <a-avatar
          v-if="record.media.length > 0"
          :size="64"
          :src="`https://socool-api.herokuapp.com${record.media[0].url}`"
      /></span>
      <span slot="action" slot-scope="text, record">
        <a
          v-if="record._id !== wishId"
          @click="onUpdate(record._id, record.type)"
          >{{ !record.type ? "Private" : "Public" }}</a
        >
        <a v-else-if="record._id === wishId">Loading...</a>
      </span>
    </a-table>

    <a-modal
      title="Create"
      :visible="visible"
      :confirm-loading="confirmLoading"
      @ok="onCreate"
      @cancel="handleCancel"
    >
      <a-form
        id="components-form-demo-normal-login"
        :form="form"
        class="login-form"
        @submit="onCreate"
      >
        <a-form-item label="Title">
          <a-input
            v-decorator="[
              'title',
              {
                rules: [{ required: true, message: 'Please input your title!' }]
              }
            ]"
            type="text"
            placeholder="Title"
          >
            <a-icon
              slot="prefix"
              type="lock"
              style="color: rgba(0, 0, 0, 0.25)"
            />
          </a-input>
        </a-form-item>
        <a-form-item label="Description">
          <a-input
            v-decorator="[
              'description',
              {
                rules: [
                  { required: true, message: 'Please input your description!' }
                ]
              }
            ]"
            type="text"
            placeholder="Description"
          >
            <a-icon
              slot="prefix"
              type="lock"
              style="color: rgba(0, 0, 0, 0.25)"
            />
          </a-input>
        </a-form-item>
        <a-form-item label="Type">
          <a-select
            @change="onChange"
            default-value="Private"
            style="width: 120px"
          >
            <a-select-option :value="1">
              Private
            </a-select-option>
            <a-select-option :value="0">
              Public
            </a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="Media">
          <upload ref="upload" />
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>
<script>
import { Upload } from "@/components/Upload.vue";
const columns = [
  {
    title: "Title",
    dataIndex: "title",
    key: "title"
  },
  {
    title: "Description",
    dataIndex: "description",
    key: "description"
  },
  {
    title: "Media",
    dataIndex: "media",
    key: "media",
    scopedSlots: { customRender: "media" }
  },
  {
    title: "Type",
    key: "type",
    dataIndex: "type"
  },
  {
    title: "Action",
    key: "action",
    scopedSlots: { customRender: "action" }
  }
];

const data = [];
import axios from "axios";
import Cookies from "js-cookie";
export default {
  layout: "admin-layout",
  components: [Upload],
  beforeCreate() {
    this.form = this.$form.createForm(this, { name: "create" });
  },

  data() {
    return {
      data,
      columns,
      ModalText: "Change password",
      visible: false,
      confirmLoading: false,
      type: false,
      wishId: null
    };
  },

  async fetch() {
    try {
      const res = await axios({
        method: "get",
        url: "https://socool-api.herokuapp.com/wishes",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${Cookies.get("token")}`
        }
      });

      this.data = res.data;
    } catch (error) {
      if (error.response.status === 401) this.$router.push("/");
    }
  },

  computed: {
    customType() {
      return this.type === 0 ? true : false;
    }
  },

  methods: {
    async onUpdate(id, type) {
      try {
        this.wishId = id;
        await axios({
          method: "put",
          url: `https://socool-api.herokuapp.com/wishes/${id}`,
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${Cookies.get("token")}`
          },
          data: {
            type: !type
          }
        });

        await this.$fetch();

        this.$notification.open({
          message: "Successful Update",
          description: "Successful Update",
          icon: <a-icon type="smile" style="color: #108ee9" />
        });

        this.wishId = null;
      } catch (error) {
        this.wishId = null;

        console.log(error);
      }
    },

    onCreate() {
      this.form.validateFields(async (err, values) => {
        if (!err) {
          this.confirmLoading = true;

          try {
            const media = await this.$refs.upload.handleUpload();
            const res = await axios({
              method: "post",
              url: "https://socool-api.herokuapp.com/wishes",
              headers: {
                "Content-Type": "application/json",
                Authorization: `Bearer ${Cookies.get("token")}`
              },
              data: {
                ...values,
                media,
                type: this.customType
              }
            });

            this.$notification.open({
              message: "Successful Create",
              description: "Successful Create",
              icon: <a-icon type="smile" style="color: #108ee9" />
            });

            await this.$fetch();
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
    },

    showModal() {
      this.form = this.$form.createForm(this, { name: "create" });
      this.visible = true;
    },

    onChange(value) {
      this.type = value;
    }
  }
};
</script>
