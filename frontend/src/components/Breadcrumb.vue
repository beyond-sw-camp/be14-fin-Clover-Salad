<template>
  <div class="breadcrumb">
    <span class="item" @click="goHome">홈</span>
    <template v-if="matchedItem">
      <span class="divider">›</span>
      <span class="item" v-if="parentGroup">{{ parentGroup }}</span>
      <span class="divider">›</span>
      <span class="item active">{{ matchedItem.label }}</span>
    </template>
  </div>
</template>

<script setup>
import { useRoute, useRouter } from "vue-router";
import { menuList, tabOnlyMenuItems } from "@/config/menuConfig";
import { computed } from "vue";

const route = useRoute();
const router = useRouter();

const allItems = [...menuList.flatMap((group) => group.items), ...tabOnlyMenuItems];
const matchedItem = computed(() =>
  allItems.find((item) => route.path.startsWith(item.path))
);

const parentGroup = computed(() => {
  const group = menuList.find((g) =>
    g.items.some((item) => route.path.startsWith(item.path))
  );
  if (group) return group.group;
  
  const tabOnlyItem = tabOnlyMenuItems.find(item => route.path.startsWith(item.path));
  return tabOnlyItem?.meta?.group || "";
});

const goHome = () => {
  router.push("/");
};
</script>

<style scoped>
.breadcrumb {
  font-size: 14px;
  color: #555;
  display: flex;
  align-items: center;
  gap: 8px;
}
.item {
  cursor: default;
}
.item.active {
  font-weight: bold;
  color: #111;
}
.item:hover {
  cursor: pointer;
  text-decoration: underline;
}
.divider {
  color: #aaa;
}
</style>
