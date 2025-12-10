<template>
  <div class="_topBar">
    <div class="_communautesSection">
      <div class="_communautesLabel">
        <DLabel :str="$t('all_communities')" />
        <button
          type="button"
          class="u-button u-button_icon u-button_transparent _addCommunityButton"
          @click="$router.push('/explore')"
          :title="$t('see_all_communities')"
        >
          <b-icon icon="three-dots-vertical" />
        </button>
      </div>
      <div class="_corpusItems">
        <button
          v-for="folder in all_folders"
          :key="folder.$path"
          type="button"
          class="u-button u-button_pill u-button_white _corpusItem"
          :class="{ 'is--active': isCommunityActive(folder.$path) }"
          @click="$emit('toggleCorpus', folder.$path)"
        >
          {{ folder.title || $t("untitled") }}
        </button>
      </div>
    </div>

    <div class="_searchBar">
      <SearchInput2
        :value="search_str"
        @input="$emit('update:search_str', $event)"
        :search_placeholder="$t('search_fields')"
      />
    </div>

    <div class="_topBarControls">
      <button
        type="button"
        class="u-button u-button_icon u-button_transparent _filterToggle"
        :class="{ 'is--active': show_filter_bar }"
        @click="$emit('update:show_filter_bar', !show_filter_bar)"
        :title="$t('toggle_filters')"
      >
        <b-icon icon="funnel" />
      </button>

      <div class="_displayOptions">
        <div class="_zoomSlider">
          <button
            type="button"
            class="u-button u-button_icon u-button_transparent"
            @click="
              $emit(
                'update:stack_preview_width',
                Math.max(50, stack_preview_width - 10)
              )
            "
          >
            <b-icon icon="dash" />
          </button>
          <input
            type="range"
            class="_inputRange"
            :value="stack_preview_width"
            min="50"
            max="250"
            step="5"
            @input="$emit('update:stack_preview_width', +$event.target.value)"
          />
          <button
            type="button"
            class="u-button u-button_icon u-button_transparent"
            @click="
              $emit(
                'update:stack_preview_width',
                Math.min(250, stack_preview_width + 10)
              )
            "
          >
            <b-icon icon="plus" />
          </button>
        </div>

        <div class="_viewModeButtons">
          <button
            class="u-button u-button_icon"
            type="button"
            :class="{ 'is--active': view_mode === 'list' }"
            @click="$emit('update:view_mode', 'list')"
            :title="$t('list_view')"
          >
            <b-icon icon="grid3x3" />
          </button>
          <button
            class="u-button u-button_icon"
            type="button"
            :class="{ 'is--active': view_mode === 'fav' }"
            @click="$emit('update:view_mode', 'fav')"
            :title="$t('favorites_view')"
          >
            <svg
              width="16"
              height="16"
              viewBox="0 0 16 16"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M7.38174 1.75501C7.59507 1.19234 8.40241 1.19234 8.61641 1.75501L9.99641 5.57767C10.0931 5.83101 10.3391 5.99967 10.6137 5.99967H14.0051C14.6317 5.99967 14.9051 6.77967 14.4124 7.16167L11.9991 9.33301C11.891 9.41611 11.812 9.53132 11.7734 9.66211C11.7348 9.7929 11.7387 9.93255 11.7844 10.061L12.6657 13.7963C12.8804 14.3963 12.1857 14.9117 11.6604 14.5423L8.38241 12.4623C8.27015 12.3834 8.13628 12.3411 7.99907 12.3411C7.86187 12.3411 7.728 12.3834 7.61574 12.4623L4.33774 14.5423C3.81307 14.9117 3.11774 14.3957 3.33241 13.7963L4.21374 10.061C4.25946 9.93255 4.26331 9.7929 4.22475 9.66211C4.18618 9.53132 4.10718 9.41611 3.99907 9.33301L1.58574 7.16167C1.09241 6.77967 1.36707 5.99967 1.99241 5.99967H5.38374C5.51727 6.00012 5.64778 5.96001 5.75802 5.88466C5.86825 5.8093 5.95301 5.70225 6.00107 5.57767L7.38107 1.75501H7.38174Z"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
          </button>
          <button
            class="u-button u-button_icon"
            type="button"
            :class="{ 'is--active': view_mode === 'map' }"
            @click="$emit('update:view_mode', 'map')"
            :title="$t('map_view')"
          >
            <b-icon icon="map-fill" />
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import SearchInput2 from "@/components/SearchInput2.vue";

