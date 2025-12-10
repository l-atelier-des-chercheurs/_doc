<template>
  <div class="_corpusManager">
    <div v-if="!hasSelectedCommunities" class="_selectionView">
      <div class="_header">
        <h1 class="_title">{{ $t("Communautés") }}</h1>
        <div class="_headerActions">
          <button
            type="button"
            class="u-button u-button_icon u-button_transparent _addCommunityButton"
            @click="show_add_community = true"
            :title="$t('add_community')"
          >
            <b-icon icon="plus" />
            {{ $t("add_community") }}
          </button>
        </div>
      </div>
      <div class="_communitiesList">
        <CommunityPreview
          v-for="folder in displayed_folders"
          :key="folder.$path"
          :folder="folder"
          :is_selected="selected_folders.includes(folder.$path)"
          @select="handleSelect"
          @remove="showRemoveModal"
        />
        <div v-if="displayed_folders.length === 0" class="_noCommunities">
          {{ $t("no_communities_available") }}
        </div>
      </div>
      <div class="_actions">
        <button
          type="button"
          class="u-button u-button_primary"
          :disabled="selected_folders.length === 0"
          @click="openSelectedCommunities"
        >
          {{ $t("open") }}
        </button>
      </div>
    </div>

    <!-- Content view: show when communities are selected -->
    <SharedFolder2
      v-else
      :shared_folder_paths="computed_shared_folder_paths"
      :select_mode="select_mode"
      :read_only="read_only"
      @toggleCorpus="toggleCorpus"
      @selectStack="$emit('selectStack', $event)"
      @selectMedias="$emit('selectMedias', $event)"
    />

    <!-- Add Community Modal -->
    <CreateFolder
      v-if="show_add_community"
      :modal_name="$t('add_community')"
      path="folders"
      @close="show_add_community = false"
      @openNew="onCommunityCreated"
    />

    <!-- Remove Community Modal -->
    <RemoveMenu2
      v-if="community_to_remove"
      :path="community_to_remove.$path"
      :modal_title="$t('remove_community')"
      :modal_expl="$t('remove_community_explanation')"
      :success_notification="$t('community_removed_successfully')"
      @close="community_to_remove = null"
      @removedSuccessfully="onCommunityRemoved"
    />
  </div>
