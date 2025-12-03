<template>
  <div class="_sharedFolder">
    <div class="_topBar">
      <div class="_communautesSection">
        <DLabel :str="$t('all_communities')" class="_communautesLabel" />
        <div class="_corpusItems">
          <div
            v-for="folder in all_folders"
            :key="folder.$path"
            class="_corpusItem"
            :class="{ 'is--active': isCommunityActive(folder.$path) }"
          >
            <button
              type="button"
              class="u-button u-button_pill u-button_white"
              :class="{ 'is--active': isCommunityActive(folder.$path) }"
              @click="toggleCorpus(folder.$path)"
            >
              {{ folder.title || $t("untitled") }}
              <!-- <b-icon
                :icon="isCommunityActive(folder.$path) ? 'x' : 'plus'"
                class="_toggleIcon"
              /> -->
            </button>
          </div>
        </div>
        <button
          type="button"
          class="u-button u-button_icon u-button_transparent _addCommunityButton"
          @click="$router.push('/explore')"
          :title="$t('see_all_communities')"
        >
          <b-icon icon="list" />
        </button>
      </div>
    </div>

    <transition name="slideup" mode="out-in">
      <StackDisplay
        v-if="opened_stack"
        class="_stackModal"
        :key="opened_stack.$path"
        :stack_path="opened_stack.$path"
        :context="'archive'"
        :can_be_added_to_fav="can_be_added_to_fav"
        :can_be_selected="select_mode"
        :is_favorite="isFavorite(opened_stack.$path)"
        :read_only="read_only"
        @toggleFav="toggleFav(opened_stack.$path)"
        @prevMedia="navMedia(-1)"
        @nextMedia="navMedia(+1)"
        @selectStack="$emit('selectStack', opened_stack)"
        @selectMedias="$emit('selectMedias', $event)"
        @close="closeStack"
      />
    </transition>

    <transition name="fade_fast" mode="out-in">
      <div class="_loader" v-if="is_loading_folder">
        <LoaderSpinner />
      </div>
      <TwoColumnLayout
        v-else
        :show-sidebar.sync="show_filter_bar"
        class="_sharedFolder--content"
      >
        <template #sidebar>
          <FilterBar
            :group_mode.sync="group_mode"
            :sort_order.sync="sort_order"
            :search_str.sync="search_str"
            :filetype_filter.sync="filetype_filter"
            :author_path_filter.sync="author_path_filter"
            :available_keywords="valid_keywords"
            :keywords_filter.sync="keywords_filter"
            :fav_filter.sync="fav_filter"
            :view_mode.sync="view_mode"
            :stack_preview_width.sync="stack_preview_width"
          >
          </FilterBar>
        </template>

        <template #content>
          <transition name="pagechange" mode="out-in">
            <transition-group
              tag="div"
              name="projectsList"
              class="_stacksList"
              appear
              :key="sort_order + '-' + group_mode"
            >
              <div
                v-if="grouped_stacks.length === 0"
                class="u-instructions _noContent"
                :key="'nocontent'"
              >
                {{ $t("no_content") }}
              </div>
              <template v-else>
                <template v-if="view_mode === 'list'">
                  <div
                    class="_dayFileSection"
                    v-for="{ label, files: stacks } in grouped_stacks"
                    :key="label"
                  >
                    <div class="_label">
                      {{ label }}
                    </div>
                    <transition-group
                      tag="div"
                      class="_itemGrid"
                      :class="{
                        'is--compact': display_mode === 'compact',
                      }"
                      name="listComplete"
                      :style="{
                        '--stack_preview_width': `${stack_preview_width}px`,
                      }"
                      appear
                    >
                      <StackPreview
                        v-for="stack in stacks"
                        :key="stack.$path"
                        :stack="stack"
                        :display="display_mode"
                        :is_selected="stack.$path === last_selected_stack_path"
                        :can_be_added_to_fav="can_be_added_to_fav"
                        :is_favorite="isFavorite(stack.$path)"
                        @toggleFav="toggleFav(stack.$path)"
                        @openStack="openStack"
                      />
                    </transition-group>
                  </div>
                </template>
                <div
                  v-else-if="view_mode === 'map'"
                  key="mediaMap"
                  class="_mediamapContainer"
                >
                  <MediaMap
                    :medias="filtered_stacks"
                    @toggleMediaFocus="toggleMediaFocus"
                  />
                </div>
              </template>
            </transition-group>
          </transition>
        </template>
      </TwoColumnLayout>
    </transition>
  </div>
</template>
<script>
import FilterBar from "@/components/archive/FilterBar.vue";
import StackPreview from "@/components/archive/StackPreview.vue";
import StackDisplay from "@/components/StackDisplay.vue";
import CorpusMenu from "@/components/archive/CorpusMenu.vue";
import TwoColumnLayout from "@/adc-core/ui/TwoColumnLayout.vue";

