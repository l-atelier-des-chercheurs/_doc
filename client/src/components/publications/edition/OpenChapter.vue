<template>
  <div class="_openChapter">
    <!-- <div class="_close_button">
      <button
        type="button"
        class="u-button u-button_icon"
        @click="$emit('close')"
      >
        <b-icon icon="x-lg" :label="$t('close')" />
      </button>
    </div> -->
    <div class="_navBtns">
      <div class="_navBtns--content">
        <div class="" />
        <div
          class="_navBtns--content--buttons"
          v-show="next_section || prev_section"
        >
          <span>
            <button
              type="button"
              class="u-linkList"
              v-if="prev_section"
              @click="$emit('prev')"
            >
              <b-icon icon="arrow-left-short" />
              <span>
                {{ prev_section.section_title }}
              </span>
            </button>
            <span v-else>–</span>
          </span>

          <span class="_separator">|</span>

          <span>
            <button
              type="button"
              class="u-linkList"
              v-if="next_section"
              @click="$emit('next')"
            >
              <span>
                {{ next_section.section_title }}
              </span>
              <b-icon icon="arrow-right-short" />
            </button>
            <span v-else>–</span>
          </span>
        </div>
        <div>
          <button type="button" class="u-linkList" @click="$emit('close')">
            <b-icon icon="x-circle" :label="$t('close')" />
            {{ $t("close") }}
          </button>
        </div>
      </div>
    </div>
    <div class="_openChapter--content">
      <div class="_topButtons">
        <TitleField
          :field_name="'section_title'"
          :content="chapter.section_title"
          :maxlength="100"
          :tag="'h1'"
          :path="chapter.$path"
          :can_edit="true"
        />

        <DropDown :right="true" :show_label="false">
          <RemoveMenu @remove="$emit('remove')" />
        </DropDown>

        <!-- <div>
        <button type="button" class="u-buttonLink" @click="$emit('close')">
          <b-icon icon="arrow-left-short" />
          {{ $t("back") }}
        </button>
      </div> -->
        <!-- <SelectField
        :field_name="'content_type'"
        :content="content_type"
        :path="chapter._main_text.$path"
        :options="[
          { key: 'html', text: 'HTML' },
          { key: 'markdown', text: 'Markdown' },
        ]"
        :can_edit="can_edit"
        :hide_validation="true"
      /> -->
      </div>

      <div class="_infos">
        <div class="_content--type">
          <template v-if="chapter.section_type === 'text'">
            <b-icon icon="markdown" />
            {{ $t("text") }}
          </template>
          <template v-else-if="chapter.section_type === 'gallery'">
            <b-icon icon="image" />
            {{ $t("gallery") }}
          </template>
          <template v-else-if="chapter.section_type === 'story'">
            <b-icon icon="list" />
            {{ $t("story") }}
          </template>
        </div>

        <transition name="fade" mode="out-in">
          <div v-if="view_mode === 'book' && chapter_position?.first_page">
            <div class="_selects--pageRange" :key="chapter_position.first_page">
              p.{{ chapter_position.first_page }}
              <template
                v-if="
                  chapter_position.first_page !== chapter_position.last_page
                "
              >
                <b-icon icon="arrow-right-short" /> p.{{
                  chapter_position.last_page
                }}
              </template>
            </div>
          </div>
        </transition>
      </div>

      <fieldset
        v-if="chapter.section_type === 'text' && view_mode === 'book'"
        class="u-spacingBottom _layout"
      >
        <legend>{{ $t("layout") }}</legend>
        <div class="_optionsRow">
          <div class="_colCount">
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
        </div>
      </fieldset>

      <div class="_content">
        <template v-if="chapter.section_type === 'text'">
          <template v-if="chapter._main_text">
            <MainText
              :text_file="chapter._main_text"
              :medias_holder="chapter"
              :publication_path="publication.$path"
            />
          </template>
          <template v-else>
            <div class="u-instructions">
              {{ $t("no_content") }}
            </div>
          </template>
        </template>
        <template v-if="chapter.section_type === 'gallery'">
          <transition-group
            tag="div"
            class="_gallery"
            name="StoryModules"
            appear
          >
            <div
              class="_gallery--item"
              v-for="media in gallery_medias"
              :key="media.$path"
            >
              <MediaContent :file="media" :context="'full'" />
              <div class="_remove_media">
                <RemoveMenu
                  :show_button_text="false"
                  @remove="removeMedia(media)"
                />
              </div>
            </div>

            <div class="_add_medias" key="add_medias">
              <button
                type="button"
                class="u-button u-button_bleuvert"
                @click="show_media_picker = true"
              >
                {{ $t("add_medias") }}
              </button>
            </div>
          </transition-group>

          <MediaPicker
            v-if="show_media_picker"
            :publication_path="publication.$path"
            :select_mode="'multiple'"
            :pick_from_types="['image']"
            @pickMedias="pickMediasForGallery"
            @close="show_media_picker = false"
          />
        </template>
        <template v-if="chapter.section_type === 'story'">
          <SingleSection
            class="_singleSection"
            :publication="publication"
            :section="chapter"
            :can_edit="true"
          />
        </template>
      </div>
    </div>
  </div>
</template>
<script>
// import MarkdownEditor from "@/adc-core/fields/collaborative-editor/MarkdownEditor.vue";

