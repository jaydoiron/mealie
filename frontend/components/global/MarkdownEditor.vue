<template>
  <div>
    <div v-if="displayPreview" class="d-flex justify-end">
      <BaseButtonGroup
        :buttons="[
          {
            icon: previewState ? $globals.icons.edit : $globals.icons.eye,
            text: previewState ? $tc('general.edit') : 'Preview Markdown',
            event: 'toggle',
          },
        ]"
        @toggle="previewState = !previewState"
      />
    </div>
    <v-textarea
      v-if="!previewState"
      v-bind="textarea"
      v-model="inputVal"
      :class="label == '' ? '' : 'mt-5'"
      :label="label"
      auto-grow
      dense
      rows="4"
    />
    <VueMarkdown v-else :source="value" />
  </div>
</template>

<script lang="ts">
// @ts-ignore vue-markdown has no types
import VueMarkdown from "@adapttive/vue-markdown";

import { defineComponent, computed, ref } from "@nuxtjs/composition-api";

export default defineComponent({
  name: "MarkdownEditor",
  components: {
    VueMarkdown,
  },
  props: {
    value: {
      type: String,
      required: true,
    },
    label: {
      type: String,
      default: "",
    },
    preview: {
      type: Boolean,
      default: undefined,
    },
    displayPreview: {
      type: Boolean,
      default: true,
    },
    textarea: {
      type: Object,
      default: () => ({}),
    },
  },
  setup(props, context) {
    const fallbackPreview = ref(false);
    const previewState = computed({
      get: () => {
        return props.preview ?? fallbackPreview.value;
      },
      set: (val) => {
        if (props.preview) {
          context.emit("input:preview", val);
        } else {
          fallbackPreview.value = val;
        }
      },
    });

    const inputVal = computed({
      get: () => {
        return props.value;
      },
      set: (val) => {
        context.emit("input", val);
      },
    });
    return {
      previewState,
      inputVal,
    };
  },
});
</script>
