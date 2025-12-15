<template>
  <div class="_filterBar">
    <div class="_filterPane">
      <div class="_topRow">
        <h2 class="_filterTitle">{{ $t("filters") }}</h2>
        <button
          type="button"
          class="u-buttonLink _closeBtn"
          @click="$emit('close')"
        >
          <b-icon icon="x" />
        </button>
      </div>

      <div class="_filterSection">
        <div class="_sectionTitle">{{ $t("sort_by") }}</div>
        <div class="_radioGroup">
          <label class="_radioLabel">
            <input
              type="radio"
              value="date_modified"
              :checked="sort_order === 'date_modified'"
              @change="$emit('update:sort_order', 'date_modified')"
            />
            <span class="_radioText">{{ $t("date_modified") }}</span>
          </label>
          <label class="_radioLabel">
            <input
              type="radio"
              value="date_created"
              :checked="sort_order === 'date_created'"
              @change="$emit('update:sort_order', 'date_created')"
            />
            <span class="_radioText">{{ $t("date_created") }}</span>
          </label>
        </div>
      </div>

      <div class="_filterSection">
        <div class="_sectionTitle">{{ $t("group_by_date") }}</div>
        <select
          class="_selectField"
          :value="group_mode"
          @change="$emit('update:group_mode', $event.target.value)"
        >
          <option
            v-for="group_option in group_options"
            :key="group_option.key"
            :value="group_option.key"
            v-text="group_option.label"
          />
        </select>
      </div>

      <div class="_filterSection">
        <div class="_sectionTitle">{{ $t("filter_by_author") }}</div>
        <select
          class="_selectField"
          :value="author_path_filter"
          @change="$emit('update:author_path_filter', $event.target.value)"
        >
          <option value="" v-text="$t('none')" />
          <option
            v-for="author in all_authors"
            :key="author.$path"
            :value="author.$path"
            v-text="author.name"
          />
        </select>
      </div>

      <div class="_filterSection">
        <div class="_tag">
          <DLabel :str="$t('filter_by_keyword')" />
          <input
            type="text"
            class="_searchField"
            :placeholder="$t('search')"
            v-model="keyword_search"
          />

          <div
            v-for="category in keywords_by_category"
            :key="category.type"
            class="_keywordCategory"
          >
            <div class="_categoryHeader">
              {{ category.type }}
            </div>
            <div class="_keywordCheckboxes">
              <label
                v-for="keyword in displayedKeywords(category)"
                :key="keyword.title"
                class="u-keywords _keywordCheckbox"
              >
                <input
                  type="checkbox"
                  :checked="isKeywordSelected(keyword.title)"
                  @change="toggleKeyword(keyword.title, $event.target.checked)"
                />
                <SingleKeyword
                  :keyword="keyword.title"
                  :show_category="false"
                  :count="keyword.count"
                  :cat_color="getCategoryColor(category.type)"
                />
              </label>
              <button
                v-if="shouldShowMoreButton(category)"
                type="button"
                class="u-buttonLink _showMoreButton"
                @click="toggleCategoryExpansion(category.type)"
              >
                <template v-if="!isCategoryExpanded(category.type)">
                  {{ $t("show_more") }} ({{
                    category.keywords.length - initial_keywords_limit
                  }})
                </template>
                <template v-else>
                  {{ $t("show_less") }}
                </template>
                <b-icon
                  :icon="
                    isCategoryExpanded(category.type)
                      ? 'chevron-up'
                      : 'chevron-down'
                  "
                />
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="_resetFilters" v-if="can_be_reset">
      <button
        type="button"
        class="u-button u-button_black u-button_pill"
        @click="resetFilters"
      >
        {{ $t("reset") }}
      </button>
    </div>
  </div>
</template>
<script>
import SingleKeyword from "@/components/SingleKeyword.vue";