export default {
  props: {
    all_folders: {
      type: Array,
      default: () => [],
    },
    active_folder_paths: {
      type: Array,
      default: () => [],
    },
    search_str: {
      type: String,
      default: "",
    },
    show_filter_bar: {
      type: Boolean,
      default: false,
    },
    stack_preview_width: {
      type: Number,
      default: 120,
    },
    view_mode: {
      type: String,
      default: "list",
    },
  },
  components: {
    SearchInput2,
  },
  i18n: {
    messages: {
      fr: {
        all_communities: "Communautés",
        untitled: "Sans titre",
        see_all_communities: "Voir toutes les communautés",
        search_fields:
          "Rechercher dans les champs titre et description des documents.",
        toggle_filters: "Afficher/masquer les filtres",
        list_view: "Vue liste",
        favorites_view: "Vue favoris",
        map_view: "Vue carte",
      },
      en: {
        all_communities: "Communities",
        untitled: "Untitled",
        see_all_communities: "See all communities",
        search_fields: "Search in titles or descriptions of documents.",
        toggle_filters: "Toggle filters",
        list_view: "List view",
        favorites_view: "Favorites view",
        map_view: "Map view",
      },
    },
  },
  computed: {
    isCommunityActive() {
      return (folder_path) => {
        return this.active_folder_paths.includes(folder_path);
      };
    },
  },
};
</script>
<style lang="scss" scoped>
._topBar {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 2);
  padding: calc(var(--spacing) / 2) calc(var(--spacing) * 2);
  // border-bottom: 1px solid var(--h-200);
  overflow-x: auto;
  overflow-y: hidden;

  > * {
    flex: 0 0 auto;

    &._communautesSection {
      flex: 1 1 100px;
    }
    &._searchBar {
      flex: 1 1 300px;
    }
    &._topBarControls {
      flex: 0 0 auto;
    }
  }
}

._topBarControls {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 2);
  flex: 0 0 auto;
  margin-left: auto;
}

._searchBar {
  flex: 0 0 auto;
  min-width: 300px;
  max-width: 500px;
}

._filterToggle {
  flex: 0 0 auto;

  &.is--active {
    background-color: var(--active-color);
    color: white;
  }
}

._displayOptions {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 2);
  flex: 0 0 auto;
}

._zoomSlider {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 4);
  flex: 0 0 auto;
}

._zoomSlider ._inputRange {
  width: 100px;
  margin: 0;
}

._viewModeButtons {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 4);
  flex: 0 0 auto;
}

._communautesSection {
  // display: flex;
  // flex-flow: row nowrap;
  // align-items: center;
  // gap: calc(var(--spacing) / 2);
  flex: 1 1 auto;
  min-width: 0;
}

._communautesLabel {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  justify-content: space-between;
  gap: calc(var(--spacing) / 2);
}

._corpusItems {
  display: flex;
  flex-flow: row nowrap;
  gap: calc(var(--spacing) / 2);
  overflow-x: auto;
  overflow-y: hidden;
  flex: 1 1 auto;
  min-width: 0;

  @include scrollbar(3px, 4px, 4px, transparent, var(--c-noir));
}

._corpusItem {
  flex: 0 0 auto;
  padding: calc(var(--spacing) / 8) calc(var(--spacing) / 2);
  font-size: 0.875rem;
  line-height: 1.2;
  white-space: nowrap;
  border-radius: 999px;
}

._toggleIcon {
  margin-left: calc(var(--spacing) / 4);
  font-size: 0.75em;
  opacity: 0.7;
  transition: opacity 0.2s ease;

  .u-button:hover & {
    opacity: 1;
  }
}

._addCommunityButton {
  flex: 0 0 auto;
  margin-left: calc(var(--spacing) / 2);
}
</style>
