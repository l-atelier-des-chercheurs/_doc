<template>
  <div class="_communityItem">
    <div class="_communityHeader">
      <div class="_communityTitle">
        <TitleField
          :field_name="'title'"
          :label="$t('title')"
          :show_label="false"
          :content="folder.title || ''"
          :path="folder.$path"
          :can_edit="can_edit"
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
    </div>
    <div class="_selectContainer">
      <label v-if="can_open_community" :for="folder.$path" class="_selectLabel">
        <span class="_selectText">{{ $t("select") }}</span>
        <input
          type="checkbox"
          :id="folder.$path"
          :value="folder.$path"
          :checked="is_selected"
          @change="$emit('select', folder.$path, $event.target.checked)"
          class="_checkbox"
        />
      </label>
      <div v-else class="_joinLabel">
        <div>{{ $t("access_restricted") }}</div>
        <button
          v-if="folder.$admins && folder.$admins.length > 0"
          type="button"
          class="u-button u-button_primary"
          @click="showAskToJoin = true"
        >
          {{ $t("ask_to_join") }}
        </button>
        <BaseModal2
          v-if="showAskToJoin"
          :title="$t('ask_to_join')"
          @close="showAskToJoin = false"
        >
          <div style="margin-bottom: 1.5em">
            <p>
              {{ $t("send_email_to_admins") }}
            </p>
            <ul>
              <li v-for="admin in folder.$admins" :key="admin.email">
                <a :href="mailtoLink(admin.email)">
                  {{ admin.email }}
                </a>
              </li>
            </ul>
            <p>
              {{ $t("email_instructions") }}
            </p>
          </div>
          <div style="text-align: right">
            <button
              type="button"
              class="u-button"
              @click="showAskToJoin = false"
            >
              {{ $t("close") }}
            </button>
          </div>
        </BaseModal2>
      </div>
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
    is_selected: {
      type: Boolean,
      default: false,
    },
  },
  components: {
    DropDown,
  },
  i18n: {
    messages: {
      fr: {
        access_restricted: "Accès restreint",
        ask_to_join: "Demander à rejoindre",
        send_email_to_admins: "Envoyer un email aux administrateurs",
        email_instructions:
          "Les administrateurs recevront un email avec votre demande.",
      },
    },
    en: {
      access_restricted: "Access restricted",
      ask_to_join: "Ask to join",
      send_email_to_admins: "Send email to admins",
      email_instructions: "The admins will receive an email with your request.",
    },
  },
  data() {
    return {
      showAskToJoin: false,
    };
  },
  computed: {
    can_edit() {
      return this.canLoggedinEditFolder({ folder: this.folder });
    },
    can_see() {
      return this.canLoggedinSeeFolder({ folder: this.folder });
    },
    can_open_community() {
      return this.can_edit || this.can_see || this.folder.$status !== "private";
    },
  },
  methods: {
    mailtoLink(email) {
      return `mailto:${email}`;
    },
  },
};
</script>
<style lang="scss" scoped>
._communityItem {
  position: relative;
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing) / 2);
  padding: calc(var(--spacing) * 0.5) calc(var(--spacing) * 1)
    calc(var(--spacing) * 1);

  background-color: white;
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
  // font-weight: 500;
  font-size: var(--sl-font-size-large);
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
  flex-flow: row nowrap;
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

._joinLabel {
  font-size: var(--sl-font-size-small);
  color: var(--h-600);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-weight: 500;

  text-align: center;
}
</style>
