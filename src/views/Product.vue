<template>
  <h1>商品列表</h1>
  <a-table
    :columns="columns"
    :data-source="products"
    :pagination="paginationProps"
  >
    <template v-slot:productName="{ text }">
      {{ text }}
    </template>
    <template v-slot:originalPrice="{ text }">
      {{ text }}
    </template>
    <template v-slot:sellingPrice="{ text }">
      {{ text }}
    </template>
    <template v-slot:stockNum="{ text }">
      {{ text }}
    </template>
    <template v-slot:action="{ record }">
      <span>
        <a @click="EditProduct(record)">編輯</a>
        <a-divider type="vertical" />
        <span v-if="record.isDeleted">
          <a @click="DeleteProduct(record)">上架</a>
        </span>
        <span v-else>
          <a @click="DeleteProduct(record)">刪除</a>
        </span>
      </span>
    </template>
    <template #expandedRowRender="{ record }">
      <p class="hiddenContent">
        <span class="title">商品ID:</span>{{ record.productId }}
      </p>
      <p class="hiddenContent">
        <span class="title">商品簡介:</span
        >{{ record.productIntro == "" ? "无" : record.productIntro }}
      </p>
      <p class="hiddenContent">
        <span class="title">商品分類:</span>{{ record.categoryId }}
      </p>
      <p class="hiddenContent">
        <span class="title">商品封面:</span>{{ record.productCoverImg }}
      </p>
      <p class="hiddenContent">
        <span class="title">商品輪播圖:</span>{{ record.productBanner }}
      </p>
      <p class="hiddenContent">
        <span class="title">商品Tag:</span>{{ record.tag }}
      </p>
      <p class="hiddenContent">
        <span class="title">商品銷售狀態:</span
        >{{ record.sellStatus == 0 ? "銷售中" : "停止銷售" }}
      </p>
      <p class="hiddenContent">
        <span class="title">商品詳細內容:</span
        >{{ record.productDetailContent }}
      </p>
    </template>
  </a-table>
  <template>
    <div>
      <a-button type="primary" @click="showProductModal"> </a-button>
      <a-modal
        title="修改商品資訊"
        v-model:visible="product_visible"
        :confirm-loading="confirmProductLoading"
        @ok="handleProductOk"
      >
        <p>
          這邊的productInfo 後面要接後端API傳來的JSON 因此格式要對映好!
          <br />
          <br />
          <a-input
            v-model:value="productInfo.productName"
            placeholder="請輸入商品名稱"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.originalPrice"
            placeholder="請輸入原始價格"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.sellingPrice"
            placeholder="請輸入銷售價格"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.stockNum"
            placeholder="請輸入庫存"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.productIntro"
            placeholder="請輸入商品簡介"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.productBanner"
            placeholder="請輸入商品封面"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.productCoverImg"
            placeholder="請輸入商品輪播圖"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.tag"
            placeholder="請輸入商品Tag"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.sellStatus"
            placeholder="請輸入商品銷售狀態"
          />
          <br /><br />
          <a-input
            v-model:value="productInfo.productDetailContent"
            placeholder="請輸入商品銷售詳細內容"
          />
          <br /><br />
        </p>
      </a-modal>
    </div>
  </template>
</template>

<script>
import { ref, onMounted, computed } from "vue";
import { useStore } from "vuex";

export default {
  name: "Product",
  setup() {
    const columns = ref([
      {
        title: "商品名稱",
        dataIndex: "productName",
        key: "productName",
        slots: { title: "customTitle", customRender: "productName" },
      },
      {
        title: "原始價格",
        dataIndex: "originalPrice",
        key: "originalPrice",
      },
      {
        title: "銷售價格",
        dataIndex: "sellingPrice",
        key: "sellingPrice",
      },
      {
        title: "庫存",
        key: "stockNum",
        dataIndex: "stockNum",
        slots: { customRender: "stockNum" },
      },
      {
        title: "操作",
        key: "action",
        slots: { customRender: "action" },
      },
    ]);
    let productId = ref("");
    let productName = ref("");
    let sellingStatus = ref("");
    let productBanner = ref("");
    let productCoverImg = ref("");
    let productDetailContent = ref("");
    let productIntro = ref("");
    // categoryId = record.
    // CategoryName: p.categoryName,
    let tag = ref("");
    // let categoryId = ref("");
    let originalPrice = ref("");
    let sellingPrice = ref("");

    let stockNum = ref("");
    let confirmProductLoading = ref(false);
    let currentProduct = ref(1);
    let product_visible = ref(false);
    const page_size = ref(10);
    const products = computed(() => store.state.product_list);
    const productTotal = computed(() => store.state.product_total);
    const productInfo = computed(() => store.state.product_info);

    const store = useStore();

    onMounted(() => {
      GetProductList(currentProduct.value, page_size.value);
    });

    function GetProductList(page, size) {
      let p = { Page: page, PageSize: size };
      store.dispatch("Get_Product_List", p);
    }

    const paginationProps = ref({
      pageSize: page_size.value,
      page: currentProduct.value,
      onChange: (page) => handleProductTableChange(page),
      total: productTotal,
    });

    function handleProductTableChange(idx) {
      currentProduct.value = idx;
      GetProductList(idx, page_size.value);
    }

    async function DeleteProduct(record) {
      console.log(`productID是${productId}`);
      let productId = record.productId;
      await store.dispatch("Delete_Product", productId);
      GetProductList(currentProduct.value, page_size.value);
    }

    function showProductModal() {
      product_visible.value = true;
    }

    async function EditProduct(record) {
      console.log("EditProduct");
      console.log(`record=${record.productBanner}`);
      productId = record.productId;
      // 利用ID再去尋找一次API取得新資料，
      await store.dispatch("Get_Product_Info", productId);
      showProductModal();
    }

    async function handleProductOk() {
      await UpdateProduct();
      confirmProductLoading.value = true;
      setTimeout(() => {
        product_visible.value = false;
        confirmProductLoading.value = false;
      }, 2000);
      GetProductList(currentProduct.value, page_size.value);
    }

    async function UpdateProduct() {
      let p = productInfo.value;
      //取不同的名字，以便確認能放進去model中
      let param = {
        ProductId: p.productId,
        sellingStatus: p.sellingStatus,
        productBanner: p.imgCarousel,
        productCoverImg: p.imgCover,
        ProductName: p.productName,
        productDetailContent: p.sellingDetail,
        productIntro: p.introduction,
        CategoryId: p.categoryId,
        CategoryName: p.categoryName,
        OriginalPrice: parseInt(p.originalPrice),
        SellingPrice: parseInt(p.sellingPrice),
        StockNum: parseInt(p.stockNum),
        tag: p.Tag,
      };
      console.log(p);
      await store.dispatch("Update_Product", param);
    }

    return {
      products,
      productInfo,
      columns,
      paginationProps,
      DeleteProduct,
      showProductModal,
      confirmProductLoading,
      handleProductOk,
      EditProduct,
      product_visible,
      // categoryId,
      productId,
      productName,
      originalPrice,
      sellingPrice,
      stockNum,
      sellingStatus,
      productBanner,
      productCoverImg,
      productDetailContent,
      productIntro,
      // categoryId = record.
      // CategoryName: p.categoryName,
      tag,
    };
  },
};
</script>

<style scoped>
.page_width {
  width: 80%;
  margin: 0 auto;
}
.title {
  padding-right: 20px;
}

hiddenContent {
  margin-left: 0;
  margin-bottom: 5px;
}
</style>