export default {
  props: {
    shared_folder_paths: {
      type: Array,
      default: () => [],
    },
    select_mode: {
      type: [Boolean, String],
      default: false,
    },
    read_only: Boolean,
  },
  components: {
    FilterBar,
    StackPreview,
    StackDisplay,
    CorpusMenu,
    TwoColumnLayout,
    MediaMap: () => import("@/adc-core/ui/MediaMap.vue"),
  },
  data() {
    return {
      folder: undefined,
      all_folders: [],
      all_stacks: [],

      show_corpus_menu: false,

      is_loading_folder: true,
      show_admin_settings: false,

      last_selected_stack_path: undefined,
      view_mode: "list",

      sort_order: localStorage.getItem("archive.sort_order") || "date_modified",

      search_str: "",
      filetype_filter: "all",
      author_path_filter: "",
      keywords_filter: [],
      group_mode: "year",

      fav_filter: false,

      selected_medias_paths: [],

      stack_preview_width:
        parseInt(localStorage.getItem("archive.stack_preview_width")) || 120,

      show_filter_bar:
        localStorage.getItem("archive.show_filter_bar") === "true",

      joined_rooms: new Set(), // Track which rooms we've joined
    };
  },
  i18n: {
    messages: {
      fr: {
        corpus_title: "Titre du corpus",
      },
      en: {
        corpus_title: "Corpus title",
      },
    },
  },
  async created() {},
  async mounted() {
    await this.checkExistingFolder();

    const primaryPath = this.active_folder_paths[0];
    if (primaryPath) {
      localStorage.setItem("last_opened_folder_path", primaryPath);
      this.folder = await this.$api.getFolder({
        path: primaryPath,
      });
    }

    // Load all folders for the add community modal
    this.all_folders = await this.$api.getFolders({ path: "folders" });

    // Load stacks from all active communities
    await this.loadStacksFromCommunities();

    // Join socket rooms for all communities (stay connected)
    this.active_folder_paths.forEach((path) => {
      this.joinRoom(path);
      this.joinRoom(path + "/stacks");
    });

    this.is_loading_folder = false;
  },
  beforeDestroy() {
    // Leave all socket rooms
    this.active_folder_paths.forEach((path) => {
      this.$api.leave({ room: path });
      this.$api.leave({ room: path + "/stacks" });
    });
  },
  watch: {
    opened_stack() {
      if (this.opened_stack) {
        this.last_selected_stack_path = this.opened_stack.$path;
      }
    },
    active_folder_paths: {
      async handler(newPaths, oldPaths) {
        if (JSON.stringify(newPaths) !== JSON.stringify(oldPaths)) {
          // Paths changed, reload stacks
          if (newPaths.length > 0) {
            // Join new rooms (stay connected, don't leave old ones)
            newPaths.forEach((path) => {
              this.joinRoom(path);
              this.joinRoom(path + "/stacks");
            });
            // Load new stacks
            await this.loadStacksFromCommunities();
            // Update primary folder
            if (newPaths[0]) {
              this.folder = await this.$api.getFolder({
                path: newPaths[0],
              });
            }
          }
        }
      },
      immediate: false,
    },
    sort_order() {
      localStorage.setItem("archive.sort_order", this.sort_order);
    },
    show_filter_bar() {
      localStorage.setItem("archive.show_filter_bar", this.show_filter_bar);
    },
    stack_preview_width() {
      localStorage.setItem(
        "archive.stack_preview_width",
        this.stack_preview_width.toString()
      );
    },
  },
  computed: {
    active_folder_paths() {
      return this.shared_folder_paths || [];
    },
    display_mode() {
      return this.stack_preview_width < 120 ? "compact" : "";
    },
    can_edit() {
      return this.canLoggedinEditFolder({ folder: this.folder });
    },
    stack_shared_folder_paths() {
      return this.active_folder_paths.map((path) => path + "/stacks");
    },
    can_be_added_to_fav() {
      return (
        this.connected_as &&
        this.connected_as?.$path !== undefined &&
        !this.read_only
      );
    },
    sorted_stacks() {
      return this.all_stacks
        .slice()
        .sort(
          (a, b) => +new Date(b.$date_modified) - +new Date(a.$date_modified)
        );
    },
    stacks_without_keyword_filter() {
      // Stacks filtered by all filters except keyword filter
      // Used to compute which keywords would result in 0 results
      return this.sorted_stacks.filter((f) => {
        if (this.fav_filter) if (!this.isFavorite(f.$path)) return false;

        if (this.author_path_filter)
          if (!f.$authors?.includes(this.author_path_filter)) return false;

        if (this.search_str) {
          if (
            (f.title &&
              f.title.toLowerCase().includes(this.search_str.toLowerCase())) ||
            (f.description &&
              f.description
                .toLowerCase()
                .includes(this.search_str.toLowerCase()))
          )
            return true;
          else return false;
        }

        return true;
      });
    },
    filtered_stacks() {
      return this.sorted_stacks.filter((f) => {
        if (this.fav_filter) if (!this.isFavorite(f.$path)) return false;

        if (this.author_path_filter)
          if (!f.$authors?.includes(this.author_path_filter)) return false;

        if (this.keywords_filter.length > 0) {
          // Inclusion logic: show only stacks that have ALL selected keywords (AND logic)
          if (!f.keywords || !Array.isArray(f.keywords)) return false;
          if (!this.keywords_filter.every((kwf) => f.keywords.includes(kwf)))
            return false;
        }
        // if (this.filetype_filter !== "all")
        //   if (!this.fileOrStackContainsType(f, this.filetype_filter))
        //     return false;

        if (this.search_str) {
          if (
            (f.title &&
              f.title.toLowerCase().includes(this.search_str.toLowerCase())) ||
            (f.description &&
              f.description
                .toLowerCase()
                .includes(this.search_str.toLowerCase()))
          )
            return true;
          else return false;
        }

        return true;
      });
    },
    grouped_stacks() {
      let order_props = ["$date_modified"];
      if (this.sort_order === "date_created")
        order_props = [
          "date_created_corrected",
          "$date_created",
          "$date_uploaded",
        ];
      else if (this.sort_order === "date_modified")
        order_props = ["$date_modified"];
      return this.groupFilesBy(
        this.filtered_stacks,
        order_props,
        this.group_mode
      );
    },
    timeline_stacks() {
      let order_props = [
        "date_created_corrected",
        "$date_created",
        "$date_uploaded",
      ];
      return this.groupFilesBy(this.filtered_stacks, order_props, "day");
    },
    available_keywords() {
      // Use sorted_stacks instead of filtered_stacks so all keywords remain visible
      // even when some are excluded from the view
      const all_kw = this.sorted_stacks.reduce((acc, f) => {
        if (f.keywords && Array.isArray(f.keywords)) {
          f.keywords.map((kw) => {
            const item = acc.find((_kw) => _kw.title === kw);
            if (item) item.count += 1;
            else acc.push({ title: kw, count: 1 });
          });
        }
        return acc;
      }, []);
      return all_kw.sort((a, b) => {
        return b.count - a.count;
      });
    },
    valid_keywords() {
      // Filter out keywords that would result in 0 results when combined with current selection
      return this.available_keywords.filter((kw) => {
        // If keyword is already selected, always show it
        if (this.keywords_filter.includes(kw.title)) return true;

        // Combine current selection with this keyword
        const test_keywords = [...this.keywords_filter, kw.title];

        // Check if any stack has ALL of these keywords
        return this.stacks_without_keyword_filter.some((stack) => {
          if (!stack.keywords || !Array.isArray(stack.keywords)) return false;
          return test_keywords.every((kwf) => stack.keywords.includes(kwf));
        });
      });
    },
    opened_stack() {
      if (!this.$route.query?.stack) return false;
      return this.all_stacks.find(
        (s) => this.getFilename(s.$path) === this.$route.query.stack
      );
    },
  },
  methods: {
    async checkExistingFolder() {
      // Load all folders
      this.all_folders = await this.$api.getFolders({ path: "folders" });

      // Verify active paths exist
      if (this.active_folder_paths.length > 0) {
        const allExist = this.active_folder_paths.every((path) =>
          this.all_folders.find((f) => f.$path === path)
        );
        if (!allExist) {
          // Some paths don't exist, redirect to selection
          this.$router.push({ path: "/explore" });
        }
      }
    },
    toggleMediaFocus(path) {
      const slug = this.getFilename(path);
      this.openStack(slug);
    },
    openStack(stack_slug) {
      let query = Object.assign({}, this.$route.query) || {};
      query.stack = stack_slug;
      this.$router.push({ query });
    },
    isFavorite(stack_path) {
      if (
        !this.connected_as?.favorites ||
        this.connected_as.favorites.length === 0
      )
        return false;
      return this.connected_as.favorites.some(
        (f) => f.stack_path === stack_path
      );
    },
    async toggleFav(stack_path) {
      let favorites = this.connected_as?.favorites
        ? this.connected_as.favorites.slice()
        : [];

      if (this.isFavorite(stack_path))
        favorites = favorites.filter((f) => f.stack_path !== stack_path);
      else
        favorites.push({
          stack_path,
          added: +new Date(),
        });

      await this.$api.updateMeta({
        path: this.connected_as.$path,
        new_meta: {
          favorites,
        },
      });
    },
    closeStack() {
      let query = Object.assign({}, this.$route.query) || {};
      delete query.stack;
      this.$router.push({ query });
    },
    toggleCorpus(folder_path) {
      this.$emit("toggleCorpus", folder_path);
    },
    isCommunityActive(folder_path) {
      return this.active_folder_paths.includes(folder_path);
    },
    getFolderTitle(folder_path) {
      const folder = this.all_folders.find((f) => f.$path === folder_path);
      return folder ? folder.title || this.$t("untitled") : this.$t("untitled");
    },
    joinRoom(roomPath) {
      // Only join if we haven't already joined this room
      if (!this.joined_rooms.has(roomPath)) {
        this.$api.join({ room: roomPath });
        this.joined_rooms.add(roomPath);
      }
    },
    async loadStacksFromCommunities() {
      // Load stacks from all active communities and merge them
      const allStacksPromises = this.stack_shared_folder_paths.map(
        (stackPath) => this.$api.getFolders({ path: stackPath })
      );

      const stacksArrays = await Promise.all(allStacksPromises);
      // Merge all stacks and remove duplicates by path
      const stacksMap = new Map();
      stacksArrays.flat().forEach((stack) => {
        if (!stacksMap.has(stack.$path)) {
          stacksMap.set(stack.$path, stack);
        }
      });
      this.all_stacks = Array.from(stacksMap.values());
    },
  },
};
</script>
<style lang="scss" scoped>
._sharedFolder {
  position: relative;
  height: 100%;
  overflow: hidden;

  display: flex;
  flex-flow: column nowrap;

  > ._topBar {
    flex: 0 0 auto;
  }

  > ._sharedFolder--content {
    flex: 1 1 0;
  }
}

