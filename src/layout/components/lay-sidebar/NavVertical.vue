<script setup lang="ts">
import { useRoute } from "vue-router";
import { emitter } from "@/utils/mitt";
import { useNav } from "@/layout/hooks/useNav";
import { responsiveStorageNameSpace } from "@/config";
import { storageLocal, isAllEmpty } from "@pureadmin/utils";
import { findRouteByPath, getParentPaths } from "@/router/utils";
import { usePermissionStoreHook } from "@/store/modules/permission";
import { ref, computed, watch, onMounted, onBeforeUnmount } from "vue";
import LaySidebarLogo from "../lay-sidebar/components/SidebarLogo.vue";
import LaySidebarItem from "../lay-sidebar/components/SidebarItem.vue";
import LaySidebarLeftCollapse from "../lay-sidebar/components/SidebarLeftCollapse.vue";
import LaySidebarCenterCollapse from "../lay-sidebar/components/SidebarCenterCollapse.vue";

const route = useRoute();
const isShow = ref(false);
const showLogo = ref(
  storageLocal().getItem<StorageConfigs>(
    `${responsiveStorageNameSpace()}configure`
  )?.showLogo ?? true
);

const {
  device,
  pureApp,
  isCollapse,
  tooltipEffect,
  menuSelect,
  toggleSideBar
} = useNav();

const subMenuData = ref([]);

const menuData = computed(() => {
  return pureApp.layout === "mix" && device.value !== "mobile"
    ? subMenuData.value
    : usePermissionStoreHook().wholeMenus;
});

const loading = computed(() =>
  pureApp.layout === "mix" ? false : menuData.value.length === 0 ? true : false
);

const defaultActive = computed(() =>
  !isAllEmpty(route.meta?.activePath) ? route.meta.activePath : route.path
);

function getSubMenuData() {
  let path = "";
  path = defaultActive.value;
  subMenuData.value = [];
  // path的上级路由组成的数组
  const parentPathArr = getParentPaths(
    path,
    usePermissionStoreHook().wholeMenus
  );
  // 当前路由的父级路由信息
  const parenetRoute = findRouteByPath(
    parentPathArr[0] || path,
    usePermissionStoreHook().wholeMenus
  );
  if (!parenetRoute?.children) return;
  subMenuData.value = parenetRoute?.children;
}

watch(
  () => [route.path, usePermissionStoreHook().wholeMenus],
  () => {
    if (route.path.includes("/redirect")) return;
    getSubMenuData();
    menuSelect(route.path);
  }
);

onMounted(() => {
  getSubMenuData();

  emitter.on("logoChange", key => {
    showLogo.value = key;
  });
});

onBeforeUnmount(() => {
  // 解绑`logoChange`公共事件，防止多次触发
  emitter.off("logoChange");
});
</script>

<template>
  <div
    v-loading="loading"
    :class="[
      'sidebar-container',
      showLogo ? 'has-logo' : 'no-logo',
      device === 'mobile' ? 'mobile-layout' : ''
    ]"
    @mouseenter.prevent="isShow = true"
    @mouseleave.prevent="isShow = false"
  >
    <LaySidebarLogo v-if="showLogo" :collapse="isCollapse" />

    <div class="sidebar-content">
      <el-scrollbar
        wrap-class="scrollbar-wrapper"
        :class="[device === 'mobile' ? 'mobile' : 'pc']"
      >
        <el-menu
          unique-opened
          mode="vertical"
          popper-class="pure-scrollbar"
          class="outer-most select-none"
          :collapse="isCollapse"
          :collapse-transition="false"
          :popper-effect="tooltipEffect"
          :default-active="defaultActive"
        >
          <LaySidebarItem
            v-for="routes in menuData"
            :key="routes.path"
            :item="routes"
            :base-path="routes.path"
            class="outer-most select-none"
          />
        </el-menu>
      </el-scrollbar>

      <!-- 固定在底部的升级套餐区域 -->
      <div
        class="absolute bottom-10 left-0 right-0 p-3 bg-gradient-to-r from-indigo-50 to-white border-t border-gray-200 transition-all duration-300 z-10"
      >
        <div class="flex items-center gap-2 w-full">
          <div
            class="flex-shrink-0 flex items-center justify-center w-9 h-9 rounded-md bg-indigo-100 text-indigo-600"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="w-5 h-5"
              viewBox="0 0 24 24"
            >
              <path
                fill="currentColor"
                d="M20.59 13.41l-7.17 7.17a2 2 0 0 1-2.83 0L2 12V2h10l8.59 8.59a2 2 0 0 1 0 2.82z"
              />
              <circle cx="11.5" cy="7.5" r="2.5" fill="currentColor" />
            </svg>
          </div>
          <div class="flex-grow min-w-0">
            <div class="text-sm font-semibold text-gray-800 truncate">
              升级套餐
            </div>
            <div class="text-xs text-gray-500 truncate">解锁更多功能</div>
          </div>
        </div>
      </div>
    </div>

    <!-- 折叠按钮现在位于升级区域下方 -->
    <LaySidebarLeftCollapse
      v-if="device !== 'mobile'"
      :is-active="pureApp.sidebar.opened"
      @toggleClick="toggleSideBar"
    />

    <!-- <LaySidebarCenterCollapse
      v-if="device !== 'mobile' && (isShow || isCollapse)"
      :is-active="pureApp.sidebar.opened"
      @toggleClick="toggleSideBar"
    /> -->
  </div>
</template>

<style scoped>
:deep(.el-loading-mask) {
  opacity: 0.45;
}
</style>
