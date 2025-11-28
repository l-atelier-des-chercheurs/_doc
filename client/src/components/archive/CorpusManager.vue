<template>
  <div class="_corpusManager">
    <!-- Selection view: show when no communities are selected -->
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
        <div
          v-for="folder in displayed_folders"
          :key="folder.$path"
          class="_communityItem"
        >
          <input
            type="checkbox"
            :id="folder.$path"
            :value="folder.$path"
            v-model="selected_folders"
            class="_checkbox"
          />
          <label :for="folder.$path" class="_label">
            <div class="_communityTitle">
              {{ folder.title || $t("untitled") }}
            </div>
            <div v-if="folder.description" class="_communityDescription">
              {{ folder.description }}
            </div>
          </label>
          <div class="_communityActions">
            <button
              type="button"
              class="u-button u-button_icon u-button_transparent _editButton"
              @click.stop="editCommunity(folder)"
              :title="$t('edit')"
            >
              <b-icon icon="pencil" />
            </button>
            <button
              type="button"
              class="u-button u-button_icon u-button_transparent _removeButton"
              @click.stop="showRemoveModal(folder)"
              :title="$t('remove')"
            >
              <b-icon icon="trash" />
            </button>
          </div>
        </div>
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
    <BaseModal2
      v-if="show_add_community"
      :title="$t('add_community')"
      @close="show_add_community = false"
    >
      <div class="_addCommunityList">
        <div
          v-for="folder in available_folders_to_add"
          :key="folder.$path"
          class="_addCommunityItem"
        >
          <button
            type="button"
            class="u-button u-button_white _addCommunityItemButton"
            @click="addCommunity(folder.$path)"
          >
            {{ folder.title || $t("untitled") }}
          </button>
        </div>
        <div
          v-if="available_folders_to_add.length === 0"
          class="_noMoreCommunities"
        >
          {{ $t("no_more_communities") }}
        </div>
      </div>
    </BaseModal2>

    <!-- Edit Community Modal -->
    <BaseModal2
      v-if="editing_community"
      :title="$t('edit_community')"
      @close="cancelEdit"
    >
      <div class="_editCommunityForm">
        <div class="u-spacingBottom">
          <DLabel :str="$t('title')" />
          <TextInput
            v-model="edit_title"
            :input_type="'text'"
            :required="true"
            :maxlength="100"
            :autofocus="true"
          />
        </div>
        <div class="u-spacingBottom">
          <DLabel :str="$t('description')" />
          <TextInput
            v-model="edit_description"
            :input_type="'text'"
            :maxlength="500"
          />
        </div>
        <SaveCancelButtons
          slot="footer"
          :is_saving="is_saving"
          :allow_save="edit_title && edit_title.length > 0"
          @save="saveEdit"
          @cancel="cancelEdit"
        />
      </div>
    </BaseModal2>

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
  },
  data() {
    return {
      all_folders: [],
      selected_folders: [],
      show_add_community: false,
      editing_community: null,
      edit_title: "",
      edit_description: "",
      is_saving: false,
      community_to_remove: null,
    };
  },
  async created() {},
  async mounted() {
    // Load all available communities
    this.all_folders = await this.$api.getFolders({ path: "folders" });

    // Initialize selected folders from shared_folder_paths or query
    if (this.use_query) {
      // If using query, get from route
      if (this.computed_shared_folder_paths.length > 0) {
        this.selected_folders = this.computed_shared_folder_paths.slice();
      }
    } else {
      // If using local state, get from prop
      if (this.shared_folder_paths.length > 0) {
        this.selected_folders = this.shared_folder_paths.slice();
      }
    }
  },
  beforeDestroy() {},
  watch: {
    computed_shared_folder_paths: {
      handler(newPaths) {
        if (this.use_query) {
          this.selected_folders = newPaths ? newPaths.slice() : [];
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
    editCommunity(folder) {
      this.editing_community = folder;
      this.edit_title = folder.title || "";
      this.edit_description = folder.description || "";
    },
    async saveEdit() {
      if (!this.editing_community || !this.edit_title) return;

      this.is_saving = true;
      try {
        await this.$api.updateMeta({
          path: this.editing_community.$path,
          new_meta: {
            title: this.edit_title,
            description: this.edit_description,
          },
        });

        // Update local folder data
        const folder = this.all_folders.find(
          (f) => f.$path === this.editing_community.$path
        );
        if (folder) {
          folder.title = this.edit_title;
          folder.description = this.edit_description;
        }

        this.$alertify
          .closeLogOnClick(true)
          .delay(4000)
          .success(this.$t("community_updated_successfully"));

        this.cancelEdit();
      } catch (error) {
        this.$alertify
          .closeLogOnClick(true)
          .delay(4000)
          .error(this.$t("error_updating_community"));
      }
      this.is_saving = false;
    },
    cancelEdit() {
      this.editing_community = null;
      this.edit_title = "";
      this.edit_description = "";
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
  max-width: 800px;
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
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 2);
  margin-bottom: calc(var(--spacing) * 2);
}

._communityItem {
  display: flex;
  flex-flow: row nowrap;
  align-items: flex-start;
  gap: calc(var(--spacing) / 2);
  padding: calc(var(--spacing) / 2);
  border: 1px solid var(--h-300);
  border-radius: 4px;
  background: var(--body-bg);
  transition: all 0.2s ease;

  &:hover {
    border-color: var(--h-400);
    background: var(--h-100);
  }
}

._checkbox {
  flex: 0 0 auto;
  margin-top: calc(var(--spacing) / 4);
}

._label {
  flex: 1 1 auto;
  cursor: pointer;
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 4);
}

._communityTitle {
  font-weight: 500;
  font-size: var(--sl-font-size-medium);
}

._communityDescription {
  font-size: var(--sl-font-size-small);
  color: var(--h-600);
}

._communityActions {
  display: flex;
  flex-flow: row nowrap;
  gap: calc(var(--spacing) / 4);
  align-items: center;
  flex: 0 0 auto;
}

._editButton,
._removeButton {
  opacity: 0.6;
  transition: opacity 0.2s ease;

  &:hover {
    opacity: 1;
  }
}

._actions {
  display: flex;
  flex-flow: row nowrap;
  justify-content: flex-end;
  gap: calc(var(--spacing) / 2);
}

._noCommunities {
  padding: calc(var(--spacing) * 2);
  text-align: center;
  color: var(--h-600);
  font-style: italic;
}

._addCommunityList {
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 2);
  max-height: 60vh;
  overflow-y: auto;
  padding: calc(var(--spacing) / 2);
}

._addCommunityItem {
  display: flex;
  flex-flow: row nowrap;
}

._addCommunityItemButton {
  width: 100%;
  justify-content: flex-start;
}

._noMoreCommunities {
  padding: calc(var(--spacing) * 2);
  text-align: center;
  color: var(--h-600);
  font-style: italic;
}

._editCommunityForm {
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing));
}
</style>
