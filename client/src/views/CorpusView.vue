<template>
  <div class="_corpusView">
    <CorpusManager
      :shared_folder_paths="shared_folder_paths"
      @communitiesSelected="openSelectedCommunities"
      @toggleCorpus="toggleCorpus"
      @communityRemoved="onCommunityRemoved"
    />
  </div>
</template>
<script>
import CorpusManager from "@/components/archive/CorpusManager.vue";

export default {
  props: {},
  components: {
    CorpusManager,
  },
  computed: {
    shared_folder_paths() {
      if (this.$route.query.communities) {
        const slugs = this.$route.query.communities.split(",").filter(Boolean);
        return slugs.map((slug) => `folders/${slug.trim()}`);
      }
      return [];
    },
  },
  methods: {
    openSelectedCommunities(selected_folders) {
      if (!selected_folders || selected_folders.length === 0) return;

      const slugs = selected_folders.map((path) => path.split("/").pop());
      this.$router.push({
        path: "/explore",
        query: { communities: slugs.join(",") },
      });
    },
    toggleCorpus(path) {
      // Toggle a community in/out of the selection
      const slug = path.split("/").pop();
      const currentSlugs = this.shared_folder_paths.map((p) =>
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
        this.$router.push({ path: "/explore" });
      } else {
        this.$router.push({
          path: "/explore",
          query: { communities: newSlugs.join(",") },
        });
      }
    },
    onCommunityRemoved(removed_path) {
      // If the removed community was selected, remove it from URL
      const currentSlugs = this.shared_folder_paths.map((p) =>
        p.split("/").pop()
      );
      const removedSlug = removed_path.split("/").pop();
      const newSlugs = currentSlugs.filter((s) => s !== removedSlug);

      if (newSlugs.length === 0) {
        this.$router.push({ path: "/explore" });
      } else {
        this.$router.push({
          path: "/explore",
          query: { communities: newSlugs.join(",") },
        });
      }
    },
  },
};
</script>
<style lang="scss" scoped>
._corpusView {
  height: 100%;
  display: flex;
  flex-flow: column nowrap;
}
</style>
