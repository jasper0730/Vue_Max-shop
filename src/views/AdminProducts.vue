<template>
  <div class="pb-5">
    <Loading :active="isLoading"></Loading>
    <div class="text-end mt-3">
      <button
        type="button"
        class="btn btn-dark rounded"
        @click="openModal(true)"
      >
        建立新的產品
      </button>
    </div>

    <table class="table mt-4">
      <thead>
        <tr>
          <th width="10%">分類</th>
          <th width="20%">產品名稱</th>
          <th width="10%">原價</th>
          <th width="10%">售價</th>
          <th width="10%">是否啟用</th>
          <th width="10%">編輯</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in products" :key="item.id">
          <td>{{ item.category }}</td>
          <td>{{ item.title }}</td>
          <td>{{ $filters.currency(item.origin_price) }}</td>
          <td>{{ $filters.currency(item.price) }}</td>
          <td v-if="item.is_enabled" class="text-success">啟用</td>
          <td v-else>未啟用</td>
          <td>
            <button
              type="button"
              class="btn btn-outline-dark rounded btn-sm ms-1"
              @click="openModal(false, item)"
            >
              編輯
            </button>
            <!-- 點擊刪除後打開刪除的modal,將item傳入openDeleteModal函式,
            透過函式將item傳入tempProduct,再從內層modal元件透過props取得資料,
            將資料套用在modal上顯示
             -->
            <button
              type="button"
              class="btn btn-outline-dark rounded btn-sm ms-1"
              @click="openDeleteModal(item)"
            >
              刪除
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <!-- AdminProductModal -->
    <AdminProductModal
      ref="adminProductModal"
      @update-product="updateProduct"
      :product="tempProduct"
      :isNew="isNew"
    ></AdminProductModal>
    <!-- DeleteModal -->
    <!-- 將外層tempProduct傳入內層的item = :item="tempProduct"(props) -->
    <!-- 從內層要使用外層的函式deleteItem = @delete-item="deleteItem"(emit)  -->
    <DeleteModal
      ref="deleteModal"
      :item="tempProduct"
      @delete-item="deleteItem"
    ></DeleteModal>
    <!-- Pagination -->
    <!-- 將外層pagination傳入內層的pages = :pages="pagination"(props) -->
    <!-- 從內層要使用外層的函式getAdminProducts = @emit-pages="getAdminProducts" (emit)  -->
    <Pagination @emit-pages="getAdminProducts" :pages="pagination"></Pagination>
  </div>
</template>

<script>
import DeleteModal from "@/components/DeleteModal.vue";
import Pagination from "@/components/Pagination.vue";
import AdminProductModal from "@/components/AdminProductModal.vue";

export default {
  data() {
    return {
      products: [],
      isLoading: false,
      isNew: false,
      tempProduct: {},
      status: {
        fileUploading: false,
      },
      pagination: {},
      currentPage: 1,
      modal: {
        editModal: "",
        deleteModal: "",
      },
    };
  },
  components: {
    AdminProductModal,
    Pagination,
    DeleteModal,
  },
  methods: {
    getAdminProducts(page = 1) {
      this.isLoading = true;
      this.currentPage = page;
      const url = `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/admin/products?page=${page}`;
      this.$http
        .get(url)
        .then((res) => {
          this.isLoading = false;
          this.products = res.data.products;
          this.pagination = res.data.pagination;
        })
        .catch(() => {
          this.isLoading = false;
          this.$swal.fire({
            title: "失敗",
            text: "產品列表",
            icon: "error",
            timer: 3000,
          });
        });
    },
    openModal(isNew, item) {
      if (isNew) {
        this.tempProduct = {};
        this.isNew = true;
      } else {
        this.tempProduct = { ...item };
        this.isNew = false;
      }
      const productComponent = this.$refs.adminProductModal;
      productComponent.openModal();
    },
    updateProduct(item) {
      this.tempProduct = item; // 將item賦予到tempProduct
      let url = `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/admin/product`;
      this.isLoading = true;
      // 新增產品
      let Method = "post";
      let status = "新增產品";

      // 如果isNew不是true,則為編輯產品
      if (!this.isNew) {
        url = `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/admin/product/${this.tempProduct.id}`;
        Method = "put";
        status = "更新產品";
      }
      const productComponent = this.$refs.adminProductModal;
      this.$http[Method](url, { data: this.tempProduct })
        .then(() => {
          this.isLoading = false;
          this.$swal.fire({
            title: "成功",
            text: status,
            icon: "success",
            timer: 3000,
          });
          productComponent.hideModal(); // 關閉modal
          this.getAdminProducts(this.currentPage); // 重新取得產品資料
        })
        .catch(() => {
          this.isLoading = false;
          this.$swal.fire({
            title: "失敗",
            text: status,
            icon: "error",
            timer: 3000,
          });
        });
    },
    openDeleteModal(item) {
      this.tempProduct = { ...item }; // item賦予到tempProduct
      const deleteComponent = this.$refs.deleteModal; // dom
      deleteComponent.openModal(); // 打開modal
    },
    deleteItem() {
      this.isLoading = true;
      const url = `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/admin/product/${this.tempProduct.id}`;
      this.$http
        .delete(url)
        .then(() => {
          this.isLoading = false;
          this.$swal.fire({
            title: "成功",
            text: "刪除品項",
            icon: "success",
            timer: 3000,
          });
          const deleteComponent = this.$refs.deleteModal; // dom
          deleteComponent.hideModal(); //關閉modal
          this.getAdminProducts(this.currentPage); //重新取得當前頁面的產品資料
        })
        .catch(() => {
          this.isLoading = false;
          this.$swal.fire({
            title: "失敗",
            text: "刪除品項",
            icon: "error",
            timer: 3000,
          });
        });
    },
  },
  mounted() {
    this.getAdminProducts();
  },
};
</script>