export default {
  props: {
    shared_files: Array,
    group_mode: String,
    sort_order: String,
    author_path_filter: String,
    available_keywords: Array,
    keywords_filter: Array,
  },
  components: {
    SingleKeyword,
  },
  data() {
    return {
      all_authors: [],
      keyword_search: "",
      expanded_categories: {},
      initial_keywords_limit: 5,

      group_options: [
        {
          key: "day",
          label: this.$t("day"),
        },
        {
          key: "month",
          label: this.$t("month"),
        },
        {
          key: "year",
          label: this.$t("year"),
        },
      ],

      type_of_media_to_display: "all",
      types_of_medias: [
        {
          key: "all",
          label: this.$t("all_medias_types"),
        },
        {
          key: "image",
          label: this.$t("image"),
        },
        {
          key: "video",
          label: this.$t("video"),
        },
        {
          key: "audio",
          label: this.$t("audio"),
        },
        {
          key: "text",
          label: this.$t("text"),
        },
        {
          key: "pdf",
          label: this.$t("pdf"),
        },
        {
          key: "stl",
          label: this.$t("stl"),
        },
        {
          key: "other",
          label: this.$t("other"),
        },
      ],
    };
  },
  i18n: {
    messages: {
      fr: {
        show_more: "Afficher plus",
        show_less: "Afficher moins",
      },
      en: {
        show_more: "Show more",
        show_less: "Show less",
      },
    },
  },
  async created() {
    this.all_authors = await this.$api.getFolders({
      path: `authors`,
    });
  },
  async mounted() {},
  beforeDestroy() {},
  watch: {},
  computed: {
    keywords_by_category() {
      const categories = {};

      // available_keywords already contains only valid keywords (computed in parent)
      this.available_keywords.forEach((kw) => {
        const keyword_name = this.getKeywordName(kw.title);
        if (
          this.keyword_search &&
          !keyword_name
            .toLowerCase()
            .includes(this.keyword_search.toLowerCase())
        ) {
          return;
        }

        const category = this.getKeywordCategory(kw.title);
        const categoryKey = category || "OTHER";

        if (!categories[categoryKey]) {
          categories[categoryKey] = {
            type: categoryKey,
            keywords: [],
          };
        }

        categories[categoryKey].keywords.push(kw);
      });

      // Sort categories and keywords within each category
      return Object.keys(categories)
        .sort()
        .map((key) => ({
          type: key,
          keywords: categories[key].keywords.sort((a, b) => {
            if (b.count !== a.count) return b.count - a.count;
            return this.getKeywordName(a.title).localeCompare(
              this.getKeywordName(b.title)
            );
          }),
        }));
    },
    can_be_reset() {
      return (
        this.group_mode !== "year" ||
        this.sort_order !== "date_modified" ||
        this.author_path_filter !== "" ||
        this.keywords_filter.length > 0
      );
    },
  },
  methods: {
    resetFilters() {
      this.$emit("update:group_mode", "year");
      this.$emit("update:sort_order", "date_modified");
      this.$emit("update:author_path_filter", "");
      this.$emit("update:keywords_filter", []);
      this.keyword_search = "";
    },
    getKeywordCategory(keyword) {
      return keyword.includes("/") ? keyword.split("/")[0] : null;
    },
    getKeywordName(keyword) {
      return keyword.includes("/") ? keyword.split("/")[1] : keyword;
    },
    isKeywordSelected(keyword) {
      return this.keywords_filter.includes(keyword);
    },
    toggleKeyword(keyword, checked) {
      let _new_kw = this.keywords_filter.slice();
      if (checked) {
        // Add to inclusion list (show only items with this keyword)
        if (!_new_kw.includes(keyword)) {
          _new_kw.push(keyword);
        }
      } else {
        // Remove from inclusion list
        _new_kw = _new_kw.filter((kw) => kw !== keyword);
      }
      this.$emit("update:keywords_filter", _new_kw);
    },
    getCategoryColorStyle(categoryType) {
      if (!categoryType || !window.app_infos?.custom_suggested_categories) {
        return "";
      }
      const category = window.app_infos.custom_suggested_categories.find(
        (c) => c.title === categoryType
      );
      if (!category?.tag_color) return "";
      return `--cat-color: ${category.tag_color}`;
    },
    getCategoryColor(categoryType) {
      if (!categoryType || !window.app_infos?.custom_suggested_categories) {
        return undefined;
      }
      const category = window.app_infos.custom_suggested_categories.find(
        (c) => c.title === categoryType
      );
      return category?.tag_color;
    },
    displayedKeywords(category) {
      if (
        this.isCategoryExpanded(category.type) ||
        this.keyword_search.length > 0
      ) {
        return category.keywords;
      }
      return category.keywords.slice(0, this.initial_keywords_limit);
    },
    shouldShowMoreButton(category) {
      if (this.keyword_search.length > 0) return false;
      return category.keywords.length > this.initial_keywords_limit;
    },
    isCategoryExpanded(categoryType) {
      return this.expanded_categories[categoryType] === true;
    },
    toggleCategoryExpansion(categoryType) {
      this.$set(
        this.expanded_categories,
        categoryType,
        !this.expanded_categories[categoryType]
      );
    },
  },
};
</script>
<style lang="scss" scoped>
._filterBar {
  position: relative;
  z-index: 1;
  height: 100%;
  display: flex;
  flex-direction: column;

  @include scrollbar(3px, 4px, 4px, transparent, var(--c-noir));
}

