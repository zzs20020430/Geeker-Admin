<!-- 分栏布局 -->
<template>
  <el-container class="layout">
    <div class="aside-split">
      <div class="logo flx-center">
        <img class="logo-img" src="@/assets/images/logo.svg" alt="logo" />
      </div>
      <el-scrollbar>
        <div class="split-list">
          <div
            v-for="item in menuList"
            :key="item.path"
            class="split-item"
            :class="{ 'split-active': splitActive === item.path || `/${splitActive.split('/')[1]}` === item.path }"
            @click="changeSubMenu(item)"
          >
            <el-icon>
              <component :is="item.meta.icon"></component>
            </el-icon>
            <span class="title">{{ item.meta.title }}</span>
          </div>
        </div>
      </el-scrollbar>
    </div>
    <el-aside :class="{ 'not-aside': !subMenuList.length }" :style="{ width: isCollapse ? '65px' : '210px' }">
      <div class="logo flx-center">
        <!-- <span v-show="subMenuList.length" class="logo-text">{{ isCollapse ? "G" : title }}</span> -->
        <div
          v-show="subMenuList.length"
          ref="logoText"
          :class="['logo-text', { 'roll-box': shouldRoll && !isCollapse }]"
          class="logo-text"
        >
          <div class="text">{{ isCollapse ? title[0] : title }}</div>
        </div>
      </div>
      <el-scrollbar>
        <el-menu
          :router="false"
          :default-active="activeMenu"
          :collapse="isCollapse"
          :unique-opened="accordion"
          :collapse-transition="false"
        >
          <SubMenu :menu-list="subMenuList" />
        </el-menu>
      </el-scrollbar>
    </el-aside>
    <el-container>
      <el-header>
        <ToolBarLeft />
        <ToolBarRight />
      </el-header>
      <Main />
    </el-container>
  </el-container>
</template>

<script setup lang="ts" name="layoutColumns">
import { ref, computed, watch, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useAuthStore } from "@/stores/modules/auth";
import { useGlobalStore } from "@/stores/modules/global";
import Main from "@/layouts/components/Main/index.vue";
import ToolBarLeft from "@/layouts/components/Header/ToolBarLeft.vue";
import ToolBarRight from "@/layouts/components/Header/ToolBarRight.vue";
import SubMenu from "@/layouts/components/Menu/SubMenu.vue";

const title = import.meta.env.VITE_GLOB_APP_TITLE;

const route = useRoute();
const router = useRouter();
const authStore = useAuthStore();
const globalStore = useGlobalStore();
const accordion = computed(() => globalStore.accordion);
const isCollapse = computed(() => globalStore.isCollapse);
const menuList = computed(() => authStore.showMenuListGet);
const activeMenu = computed(() => (route.meta.activeMenu ? route.meta.activeMenu : route.path) as string);

const subMenuList = ref<Menu.MenuOptions[]>([]);
const splitActive = ref("");
watch(
  () => [menuList, route],
  () => {
    // 当前菜单没有数据直接 return
    if (!menuList.value.length) return;
    splitActive.value = route.path;
    const menuItem = menuList.value.filter((item: Menu.MenuOptions) => {
      return route.path === item.path || `/${route.path.split("/")[1]}` === item.path;
    });
    if (menuItem[0].children?.length) return (subMenuList.value = menuItem[0].children);
    subMenuList.value = [];
  },
  {
    deep: true,
    immediate: true
  }
);

// change SubMenu
const changeSubMenu = (item: Menu.MenuOptions) => {
  splitActive.value = item.path;
  if (item.children?.length) return (subMenuList.value = item.children);
  subMenuList.value = [];
  router.push(item.path);
};

const logoText = ref<HTMLElement | null>(null);
const shouldRoll = ref(false);

onMounted(() => {
  if (logoText.value) {
    const parentWidth = 210; //logoText.value.parentElement?.offsetWidth || 0;
    const textWidth = logoText.value.offsetWidth;
    shouldRoll.value = textWidth > parentWidth;
  }
});
</script>

<style scoped lang="scss">
@import "./index.scss";

.roll-box {
  height: 55px;
  // border: 1px solid #ccc;
  border-radius: 6px;
  margin: 50px auto;
  width: calc(100%);
  position: relative;
  line-height: 55px;
  overflow: hidden;
  // font-size: 16px;
  &:hover {
    .text {
      animation-play-state: paused;
    }
  }
  &::before {
    content: "";
    position: absolute;
    width: 10px;
    height: 100%;
    overflow-x: hidden;
    overflow-y: hidden;
    left: 0px;
    box-shadow: inset 10px 0px 10px -10px rgba(0, 0, 0, 0.15);
  }

  &::after {
    content: "";
    position: absolute;
    width: 10px;
    height: 100%;
    overflow-x: hidden;
    overflow-y: hidden;
    right: 0px;
    box-shadow: inset -10px 0px 10px -10px rgba(0, 0, 0, 0.15);
  }

  .text {
    position: absolute;
    white-space: nowrap;
    left: 0;
    top: 0;
    animation-name: roll;
    animation-duration: 8s;
    animation-timing-function: linear;
    padding-right: 20px;
    animation-iteration-count: infinite;
    &:before {
      content: "";
      display: inline-block;
      width: 200px;
      height: 100%;
    }
  }
}

@keyframes roll {
  from {
    transform: translate(0%, 0);
  }
  to {
    transform: translate(-100%, 0);
  }
}
</style>