</template>
<script>
import SharedFolder2 from "@/components/archive/SharedFolder2.vue";
import CommunityPreview from "@/components/archive/CommunityPreview.vue";
import CreateFolder from "@/adc-core/modals/CreateFolder.vue";

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
    read_only: {
      type: Boolean,
      default: false,
    },
    use_query: {
      type: Boolean,
      default: true,
    },
  },
  components: {
    SharedFolder2,
    CommunityPreview,
    CreateFolder,
  },
  data() {
    return {
      all_folders: [],
      selected_folders: [],
      show_add_community: false,
      community_to_remove: null,
    };
  },
  async created() {},
  async mounted() {
    // Load all available communities
    this.all_folders = await this.$api.getFolders({ path: "folders" });
    this.$api.join({ room: "folders" });

    // Initialize selected folders
    if (this.use_query) {
      // If route has query params, use those (override localStorage)
      if (this.$route && this.$route.query && this.$route.query.communities) {
        const queryPaths = this.computed_shared_folder_paths;
        if (queryPaths.length > 0) {
          this.selected_folders = queryPaths.slice();
          // Save to localStorage
          this.saveToLocalStorage(this.selected_folders);
        }
      } else {
        // No query params, try to load from localStorage
        const saved = this.loadFromLocalStorage();
        if (saved && saved.length > 0) {
          // Verify that saved communities still exist
          const validPaths = saved.filter((path) =>
            this.all_folders.find((f) => f.$path === path)
          );
          if (validPaths.length > 0) {
            this.selected_folders = validPaths;
            // Update route to reflect localStorage
            const slugs = validPaths.map((path) => path.split("/").pop());
            this.$router.replace({
              path: this.$route.path,
              query: { communities: slugs.join(",") },
            });
          }
        }
      }
    } else {
      // If using local state, try to get from prop first, then localStorage
      if (this.shared_folder_paths && this.shared_folder_paths.length > 0) {
        this.selected_folders = this.shared_folder_paths.slice();
      } else {
        // Load from localStorage
        const saved = this.loadFromLocalStorage();
        if (saved && saved.length > 0) {
          // Verify that saved communities still exist
          const validPaths = saved.filter((path) =>
            this.all_folders.find((f) => f.$path === path)
          );
          if (validPaths.length > 0) {
            this.selected_folders = validPaths;
            // Update parent component (this will trigger the watcher to save to localStorage)
            this.$nextTick(() => {
              this.$emit("communitiesSelected", this.selected_folders);
            });
          }
        }
      }
    }
  },
  beforeDestroy() {},
  watch: {
    computed_shared_folder_paths: {
      handler(newPaths) {
        if (this.use_query) {
          // When query params are present, use them (override localStorage)
          if (
            this.$route &&
            this.$route.query &&
            this.$route.query.communities
          ) {
            this.selected_folders = newPaths ? newPaths.slice() : [];
            // Save to localStorage (will be saved by selected_folders watcher too)
          }
          // If no query params, don't update from computed (let localStorage handle it in mounted)
        }
      },
      immediate: true,
    },
    shared_folder_paths: {
      handler(newPaths) {
        if (!this.use_query) {
          this.selected_folders = newPaths ? newPaths.slice() : [];
        }
      },
      immediate: true,
    },
    selected_folders: {
      handler(newFolders) {
        // Always save to localStorage
        this.saveToLocalStorage(newFolders);
      },
      deep: true,
    },
  },
  computed: {
    computed_shared_folder_paths() {
      if (this.use_query) {
        // Use route query
        if (this.$route && this.$route.query && this.$route.query.communities) {
          const slugs = this.$route.query.communities
            .split(",")
            .filter(Boolean);
          return slugs.map((slug) => `folders/${slug.trim()}`);
        }
        return [];
      } else {
        // Use local state (prop)
        return this.shared_folder_paths || [];
      }
    },
    hasSelectedCommunities() {
      return (
        this.computed_shared_folder_paths &&
        this.computed_shared_folder_paths.length > 0
      );
    },
    available_folders_to_add() {
      // Return folders that are not already selected
      return this.all_folders.filter(
        (folder) => !this.selected_folders.includes(folder.$path)
      );
    },
    displayed_folders() {
      // Show all folders by default, or filter if needed
      return this.all_folders;
    },
  },
  methods: {
    openSelectedCommunities() {
      if (this.selected_folders.length === 0) return;

      if (this.use_query) {
        // Update route query
        const slugs = this.selected_folders.map((path) =>
          path.split("/").pop()
        );
        this.$router.push({
          path: this.$route.path,
          query: { communities: slugs.join(",") },
        });
      } else {
        // Emit event for local state management
        this.$emit("communitiesSelected", this.selected_folders);
      }
    },
    handleSelect(folder_path, is_selected) {
      if (is_selected) {
        if (!this.selected_folders.includes(folder_path)) {
          this.selected_folders.push(folder_path);
        }
      } else {
        this.selected_folders = this.selected_folders.filter(
          (f) => f !== folder_path
        );
      }
    },
    toggleCorpus(path) {
      if (this.use_query) {
        // Toggle via route query
        const slug = path.split("/").pop();
        const currentSlugs = this.computed_shared_folder_paths.map((p) =>
          p.split("/").pop()
        );
        const isActive = currentSlugs.includes(slug);

        let newSlugs;
        if (isActive) {
          // Remove it
          newSlugs = currentSlugs.filter((s) => s !== slug);
        } else {
          // Add it
          newSlugs = [...currentSlugs, slug];
        }

        if (newSlugs.length === 0) {
          // If no communities selected, go to explore without params
          this.$router.push({ path: this.$route.path });
        } else {
          this.$router.push({
            path: this.$route.path,
            query: { communities: newSlugs.join(",") },
          });
        }
      } else {
        // Toggle via local state
        const isActive = this.selected_folders.includes(path);
        if (isActive) {
          this.selected_folders = this.selected_folders.filter(
            (f) => f !== path
          );
        } else {
          this.selected_folders.push(path);
        }
        // Update local shared_folder_paths and emit
        this.$emit("communitiesSelected", this.selected_folders);
      }
    },
    async onCommunityCreated(new_folder_slug) {
      // Add the newly created community to selected list
      // const new_folder_path = `folders/${new_folder_slug}`;
      // if (!this.selected_folders.includes(new_folder_path)) {
      //   this.selected_folders.push(new_folder_path);
      // }

      this.show_add_community = false;

      // if (this.use_query) {
      //   // Update route query
      //   const slugs = this.selected_folders.map((path) =>
      //     path.split("/").pop()
      //   );
      //   this.$router.push({
      //     path: this.$route.path,
      //     query: { communities: slugs.join(",") },
      //   });
      // } else {
      //   // Emit event for local state management
      //   this.$emit("communitiesSelected", this.selected_folders);
      // }
    },
    addCommunity(folder_path) {
      // Add community to selected list if not already selected
      if (!this.selected_folders.includes(folder_path)) {
        this.selected_folders.push(folder_path);
      }
      this.show_add_community = false;

      if (this.use_query) {
        // Update route query
        const slugs = this.selected_folders.map((path) =>
          path.split("/").pop()
        );
        this.$router.push({
          path: this.$route.path,
          query: { communities: slugs.join(",") },
        });
      } else {
        // Emit event for local state management
        this.$emit("communitiesSelected", this.selected_folders);
      }
    },
    showRemoveModal(folder) {
      this.community_to_remove = folder;
    },
    async onCommunityRemoved() {
      // Remove from local list
      this.all_folders = this.all_folders.filter(
        (f) => f.$path !== this.community_to_remove.$path
      );
      this.selected_folders = this.selected_folders.filter(
        (f) => f !== this.community_to_remove.$path
      );

      if (this.use_query) {
        // Update route query
        const currentSlugs = this.computed_shared_folder_paths.map((p) =>
          p.split("/").pop()
        );
        const removedSlug = this.community_to_remove.$path.split("/").pop();
        const newSlugs = currentSlugs.filter((s) => s !== removedSlug);

        if (newSlugs.length === 0) {
          this.$router.push({ path: this.$route.path });
        } else {
          this.$router.push({
            path: this.$route.path,
            query: { communities: newSlugs.join(",") },
          });
        }
      } else {
        // Emit event to parent for local state management
        this.$emit("communityRemoved", this.community_to_remove.$path);
      }

      this.community_to_remove = null;
    },
    saveToLocalStorage(folders) {
      try {
        if (folders && folders.length > 0) {
          localStorage.setItem(
            "corpusManager.selected_communities",
            JSON.stringify(folders)
          );
        } else {
          localStorage.removeItem("corpusManager.selected_communities");
        }
      } catch (error) {
        console.warn(
          "Failed to save selected communities to localStorage",
          error
        );
      }
    },
    loadFromLocalStorage() {
      try {
        const saved = localStorage.getItem(
          "corpusManager.selected_communities"
        );
        if (saved) {
          return JSON.parse(saved);
        }
      } catch (error) {
        console.warn(
          "Failed to load selected communities from localStorage",
          error
        );
      }
      return null;
    },
  },
};
</script>
<style lang="scss" scoped>
._corpusManager {
  height: 100%;
  display: flex;
  flex-flow: column nowrap;
}

._selectionView {
  padding: calc(var(--spacing) * 2);
  // max-width: 800px;
  margin: 0 auto;
  width: 100%;
}

._header {
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
  align-items: center;
  margin-bottom: calc(var(--spacing) * 2);
  gap: calc(var(--spacing));
}

._title {
  font-size: var(--sl-font-size-xx-large);
  font-weight: 400;
  margin: 0;
}

._headerActions {
  display: flex;
  flex-flow: row nowrap;
  gap: calc(var(--spacing) / 2);
  align-items: center;
}

._addCommunityButton {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: calc(var(--spacing) / 4);
}

._communitiesList {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: calc(var(--spacing) / 2);
  margin-bottom: calc(var(--spacing) * 2);
}

._actions {
  position: sticky;
  bottom: 0;
  width: 100%;
  text-align: center;
  // background-color: var(--h-50);
  padding: calc(var(--spacing) / 2);
  // border-top: 1px solid var(--h-200);
}

._noCommunities {
  padding: calc(var(--spacing) * 2);
  text-align: center;
  color: var(--h-600);
  font-style: italic;
}
</style>