._filterPane {
  flex: 1 1 auto;
  overflow-y: auto;

  padding: calc(var(--spacing) * 2);
  padding-bottom: calc(var(--spacing) * 4);
}

._topRow {
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
  align-items: center;
  margin-bottom: calc(var(--spacing) * 1);
}

._filterTitle {
  font-family: var(--font-title);
  font-weight: 600;
  margin: 0;
  color: var(--c-noir);
}

._closeBtn {
  font-size: 1.5rem;
  line-height: 1;
  color: var(--c-noir);
  padding: 0;

  &:hover {
    color: var(--c-rouge);
  }
}

._filterSection {
  margin-bottom: calc(var(--spacing) * 1.5);
}

._sectionTitle {
  font-family: var(--font-sans);
  font-weight: 600;
  font-size: 1rem;
  margin-bottom: calc(var(--spacing) / 2);
  color: var(--c-gris-fonce);
}

._radioGroup {
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 4);
}

._radioLabel {
  display: flex;
  align-items: center;
  gap: calc(var(--spacing) / 2);
  cursor: pointer;
  font-family: var(--font-sans);
  font-size: 0.95rem;

  input[type="radio"] {
    appearance: none;
    width: 16px;
    height: 16px;
    border: 1px solid var(--c-gris);
    border-radius: 50%;
    outline: none;
    margin: 0;
    position: relative;

    &:checked {
      border-color: var(--c-noir);

      &::after {
        content: "";
        position: absolute;
        top: 3px;
        left: 3px;
        width: 8px;
        height: 8px;
        border-radius: 50%;
        background-color: var(--c-noir);
      }
    }
  }

  ._radioText {
    color: var(--c-noir);
  }

  &:hover ._radioText {
    text-decoration: underline;
  }
}

._selectField {
  width: 100%;
  border: 1px solid var(--c-gris);
  border-radius: 4px;
  padding: calc(var(--spacing) / 4) calc(var(--spacing) / 2);
  color: var(--c-noir);
  background-color: transparent;
  font-family: var(--font-sans);
}

._searchField {
  width: 100%;
  border: 1px solid var(--c-gris);
  border-radius: 4px;
  padding: calc(var(--spacing) / 4) calc(var(--spacing) / 2);
  margin-top: calc(var(--spacing) / 2);
  margin-bottom: calc(var(--spacing) * 1);
  color: var(--c-noir);
  background-color: transparent;
  font-family: var(--font-sans);

  &:focus {
    outline: none;
    border-color: var(--c-noir);
  }
}

._keywordCategory {
  margin-bottom: calc(var(--spacing) * 1);
}

._categoryHeader {
  position: sticky;
  font-weight: 600;
}

._keywordCheckboxes {
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 4);
}

._showMoreButton {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 4);
  margin-top: calc(var(--spacing) / 4);
  font-size: var(--sl-font-size-small);
  color: var(--h-600);
  cursor: pointer;

  &:hover {
    color: var(--h-700);
  }
}

._keywordCheckbox {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 2);
  cursor: pointer;

  input[type="checkbox"] {
    cursor: pointer;
    flex: 0 0 auto;
  }
}

._resetFilters {
  flex: 0 0 auto;
  text-align: center;
  padding: calc(var(--spacing) / 2);
  margin-top: auto;
  background-color: white;
  border-top: 1px solid var(--c-gris);
}
</style>
