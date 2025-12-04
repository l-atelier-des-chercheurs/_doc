<template>
  <div class="_chapterLayout">
    <fieldset
      v-if="
        ['text', 'gallery', 'grid'].includes(chapter.section_type) &&
        view_mode === 'book'
      "
      class="u-spacingBottom _layout"
    >
      <legend>{{ $t("layout") }}</legend>
      <div class="_optionsRow">
        <div class="_colCount" v-if="chapter.section_type === 'text'">
          <DLabel :str="$t('column_count')" />
          <div class="">
            <SelectField2
              :field_name="'column_count'"
              :value="chapter.column_count || 1"
              :path="chapter.$path"
              size="small"
              :hide_validation="true"
              :can_edit="true"
              :options="[
                { key: 1, text: '1' },
                { key: 2, text: '2' },
                { key: 3, text: '3' },
              ]"
            />
          </div>
        </div>
        <div class="_selects--starts_on_page">
          <DLabel :str="$t('starts_on_page')" />
          <SelectField2
            :field_name="'section_starts_on_page'"
            :value="chapter.section_starts_on_page || ''"
            :path="chapter.$path"
            size="small"
            :hide_validation="true"
            :can_edit="true"
            :options="starts_on_page_options"
          />
        </div>
        <NumberInput
          :label="$t('column_count')"
          :value="chapter.column_count || 6"
          :size="'small'"
          :min="1"
          :max="12"
          @save="updateChapterMeta({ column_count: $event })"
        />
        <NumberInput
          :label="$t('row_count')"
          :value="chapter.row_count || 6"
          :size="'small'"
          :min="1"
          :max="12"
          @save="updateChapterMeta({ row_count: $event })"
        />
      </div>

      <template v-if="chapter.section_type === 'grid'">
        <GridAreas :chapter="chapter" @deleteArea="deleteArea" />
      </template>
    </fieldset>
  </div>
</template>

<script>
import GridAreas from "@/components/publications/edition/GridAreas.vue";

export default {
  props: {
    chapter: Object,
    publication: Object,
    view_mode: String,
  },
  components: {
    GridAreas,
  },
  data() {
    return {};
  },
  computed: {
    starts_on_page_options() {
      if (this.chapter.section_type === "gallery")
        return [
          {
            key: "page",
            text: this.$t("next_page"),
          },
          {
            key: "left",
            text: this.$t("next_left_page"),
          },
          {
            key: "right",
            text: this.$t("next_right_page"),
          },
        ];
      else
        return [
          {
            key: "",
            text: this.$t("in_flow"),
          },
          {
            key: "page",
            text: this.$t("next_page"),
          },
          {
            key: "left",
            text: this.$t("next_left_page"),
          },
          {
            key: "right",
            text: this.$t("next_right_page"),
          },
        ];
    },
  },
  methods: {
    updateChapterMeta(new_meta) {
      this.$api.updateMeta({
        path: this.chapter.$path,
        new_meta,
      });
    },

    async deleteArea(areaId) {
      // Find and delete associated text file if it exists
      const text_meta = this.publication.$files.find(
        (f) => f.grid_area_id === areaId
      );

      if (text_meta) {
        try {
          await this.$api.deleteItem({
            path: text_meta.$path,
          });
        } catch (error) {
          console.error("Error deleting text block:", error);
        }
      }

      // Remove area from grid_areas
      const grid_areas = this.chapter.grid_areas.filter(
        (area) => area.id !== areaId
      );
      this.updateChapterMeta({ grid_areas });
    },
  },
};
</script>

<style lang="scss" scoped>
._chapterLayout {
}

._colCount {
  max-width: 20ch;
}

._optionsRow {
  display: flex;
  flex-flow: row nowrap;
  align-items: flex-start;
  gap: calc(var(--spacing) * 1);
  margin-bottom: calc(var(--spacing) * 1);
}

._selects--starts_on_page {
  width: 30ch;
  flex: 0 0 auto;
  position: relative;
  z-index: 2;
}

._gridConfiguration {
  margin-top: calc(var(--spacing) * 1);
}
</style>
