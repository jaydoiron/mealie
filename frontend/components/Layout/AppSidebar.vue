<template>
  <v-navigation-drawer v-model="drawer" class="d-flex flex-column d-print-none" clipped app width="240px">
    <!-- User Profile -->
    <template v-if="$auth.user">
      <v-list-item two-line to="/user/profile" exact>
        <UserAvatar list :user-id="$auth.user.id" />

        <v-list-item-content>
          <v-list-item-title> {{ $auth.user.fullName }}</v-list-item-title>
          <v-list-item-subtitle>
            <NuxtLink class="favorites-link" :to="`/user/${$auth.user.id}/favorites`"> Favorite Recipes </NuxtLink>
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
      <v-divider></v-divider>
    </template>

    <slot></slot>

    <!-- Primary Links -->
    <template v-if="topLink">
      <v-list nav dense>
        <template v-for="nav in topLink">
          <!-- Multi Items -->
          <v-list-group
            v-if="nav.children && ($auth.loggedIn || !nav.restricted)"
            :key="nav.title + 'multi-item'"
            v-model="dropDowns[nav.title]"
            color="primary"
            :prepend-icon="nav.icon"
          >
            <template #activator>
              <v-list-item-title>{{ nav.title }}</v-list-item-title>
            </template>

            <v-list-item v-for="child in nav.children" :key="child.title" exact :to="child.to">
              <v-list-item-icon>
                <v-icon>{{ child.icon }}</v-icon>
              </v-list-item-icon>
              <v-list-item-title>{{ child.title }}</v-list-item-title>
            </v-list-item>
            <v-divider class="mb-4"></v-divider>
          </v-list-group>

          <!-- Single Item -->
          <v-list-item-group
            v-else-if="$auth.loggedIn || !nav.restricted"
            :key="nav.title + 'single-item'"
            v-model="secondarySelected"
            color="primary"
          >
            <v-list-item exact link :to="nav.to">
              <v-list-item-icon>
                <v-icon>{{ nav.icon }}</v-icon>
              </v-list-item-icon>
              <v-list-item-title>{{ nav.title }}</v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </template>
      </v-list>
    </template>

    <!-- Secondary Links -->
    <template v-if="secondaryLinks">
      <v-subheader v-if="secondaryHeader" :to="secondaryHeaderLink" class="pb-0">
        {{ secondaryHeader }}
      </v-subheader>
      <v-divider></v-divider>
      <v-list nav dense exact>
        <template v-for="nav in secondaryLinks">
          <!-- Multi Items -->
          <v-list-group
            v-if="nav.children"
            :key="nav.title + 'multi-item'"
            v-model="dropDowns[nav.title]"
            color="primary"
            :prepend-icon="nav.icon"
          >
            <template #activator>
              <v-list-item-title>{{ nav.title }}</v-list-item-title>
            </template>

            <v-list-item v-for="child in nav.children" :key="child.title" exact :to="child.to">
              <v-list-item-icon>
                <v-icon>{{ child.icon }}</v-icon>
              </v-list-item-icon>
              <v-list-item-title>{{ child.title }}</v-list-item-title>
            </v-list-item>
            <v-divider class="mb-4"></v-divider>
          </v-list-group>

          <!-- Single Item -->
          <v-list-item-group v-else :key="nav.title + 'single-item'" v-model="secondarySelected" color="primary">
            <v-list-item exact link :to="nav.to">
              <v-list-item-icon>
                <v-icon>{{ nav.icon }}</v-icon>
              </v-list-item-icon>
              <v-list-item-title>{{ nav.title }}</v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </template>
      </v-list>
    </template>

    <!-- Bottom Navigation Links -->
    <template v-if="bottomLinks" #append>
      <v-list nav dense>
        <v-list-item-group v-model="bottomSelected" color="primary">
          <template v-for="nav in bottomLinks">
            <v-list-item
              v-if="!nav.restricted || $auth.loggedIn"
              :key="nav.title"
              exact
              link
              :to="nav.to || null"
              :href="nav.href || null"
              :target="nav.href ? '_blank' : null"
            >
              <v-list-item-icon>
                <v-icon>{{ nav.icon }}</v-icon>
              </v-list-item-icon>
              <v-list-item-title>{{ nav.title }}</v-list-item-title>
            </v-list-item>
          </template>
        </v-list-item-group>
        <slot name="bottom"></slot>
      </v-list>
    </template>
  </v-navigation-drawer>
</template>

<script lang="ts">
import { computed, defineComponent, reactive, toRefs } from "@nuxtjs/composition-api";
import { SidebarLinks } from "~/types/application-types";
import UserAvatar from "~/components/Domain/User/UserAvatar.vue";

export default defineComponent({
  components: {
    UserAvatar,
  },
  props: {
    value: {
      type: Boolean,
      default: null,
    },
    user: {
      type: Object,
      default: null,
    },
    topLink: {
      type: Array as () => SidebarLinks,
      required: true,
    },
    secondaryLinks: {
      type: Array as () => SidebarLinks,
      required: false,
      default: null,
    },
    bottomLinks: {
      type: Array as () => SidebarLinks,
      required: false,
      default: null,
    },
    secondaryHeader: {
      type: String,
      default: null,
    },
    secondaryHeaderLink: {
      type: String,
      default: null,
    },
  },
  setup(props, context) {
    // V-Model Support
    const drawer = computed({
      get: () => {
        return props.value;
      },
      set: (val) => {
        context.emit("input", val);
      },
    });

    const state = reactive({
      dropDowns: {},
      topSelected: null as string[] | null,
      secondarySelected: null as string[] | null,
      bottomSelected: null as string[] | null,
    });

    return {
      ...toRefs(state),
      drawer,
    };
  },
});
</script>

<style>
@media print {
  .no-print {
    display: none;
  }
}

.favorites-link {
  text-decoration: none;
}

.favorites-link:hover {
  text-decoration: underline;
}
</style>
