<template>
  <div>
    <van-popup v-model="areaShow" position="bottom">
      <van-picker
        show-toolbar
        title="企业纳税类型"
        :columns="columns"
        @cancel="onCancel"
        @confirm="onConfirm"
      />
    </van-popup>
  </div>
</template>

<script>
export default {
  data() {
    return {
      areaShow: false,
      columns: []
    };
  },
  methods: {
    onConfirm(e) {
      this.areaShow = false;
      this.$bus.emit("UPDATA_TAX_TYPE", e);
    },
    onCancel() {
      this.areaShow = false;
    }
  },
  created() {
    let _self = this;
    _self.$bus.off("OPEN_TAX_TYPE");
    _self.$bus.on("OPEN_TAX_TYPE", e => {
      _self.areaShow = true;
      _self.columns = e;
    });
  }
};
</script>