._sharedFolder--content {
  position: relative;
  overflow: auto;
  height: 100%;

  @include scrollbar(3px, 4px, 4px, transparent, var(--c-noir));
}

._dayFileSection {
  text-align: center;
  margin: 0 calc(var(--spacing) / 1);
}

._itemGrid {
  display: grid;
  grid-template-columns: repeat(
    auto-fill,
    minmax(var(--stack_preview_width, 120px), 1fr)
  );
  align-items: baseline;
  gap: calc(var(--stack_preview_width, 120px) / 10)
    calc(var(--stack_preview_width, 120px) / 40);
}

._itemGrid.is--compact {
  gap: 2px;
}

._stacksList {
  background: var(--sd-bg);
  color: var(--sd-textcolor);
  width: 100%;
  height: 100%;
}

._label {
  position: sticky;
  top: 0;
  z-index: 10;
  font-size: var(--sl-font-size-xx-large);
  font-weight: 400;
  padding: calc(var(--spacing) / 4);
  margin-top: calc(var(--spacing) / 2);
  background-color: var(--body-bg);
}

._stackModal {
  --sd-separator: var(--h-200);
  --sd-textcolor: var(--h-900);
  --sd-bg: var(--body-bg);
}

._footer {
  text-align: center;
  margin: calc(var(--spacing) * 2) auto;
}

