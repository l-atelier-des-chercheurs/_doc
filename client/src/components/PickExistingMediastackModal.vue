<template>
  <BaseModal2
    :title="modal_title || $t('pick_existing_mediastack')"
    :is_closable="true"
    :size="'x-large'"
    @close="$emit('close')"
  >
    <div class="_pickExistingMediastackModal">
      <!-- <div class="u-spacingBottom">
        <DLabel :str="$t('corpus')" />
        <DestinationCorpusSelector
          :selected_destination_folder_path.sync="
            selected_destination_folder_path
          "
        />
      </div> -->

      <div class="_stackPickerFrame">
        <CorpusManager
          :shared_folder_paths="shared_folder_paths"
          :select_mode="select_mode"
          :read_only="true"
          :use_query="false"
          @communitiesSelected="onCommunitiesSelected"
          @selectStack="$emit('stackSelected', $event)"
          @selectMedias="$emit('mediasSelected', $event)"
        />
      </div>
    </div>
  </BaseModal2>
</template>
<script>
import DestinationCorpusSelector from "@/components/DestinationCorpusSelector.vue";
import CorpusManager from "@/components/archive/CorpusManager.vue";

export default {
  props: {
    modal_title: String,
    select_mode: {
      type: String,
      default: "single_stack",
    },
  },
  components: {
    DestinationCorpusSelector,
    CorpusManager,
  },
  data() {
    return {
      shared_folder_paths: [],
    };
  },
  computed: {},
  methods: {
    onCommunitiesSelected(selected_folders) {
      // When communities are selected, update the shared_folder_paths
      this.shared_folder_paths = selected_folders;
    },
  },
  i18n: {
    messages: {
      fr: {
        pick_existing_mediastack: "Choisir un document",
        corpus: "Corpus",
      },
      en: {
        pick_existing_mediastack: "Pick document",
        corpus: "Corpus",
      },
    },
  },
};
</script>
<style lang="scss" scoped>
._pickExistingMediastackModal {
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 2);

  > * {
    flex: 1 1 auto;
  }
}

._stackPickerFrame {
  border: 1px solid var(--c-gris);
  // border-radius: var(--input-border-radius);
  overflow: auto;
  height: 70dvh;
}
</style>
