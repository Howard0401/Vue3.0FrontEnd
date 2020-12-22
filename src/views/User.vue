<template>
  <h1>用户列表</h1>
  <div class="page_width">
    <a-table
      :columns="columns"
      :data-source="users"
      :pagination="paginationProps"
    >
      <template v-slot:action="{ record }">
        <span>
          <a @click="EditUser(record)">编辑</a>
          <span v-if="record.user_deleted">
            <a-divider type="vertical" />
            <a @click="DeleteUser(record)">复活</a>
          </span>
          <span v-else>
            <a-divider type="vertical" />
            <a @click="DeleteUser(record)">删除</a>
          </span>
        </span>
      </template>
    </a-table>
    <template>
      <div>
        <a-button type="primary" @click="showModal"> </a-button>
        <a-modal
          title="修改用户信息"
          v-model:visible="visible"
          :confirm-loading="confirmLoading"
          @ok="handleOk"
        >
          <p>
            <a-input
              v-model:value="userInfo.nickName"
              placeholder="请输入昵称"
            /><br /><br />
            <a-input
              v-model:value="userInfo.mobile"
              placeholder="请输入手机号码"
            /><br /><br />
            <a-input
              v-model:value="userInfo.address"
              placeholder="请输入收获地址"
            /><br /><br />
          </p>
        </a-modal>
      </div>
    </template>
  </div>
</template>

<script>
import { reactive, toRefs, computed, ref, onMounted } from "vue";
import { useStore } from "vuex";
import { message } from "ant-design-vue";

export default {
  name: "User",
  setup() {
    const columns = ref([
      {
        title: "用户昵称",
        dataIndex: "nickName",
        key: "nickName",
      },
      {
        title: "手机号",
        dataIndex: "mobile",
        key: "mobile",
      },
      {
        title: "收货地址",
        dataIndex: "address",
        key: "address",
      },
      {
        title: "Action",
        key: "action",
        slots: { customRender: "action" },
      },
    ]);
    let current = ref(1);
    const page_size = ref(10);
    let visible = ref(false);
    let confirmLoading = ref(false);
    let nickName = ref("");
    let mobile = ref("");
    let address = ref("");
    let userId = ref("");
    let isEdit = ref(false);
    const store = useStore();
    const users = computed(() => store.state.user_list);
    const userInfo = computed(() => store.state.user_info);
    const userTotal = computed(() => store.state.user_total);
    const user_deleted = computed(() => store.state.user_deleted);
    const state = reactive({
      userList: [
        { name: "aaa", age: 18 },
        { name: "bbb", age: 23 },
      ],
      total: 888,
    });

    const paginationProps = ref({
      pageSize: page_size.value,
      page: current.value,
      onChange: (page) => handleUserTableChange(page),
      total: userTotal,
    });

    onMounted(() => {
      console.log("onMounted");
      console.log(page_size.value);
      GetUserList(1, page_size.value);
    });

    function handleUserTableChange(idx) {
      console.log(idx);
      current.value = idx;
      GetUserList(idx, page_size.value);
    }

    function showModal() {
      visible.value = true;
    }

    async function handleOk() {
      // let nickNameVal = nickName.value;
      // let mobileVal = mobile.value;
      // let addressVal = address.value;
      // console.log("prepare UpdateUser");
      // UpdateUser(nickNameVal, mobileVal, addressVal);
      // console.log(record);
      isEdit = true;
      await UpdateUser();
      confirmLoading.value = true;
      setTimeout(() => {
        visible.value = false;
        confirmLoading.value = false;
        isEdit = false;
      }, 2000);
      await GetUserList(current.value, page_size.value);
    }

    function GetUserList(page, size) {
      const p = { Page: page, PageSize: size };
      store.dispatch("Get_User_List", p);
    }

    async function EditUser(record) {
      // let userId = record.userId;
      isEdit = true;
      console.log("EditUser");

      await store.dispatch("Get_User_Info", record.userId);
      userId.value = record.userId;
      // user_deleted.value = record.user_deleted;
      nickName.value = record.nickName;
      mobile.value = record.mobile;
      address.value = record.address;
      showModal();
    }

    function DeleteUser(record) {
      console.log(user_deleted);
      store.dispatch("Delete_User", record.userId).then(() => {
        console.log(record.userId);
        if (record.userId) {
          message.info("操作成功");
          record.user_deleted = !record.user_deleted;
          // GetUserList(current, page_size.value);
        }
      });
    }

    // async function UpdateUser(nickName, mobile, address) {
    async function UpdateUser() {
      let p = userInfo.value;
      let payload = {
        UserId: p.userId,
        NickName: p.nickName,
        Mobile: p.mobile,
        Address: p.address,
      };
      console.log(payload);
      // console.log(userId);
      // let res = await store.dispatch("Update_User", payload);
      await store.dispatch("Update_User", payload);
      GetUserList(current.value, page_size.value);
    }

    return {
      ...toRefs(state),
      users,
      userInfo,
      columns,
      visible,
      nickName,
      mobile,
      address,
      confirmLoading,
      paginationProps,
      handleUserTableChange,
      current,
      EditUser,
      DeleteUser,
      UpdateUser,
      showModal,
      handleOk,
      userId,
      isEdit,
    };
  },
};
</script>

<style scoped>
.page_width {
  width: 80%;
  margin: 0 auto;
}
</style>
