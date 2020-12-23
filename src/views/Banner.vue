<template>
  <h1>Carousel圖檔編輯</h1>
  <div class="page_width">
    <div class="addBtn">
      <a-button type="primary" @click="AddBanner"> 添加Carousel</a-button>
    </div>
    <a-table
      :columns="columns"
      :data-source="banners"
      :pagination="paginationProps"
    >
      <template v-slot:action="{ record }">
        <span>
          <a @click="EditBanner(record)">編輯</a>
          <span v-if="record.isDeleted">
            <a-divider type="vertical" />
            <a @click="DeleteBanner(record)">上架</a>
          </span>
          <span v-else>
            <a-divider type="vertical" />
            <a @click="DeleteBanner(record)">刪除</a>
          </span>
        </span>
      </template>
    </a-table>
    <template>
      <div>
        <a-button type="primary" @click="showBannerModal"> </a-button>
        <a-modal
          title="添加Carousel資訊"
          v-model:visible="banner_visible"
          :confirm-loading="confirmBannerLoading"
          @ok="handleBannerOk"
        >
          <p>
            <a-input v-model:value="url" placeholder="請輸入Carousel地址" />
            <br /><br />
            <a-input
              v-model:value="redirectUrl"
              placeholder="請輸入Carousel跳轉地址"
            />
            <br /><br />
            <a-input v-model:value="order" placeholder="請輸入排序" />
            <br /><br />
          </p>
        </a-modal>
      </div>
    </template>
  </div>
</template>

<script>
import { ref, computed, onMounted } from "vue";
import { useStore } from "vuex";
import { message } from "ant-design-vue";

export default {
  name: "Banner",
  setup() {
    const columns = ref([
      {
        title: "封面地址",
        dataIndex: "url",
        key: "url",
        slots: { title: "customTitle", customRender: "productName" },
      },
      {
        title: "跳轉地址",
        dataIndex: "redirectUrl",
        key: "redirectUrl",
      },
      {
        title: "排序",
        dataIndex: "order",
        key: "order",
      },
      {
        title: "操作",
        key: "action",
        slots: { customRender: "action" },
      },
    ]);
    let bannerId = ref("");
    let url = ref("");
    let redirectUrl = ref("");
    let order = ref("");
    let createUser = ref("");
    let updateUser = ref("");
    let confirmBannerLoading = ref(false);
    let currentBanner = ref(1);
    let banner_visible = ref(false);
    let isEdit = ref(false);
    const page_size = ref(10);
    const banners = computed(() => store.state.banner_list);
    const bannerTotal = computed(() => store.state.banner_total);
    const bannerInfo = computed(() => store.state.banner_info);

    const store = useStore();

    onMounted(() => {
      GetBannerList(currentBanner.value, page_size.value);
    });

    function GetBannerList(page, size) {
      let p = { Page: page, PageSize: size };
      store.dispatch("Get_Banner_List", p);
    }

    const paginationProps = ref({
      pageSize: page_size.value,
      page: currentBanner.value,
      onChange: (page) => handleBannerTableChange(page),
      total: bannerTotal,
    });

    function handleBannerTableChange(idx) {
      currentBanner.value = idx;
      GetBannerList(idx, page_size.value);
    }

    async function DeleteBanner(record) {
      let bannerId = record.bannerId;
      // Unhandled error during execution of native event handler  可能是api格式沒接上
      await store.dispatch("Delete_Banner", bannerId);
      GetBannerList(currentBanner.value, page_size.value);
      message.info("操作成功");
    }

    function showBannerModal() {
      banner_visible.value = true;
    }

    async function AddBanner() {
      showBannerModal();
      // isEdit = false;
    }

    async function EditBanner(record) {
      isEdit.value = true;
      console.log("EditProduct");
      console.log(record.value);
      bannerId.value = record.bannerId;
      url.value = record.url;
      redirectUrl.value = record.redirectUrl;
      order.value = record.order;
      createUser.value = record.createUser;
      updateUser.value = record.updateUser;
      message.info("操作成功");
      showBannerModal();
    }

    async function handleBannerOk() {
      await UpdateBanner();
      confirmBannerLoading.value = true;
      setTimeout(() => {
        banner_visible.value = false;
        confirmBannerLoading.value = false;
        isEdit.value = false;
      }, 500);
      await GetBannerList(currentBanner.value, page_size.value);
      message.info("操作成功");
    }

    async function UpdateBanner() {
      let param = {
        bannerId: bannerId.value,
        url: url.value,
        redirectUrl: redirectUrl.value,
        order: parseInt(order.value),
      };
      console.log(param);
      if (isEdit.value) {
        await store.dispatch("Update_Banner", param);
      } else {
        await store.dispatch("ADD_Banner", param);
      }
    }

    return {
      banners,
      bannerInfo,
      columns,
      paginationProps,
      AddBanner,
      DeleteBanner,
      showBannerModal,
      confirmBannerLoading,
      handleBannerOk,
      EditBanner,
      banner_visible,
      bannerId,
      url,
      redirectUrl,
      order,
      isEdit,
      createUser,
      updateUser,
    };
  },
};
</script>

<style scoped>
.addBtn {
  padding-left: 35px;
  padding-bottom: 30px;
  text-align: left;
}

.page_width {
  width: 80%;
  margin: 0 auto;
}
</style>