import GalleryChapter from "@/components/publications/edition/GalleryChapter.vue";
import GridChapter from "@/components/publications/edition/GridChapter.vue";
import ChapterLayout from "@/components/publications/edition/ChapterLayout.vue";
import MainText from "@/components/publications/edition/MainText.vue";

export default {
  props: {
    chapter: Object,
    chapters: Array,
    prev_section: Object,
    next_section: Object,
    publication: Object,
    chapter_position: Object,
    view_mode: String,
  },
  components: {
    // MarkdownEditor,
    PickMediaForMarkdown,
    MediaPicker,
    SingleSection: () =>
      import("@/components/publications/story/SingleSection.vue"),
  },
  data() {
    return {};
  },
  created() {},
  mounted() {},
  beforeDestroy() {},

  watch: {},
  computed: {},
  methods: {},
};
</script>
<style lang="scss" scoped>
._openChapter {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  scroll-behavior: smooth;
  scroll-padding: 10vh;
  background: var(--c-gris_clair);
  z-index: 10;

  // display: flex;
  // flex-direction: row nowrap;

  // > * {
  //   flex: 1 1 0;
  // }
}
._openChapter--content {
  position: relative;
  min-height: calc(100% - calc(var(--spacing) * 1));
  background-color: white;
  // box-shadow: 0 0 0 1px hsla(230, 13%, 9%, 0.05),
  //   0 0.3px 0.4px hsla(230, 13%, 9%, 0.02),
  //   0 0.9px 1.5px hsla(230, 13%, 9%, 0.025),
  //   0 3.5px 6px hsla(230, 13%, 9%, 0.09);
  filter: drop-shadow(0 5px 10px rgba(0, 0, 0, 0.1));

  border-top-left-radius: 4px;
  border-top-right-radius: 4px;

  margin: 0 calc(var(--spacing) / 1);
  margin-bottom: 0;
  padding: calc(var(--spacing) * 2);
}

._close_button {
  position: sticky;
  height: 0;
  top: 0;
  text-align: right;
  right: 0;
  z-index: 100;
}

._topButtons {
  display: flex;
  flex-direction: row nowrap;
  justify-content: space-between;
  align-items: center;
  margin-bottom: calc(var(--spacing) * 1);
  padding-bottom: calc(var(--spacing) / 2);
}

._content {
  padding-bottom: calc(var(--spacing) * 1);
}

._navBtns {
  padding: calc(var(--spacing) / 2);
  // padding-top: calc(var(--spacing) * 4);
  // padding-bottom: calc(var(--spacing) * 4);
}

._navBtns--content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: calc(var(--spacing) / 1);
  height: 20px;

  &:first-child,
  &:last-child {
    flex: 0 0 10ch;
  }

  > * {
    flex: 0 1 10ch;
    overflow: hidden;
  }
}

._navBtns--content--buttons {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  justify-content: center;
  flex: 1 1 0;

  > * {
    display: block;
    flex: 1 0 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;

    > * {
      padding: calc(var(--spacing) / 4) calc(var(--spacing) / 2);
    }

    &:first-child {
      text-align: right;

      > * {
        justify-content: flex-end;
      }
    }

    &:last-child > * {
      text-align: left;
    }
  }

  ._separator {
    flex: 0 0 1ch;
    margin: 0ch;
    text-align: center;
  }
}

// ._customBtn {
// background-color: var(--c-bleumarine) !important;
// color: white !important;
// border-radius: var(--input-border-radius) !important;
// }

._textField {
  resize: vertical;
  min-height: 8rem;
}
._content--type {
  .b-icon {
    vertical-align: middle;
  }
}

._infos {
  margin-bottom: calc(var(--spacing) * 1);
  font-size: var(--sl-font-size-small);
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: baseline;
  gap: calc(var(--spacing) * 1);
}

._gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: calc(var(--spacing) * 1);

  ._add_medias {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
  }

  ._gallery--item {
    position: relative;
    aspect-ratio: 1/1;

    border: 2px solid var(--c-gris_clair);
    // aspect-ratio: 1/1;
    overflow: hidden;

    ::v-deep {
      ._mediaContent {
        width: 100%;
        height: 100%;
      }

      ._mediaContent--image,
      .plyr--video,
      .plyr__poster,
      ._mediaContent--iframe,
      ._iframeStylePreview {
        position: absolute;
        height: 100%;
        width: 100%;
        top: 0;
        left: 0;
        object-fit: scale-down;
        background-size: scale-down;
        background-color: var(--c-gris_clair);
      }
    }
  }
}

._remove_media {
  position: absolute;
  top: 0;
  right: 0;
  margin: calc(var(--spacing) / 2);
}

._selects--starts_on_page {
  width: 30ch;
  // width: auto;
  flex: 0 0 auto;
  position: relative;
  z-index: 2;
  // margin-bottom: calc(var(--spacing) * 1);
}

._selects--pageRange {
  // font-size: var(--sl-font-size);
  color: var(--c-gris_fonce);
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  // gap: calc(var(--spacing) / 2);
}

._singleSection {
  ::v-deep {
    ._topbar {
      display: none;
    }
  }
}

._colCount {
  max-width: 20ch;
}

._optionsRow {
  display: flex;
  flex-flow: row nowrap;
  align-items: flex-start;
  gap: calc(var(--spacing) * 1);
}
</style>