._loader {
  position: relative;
  min-height: 80vh;
}

._mediamapContainer {
  width: 100%;
  height: 100%;
}

._noContent {
  margin: calc(var(--spacing) / 1);
}
._topBar {
  background: var(--h-100);
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 2);
  padding: calc(var(--spacing) / 4) calc(var(--spacing) * 2);
  border-bottom: 1px solid var(--h-200);
  overflow-x: auto;
  overflow-y: hidden;

  @include scrollbar(3px, 4px, 4px, transparent, var(--c-noir));

  :deep(._content) {
    margin-right: 0;
  }
}

._communautesSection {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 2);
  flex: 1 1 auto;
  min-width: 0;
}

._communautesLabel {
  flex: 0 0 auto;
  margin-bottom: 0;
}

._corpusItems {
  display: flex;
  flex-flow: row nowrap;
  gap: calc(var(--spacing) / 2);
  overflow-x: auto;
  overflow-y: hidden;
  flex: 1 1 auto;
  min-width: 0;
  // padding: calc(var(--spacing) / 4) 0;

  @include scrollbar(3px, 4px, 4px, transparent, var(--c-noir));
}

._corpusItem {
  flex: 0 0 auto;
  display: flex;
  flex-flow: row nowrap;
  align-items: center;

  // &.is--active {
  //   .u-button {
  //     background: var(--active-color);
  //     color: white;
  //     border-color: var(--active-color);
  //   }
  // }
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
