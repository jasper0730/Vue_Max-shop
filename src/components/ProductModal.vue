<template>
  <div
    class="modal fade"
    id="productModal"
    tabindex="-1"
    aria-labelledby="productModalLabel"
    aria-hidden="true"
    ref="modal"
  >
    <div class="modal-dialog modal-xl">
      <div class="modal-content">
        <div class="modal-header bg-dark text-white">
          <h5 class="modal-title" id="productModalLabel">
            {{ modalData.title }}
          </h5>
          <button
            type="button"
            class="btn-close btn-close-white"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <div class="row">
            <div class="col-sm-6">
              <img
                class="img-fluid object-fit"
                :src="modalData.imageUrl"
                :alt="modalData.title"
              />
            </div>
            <div class="col-sm-6">
              <h5>
                <span class="badge mb-1 bg-secondary rounded-pill fs-7">{{
                  modalData.category
                }}</span>
              </h5>
              <h4 class="fs-6">商品描述:</h4>
              <p class="fs-7">{{ modalData.description }}</p>
              <h4 class="fs-6 mt-3">商品內容:</h4>
              <p class="fs-7">{{ modalData.content }}</p>
              <!-- 若沒有優惠價或是原價與優惠價同價時顯示 -->
              <div
                v-if="
                  !modalData.price || modalData.origin_price === modalData.price
                "
              >
                <p class="h5">{{ modalData.origin_price }}</p>
              </div>
              <div v-else>
                <del class="h6 text-secondary"
                  >售價：{{ modalData.origin_price }}</del
                >
                <div class="h5 fw-bold text-danger">
                  優惠價：{{ modalData.price }}
                </div>
              </div>
              <div class="input-group mb-3">
                <input
                  type="number"
                  class="form-control"
                  v-model.number="qty"
                  min="1"
                />
                <!-- 若input綁定的qty為false或是等於小於0怎無法按按鈕 -->
                <button
                  :disabled="qty <= 0 || !qty"
                  class="btn btn-outline-dark"
                  type="button"
                  @click="$emit('add-to-cart', modalData.id, qty)"
                >
                  加入購物車
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import modalMixin from "@/mixins/modalMixin";
export default {
  props: {
    modalData: {
      type: Object,
    },
  },
  data() {
    return {
      modal: "",
      qty: 1,
    };
  },
  mixins:[modalMixin],
};
</script>