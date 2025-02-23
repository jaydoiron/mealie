<template>
  <v-data-table
    v-model="selected"
    item-key="id"
    show-select
    :headers="headers"
    :items="recipes"
    :items-per-page="15"
    class="elevation-0"
    :loading="loading"
    @input="setValue(selected)"
  >
    <template #body.preappend>
      <tr>
        <td></td>
        <td>Hello</td>
        <td colspan="4"></td>
      </tr>
    </template>
    <template #item.tags="{ item }">
      <RecipeChip small :items="item.tags" :is-category="false" url-prefix="tags" />
    </template>
    <template #item.recipeCategory="{ item }">
      <RecipeChip small :items="item.recipeCategory" />
    </template>
    <template #item.tools="{ item }">
      <RecipeChip small :items="item.tools" url-prefix="tools" />
    </template>
    <template #item.userId="{ item }">
      <v-list-item class="justify-start">
        <v-list-item-avatar>
          <img src="https://i.pravatar.cc/300" alt="John" />
        </v-list-item-avatar>
        <v-list-item-content>
          <v-list-item-title>
            {{ getMember(item.userId) }}
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </template>
  </v-data-table>
</template>

<script lang="ts">
import { computed, defineComponent, onMounted, ref } from "@nuxtjs/composition-api";
import RecipeChip from "./RecipeChips.vue";
import { Recipe } from "~/types/api-types/recipe";
import { useUserApi } from "~/composables/api";
import { UserOut } from "~/types/api-types/user";

const INPUT_EVENT = "input";

interface ShowHeaders {
  id: boolean;
  owner: boolean;
  tags: boolean;
  categories: boolean;
  tools: boolean;
  recipeYield: boolean;
  dateAdded: boolean;
}

export default defineComponent({
  components: { RecipeChip },
  props: {
    value: {
      type: Array,
      required: false,
      default: () => [],
    },
    loading: {
      type: Boolean,
      required: false,
      default: false,
    },
    recipes: {
      type: Array as () => Recipe[],
      default: () => [],
    },
    showHeaders: {
      type: Object as () => ShowHeaders,
      required: false,
      default: () => {
        return {
          id: true,
          owner: false,
          tags: true,
          categories: true,
          recipeYield: true,
          dateAdded: true,
        };
      },
    },
  },
  setup(props, context) {
    function setValue(value: Recipe[]) {
      context.emit(INPUT_EVENT, value);
    }

    const headers = computed(() => {
      const hdrs = [];

      if (props.showHeaders.id) {
        hdrs.push({ text: "Id", value: "id" });
      }
      if (props.showHeaders.owner) {
        hdrs.push({ text: "Owner", value: "userId", align: "center" });
      }
      hdrs.push({ text: "Name", value: "name" });
      if (props.showHeaders.categories) {
        hdrs.push({ text: "Categories", value: "recipeCategory" });
      }

      if (props.showHeaders.tags) {
        hdrs.push({ text: "Tags", value: "tags" });
      }
      if (props.showHeaders.tools) {
        hdrs.push({ text: "Tools", value: "tools" });
      }
      if (props.showHeaders.recipeYield) {
        hdrs.push({ text: "Yield", value: "recipeYield" });
      }
      if (props.showHeaders.dateAdded) {
        hdrs.push({ text: "Date Added", value: "dateAdded" });
      }

      return hdrs;
    });

    // ============
    // Group Members
    const api = useUserApi();
    const members = ref<UserOut[]>([]);

    async function refreshMembers() {
      const { data } = await api.groups.fetchMembers();
      if (data) {
        members.value = data;
      }
    }

    onMounted(() => {
      refreshMembers();
    });

    function getMember(id: string) {
      if (members.value[0]) {
        return members.value.find((m) => m.id === id)?.username;
      }

      return "None";
    }

    return { setValue, headers, members, getMember };
  },

  data() {
    return {
      selected: [],
    };
  },
  watch: {
    value(val) {
      this.selected = val;
    },
  },
});
</script>
