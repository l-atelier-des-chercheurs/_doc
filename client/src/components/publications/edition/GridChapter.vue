<template>
  <div class="_gridChapter">
    <div class="_gridItems">
      <GridItem
        v-for="area in sorted_grid_areas"
        :key="area.id"
        :area="area"
        :area_text_meta="getAreaTextMeta(area)"
        :publication="publication"
        @createText="createText"
      />
    </div>
  </div>
</template>

<script>
import GridItem from "./GridItem.vue";

export default {
  props: {
    chapter: Object,
    publication: Object,
  },
  components: {
    GridItem,
  },
  data() {
    return {};
  },
  computed: {
    sorted_grid_areas() {
      return this.chapter.grid_areas?.sort((a, b) => a.id.localeCompare(b.id));
    },
  },
  methods: {
    async createText(areaId) {
      const filename = areaId + "_text.md";
      const { meta_filename } = await this.$api.uploadText({
        path: this.publication.$path,
        filename,
        content: "",
        additional_meta: {
          content_type: "markdown",
          grid_area_id: areaId,
        },
      });
    },
    getAreaTextMeta(area) {
      return this.publication.$files.find((f) => f.grid_area_id === area.id);
    },
  },
};
</script>

<style lang="scss" scoped>
._gridChapter {
  padding-bottom: calc(var(--spacing) * 1);
}

._gridItems {
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) * 1);
}
</style>
