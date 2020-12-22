<template>
  <h1>分类列表</h1>
  <div class="page_width">
    <a-table
      :columns="columns"
      :data-source="categories"
      :pagination="paginationProps"
    >
      <template v-slot:action="{ record }">
        <span>
          <a @click="EditCategory(record)">编辑</a>
          <span v-if="record.isDeleted">
            <a-divider type="vertical" />
            <a @click="DeleteCategory(record)">上架</a>
          </span>
          <span v-else>
            <a-divider type="vertical" />
            <a @click="DeleteCategory(record)">删除</a>
          </span>
        </span>
      </template>
    </a-table>
    <template>
      <div>
        <a-button type="primary" @click="showCategoryModal"> </a-button>
        <a-modal
          title="修改用户信息"
          v-model:visible="category_visible"
          :confirm-loading="confirmCategoryLoading"
          @ok="handleCategoryOk"
        >
          <p>
            <!--            <a-input v-model:value="C1Name" placeholder="一级分类名称"/>-->
            <!--            <br/><br/>-->
            <!--            <a-input v-model:value="C1Order" placeholder="一级分类排序"/>-->
            <!--            <br/><br/>-->
            <!--            <a-input v-model:value="C2Name" placeholder="二级分类名称"/>-->
            <!--            <br/><br/>-->
            <!--            <a-input v-model:value="C2Order" placeholder="二级分类排序"/>-->
            <!--            <br/><br/>-->
            <a-input v-model:value="c3Name" placeholder="三级分类名称" />
            <br /><br />
            <a-input v-model:value="c3Order" placeholder="三级分类排序" />
            <br /><br />
          </p>
        </a-modal>
      </div>
    </template>
  </div>
</template>

<script>
import { computed, onMounted, ref } from "vue";
import { useStore } from "vuex";

export default {
  name: "Category",
  setup() {
    const columns = ref([
      {
        title: "一级分类名称",
        dataIndex: "c1Name",
        key: "c1Name",
      },
      {
        title: "一级分类描述",
        dataIndex: "c1Desc",
        key: "c1Desc",
      },
      {
        title: "一级分类排序",
        dataIndex: "c1Order",
        key: "c1Order",
      },
      {
        title: "二级分类名称",
        dataIndex: "c2Name",
        key: "c2Name",
      },
      {
        title: "二级分类排序",
        dataIndex: "c2Order",
        key: "c2Order",
      },
      {
        title: "三级分类名称",
        dataIndex: "c3Name",
        key: "c3Name",
      },
      {
        title: "三级分类排序",
        dataIndex: "c3Order",
        key: "c3Order",
      },
      {
        title: "操作",
        key: "action",
        slots: { customRender: "action" },
      },
    ]);
    // let bannerId = ref("");
    // let url = ref("");
    // let redirectUrl = ref("");
    // let order = ref("");
    let confirmCategoryLoading = ref(false);
    let currentCategory = ref(1);
    let category_visible = ref(false);
    const page_size = ref(10);
    const categories = computed(() => store.state.category_list);
    const bannerTotal = computed(() => store.state.category_total);
    const bannerInfo = computed(() => store.state.category_info);

    // let C1CategoryID = ref("");
    // let C1Name = ref("");
    // let C1Desc = ref("");
    // let C1Order = ref(0);
    // let C2CategoryID = ref("");
    // let C2Name = ref("");
    // let C2Order = ref(0);
    // let C3CategoryID = ref("");
    // let C3Name = ref("");
    // let C3Order = ref(0);

    let c1CategoryId = ref("");
    let c1Name = ref("");
    let c1Desc = ref("");
    let c1Order = ref(0);
    let c2CategoryId = ref("");
    let c2Name = ref("");
    let c2Order = ref(0);
    let c3CategoryId = ref("");
    let c3Name = ref("");
    let c3Order = ref(0);

    const store = useStore();

    onMounted(() => {
      GetCategoryList(currentCategory.value, page_size.value);
    });

    function GetCategoryList(page, size) {
      let p = { Page: page, PageSize: size };
      store.dispatch("Get_Category_List", p);
    }

    const paginationProps = ref({
      pageSize: page_size.value,
      page: currentCategory.value,
      onChange: (page) => handleCategoryTableChange(page),
      total: bannerTotal,
    });

    function handleCategoryTableChange(idx) {
      currentCategory.value = idx;
      GetCategoryList(idx, page_size.value);
    }

    async function DeleteCategory(record) {
      let categoryId = record.c3CategoryId;
      await store.dispatch("Delete_Category", categoryId);
      GetCategoryList(currentCategory.value, page_size.value);
    }

    function showCategoryModal() {
      category_visible.value = true;
    }

    async function EditCategory(record) {
      console.log("EditProduct");
      console.log(record.value);
      // C1Name.value = record.C1Name;
      // C1Desc.value = record.url;
      // C1Order.value = record.C1Order;
      // C2Name.value = record.C2Name;
      // C2Order.value = record.C2Order;
      // C3CategoryID.value = record.C3CategoryID;
      // C3Name.value = record.C3Name;
      // C3Order.value = record.C3Order;
      c1Name.value = record.c1Name;
      c1Desc.value = record.url;
      c1Order.value = record.c1Order;
      c2Name.value = record.c2Name;
      c2Order.value = record.c2Order;
      c3CategoryId.value = record.c3CategoryId;
      c3Name.value = record.c3Name;
      c3Order.value = record.c3Order;
      showCategoryModal();
    }

    async function handleCategoryOk() {
      await UpdateCategory();
      confirmCategoryLoading.value = true;
      setTimeout(() => {
        category_visible.value = false;
        confirmCategoryLoading.value = false;
      }, 2000);
      await GetCategoryList(currentCategory.value, page_size.value);
    }

    async function UpdateCategory() {
      let param = {
        CategoryID: c3CategoryId.value,
        Name: c3Name.value,
        Order: parseInt(c3Order.value),
      };
      console.log(param);
      await store.dispatch("Update_Category", param);
    }

    return {
      categories,
      bannerInfo,
      columns,
      paginationProps,
      DeleteCategory,
      showCategoryModal,
      confirmCategoryLoading,
      handleCategoryOk,
      EditCategory,
      category_visible,
      // bannerId,
      // url,
      // redirectUrl,
      // order,
      // C1CategoryID,
      // C1Name,
      // C1Desc,
      // C1Order,
      // C2CategoryID,
      // C2Name,
      // C2Order,
      // C3CategoryID,
      // C3Name,
      // C3Order,
      c1CategoryId,
      c1Name,
      c1Desc,
      c1Order,
      c2CategoryId,
      c2Name,
      c2Order,
      c3CategoryId,
      c3Name,
      c3Order,
    };
  },
};
</script>

<style scoped>
</style>
