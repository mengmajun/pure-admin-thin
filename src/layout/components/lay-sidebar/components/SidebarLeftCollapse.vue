<script setup lang="ts">
import { computed } from "vue";
import { useGlobal } from "@pureadmin/utils";
import { useNav } from "@/layout/hooks/useNav";

import MenuFold from "~icons/ri/menu-fold-fill";

interface Props {
  isActive?: boolean;
}

withDefaults(defineProps<Props>(), {
  isActive: false
});

const { tooltipEffect } = useNav();

const iconClass = computed(() => {
  return [
    "ml-4",
    "mb-1",
    "w-[16px]",
    "h-[16px]",
    "inline-block!",
    "align-middle",
    "cursor-pointer",
    "duration-[100ms]"
  ];
});

const { $storage } = useGlobal<GlobalPropertiesApi>();
const themeColor = computed(() => $storage.layout?.themeColor);

const emit = defineEmits<{
  (e: "toggleClick"): void;
}>();

const toggleClick = () => {
  emit("toggleClick");
};
</script>

<template>
  <div class="left-collapse">
    <IconifyIconOffline
      v-tippy="{
        content: isActive ? '点击折叠' : '点击展开',
        theme: tooltipEffect,
        hideOnClick: 'toggle',
        placement: 'right'
      }"
      :icon="MenuFold"
      :class="[iconClass, themeColor === 'light' ? '' : 'text-primary']"
      :style="{ transform: isActive ? 'none' : 'rotateY(180deg)' }"
      @click="toggleClick"
    />
  </div>
</template>

<style lang="scss" scoped>
.left-collapse {
  position: absolute;
  bottom: 0;
  display: flex;
  justify-content: flex-end;
  width: 100%;
  height: 40px;
  padding-top: 12px;
  padding-right: 16px; // 添加右边距
  line-height: 40px;
  box-shadow: 0 0 6px -3px var(--el-color-primary);
}
</style>
