<template>
  <div class="_communityItem">
    <div class="_communityHeader">
      <div class="_communityTitle">
        <TitleField
          :field_name="'title'"
          :label="$t('title')"
          :content="folder.title || ''"
          :path="folder.$path"
          :can_edit="can_edit"
          :show_label="true"
          :tag="'div'"
          :maxlength="100"
        />
      </div>
      <div class="_communityDescription">
        <TitleField
          :field_name="'description'"
          :label="$t('description')"
          :content="folder.description || ''"
          :path="folder.$path"
          :can_edit="can_edit"
          :show_label="true"
          :tag="'div'"
          :maxlength="500"
        />
      </div>
      <div class="_communityAuthor">
        <AdminsAndContributorsField
          :folder="folder"
          :can_edit="can_edit"
          :admin_label="$t('referent')"
          :admin_instructions="$t('community_admin_instructions')"
          :contrib_instructions="$t('community_contrib_instructions')"
        />
      </div>
    </div>
    <div class="_communityActions">
      <DropDown :show_label="false" :right="true" v-if="can_edit">
        <button
          type="button"
          class="u-buttonLink"
          @click.stop="$emit('remove', folder)"
        >
          <b-icon icon="trash" />
          {{ $t("remove") }}
        </button>
      </DropDown>
      {{ folder.$status }}
    </div>
    <div class="_selectContainer">
      <label :for="folder.$path" class="_selectLabel">
        <span class="_selectText" v-if="can_see">{{ $t("select") }}</span>
        <input
          type="checkbox"
          :id="folder.$path"
          :value="folder.$path"
          :checked="is_selected"
          @change="$emit('select', folder.$path, $event.target.checked)"
          class="_checkbox"
          :disabled="!can_see"
        />
      </label>
    </div>
  </div>
</template>
<script>
import DropDown from "@/adc-core/ui/DropDown.vue";

export default {
  name: "CommunityPreview",
  props: {
    folder: {
      type: Object,
      required: true,
    },
    can_edit: {
      type: Boolean,
      default: false,
    },
    can_see: {
      type: Boolean,
      default: true,
    },
    is_selected: {
      type: Boolean,
      default: false,
    },
  },
  components: {
    DropDown,
  },
};
</script>
<style lang="scss" scoped>
._communityItem {
  position: relative;
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 2);
  padding: calc(var(--spacing) * 0.5);

  background-color: var(--h-50);
  border: 1px solid var(--h-200);
  border-radius: 4px;
}

._checkbox {
  flex: 0 0 auto;
  cursor: pointer;
}

._communityHeader {
  flex: 1;
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 2);
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
  position: absolute;
  top: 0;
  right: 0;
}

._selectContainer {
  flex: 0 0 auto;
  display: flex;
  flex-flow: column nowrap;
  align-items: center;
  justify-content: center;
  flex: 0 0 auto;
  padding-left: calc(var(--spacing));
}

._selectLabel {
  display: flex;
  flex-flow: column nowrap;
  align-items: center;
  justify-content: center;
  gap: calc(var(--spacing) / 4);
  cursor: pointer;
}

._selectText {
  font-size: var(--sl-font-size-small);
  color: var(--h-600);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-weight: 500;
}
</style>
