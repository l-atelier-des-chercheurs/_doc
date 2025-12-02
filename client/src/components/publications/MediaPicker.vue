<template>
  <div class="_mediaPicker">
    <PickExistingMediastackModal
      :modal_title="$t('pick_medias')"
      :select_mode="select_mode"
      @close="$emit('close')"
      @mediasSelected="mediasSelected"
    />

  </BaseModal2>
</template>
<script>
import PickExistingMediastackModal from "@/components/PickExistingMediastackModal.vue";

export default {
  props: {
    publication_path: String,
    select_mode: {
      type: String,
      default: "multiple",
    },
    pick_from_types: [String, Array],
  },
  components: {
    PickExistingMediastackModal,
  },
  data() {
    return {
      selected_source: null,
    };
  },
  async created() {},
  beforeDestroy() {},
  watch: {},
  computed: {},
  methods: {
    mediasSelected(medias) {
      this.$emit("pickMedias", medias);
      this.$emit("close");
    },
  },
};
</script>
<style lang="scss" scoped>
._sourceSelector {
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
  text-align: center;
  gap: calc(var(--spacing) / 2);
  // padding: 1rem 0;

  > * {
    flex: 1 1 0;
  }
}

._sourceBtn {
  // display: flex;
  // align-items: center;
  // gap: 1rem;
  // padding: 1.5rem;
  // text-align: left;
  // min-height: 80px;

  ._sourceBtnContent {
    // display: flex;
    // flex-direction: column;
    // gap: 0.25rem;

    // strong {
    //   font-size: 1.1rem;
    // }

    // small {
    //   opacity: 0.8;
    //   font-size: 0.9rem;
    // }
  }
}
</style>
