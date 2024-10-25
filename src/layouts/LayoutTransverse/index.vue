<!-- 横向布局 -->
<template>
  <el-container class="layout">
    <el-header>
      <div class="logo flx-center">
        <img ref="logoImg" :class="['logo-img', { roll: shouldRoll }]" src="@/assets/images/logo.svg" alt="logo" />
        <div ref="logoText" :class="['logo-text', { 'roll-box': shouldRoll }]" class="logo-text">
          <div class="text">{{ title }}</div>
        </div>
      </div>
      <el-menu mode="horizontal" :router="false" :default-active="activeMenu">
        <!-- 不能直接使用 SubMenu 组件，无法触发 el-menu 隐藏省略功能 -->
        <template v-for="subItem in menuList" :key="subItem.path">
          <el-sub-menu v-if="subItem.children?.length" :key="subItem.path" :index="subItem.path + 'el-sub-menu'">
            <template #title>
              <el-icon>
                <component :is="subItem.meta.icon"></component>
              </el-icon>
              <span>{{ subItem.meta.title }}</span>
            </template>
            <SubMenu :menu-list="subItem.children" />
          </el-sub-menu>
          <el-menu-item v-else :key="subItem.path + 'el-menu-item'" :index="subItem.path" @click="handleClickMenu(subItem)">
            <el-icon>
              <component :is="subItem.meta.icon"></component>
            </el-icon>
            <template #title>
              <span>{{ subItem.meta.title }}</span>
            </template>
          </el-menu-item>
        </template>
      </el-menu>
      <ToolBarRight />
    </el-header>
    <Main />
  </el-container>
</template>

<script setup lang="ts" name="layoutTransverse">
import { computed, onMounted, ref } from "vue";
import { useAuthStore } from "@/stores/modules/auth";
import { useRoute, useRouter } from "vue-router";
import Main from "@/layouts/components/Main/index.vue";
import ToolBarRight from "@/layouts/components/Header/ToolBarRight.vue";
import SubMenu from "@/layouts/components/Menu/SubMenu.vue";

const title = import.meta.env.VITE_GLOB_APP_TITLE;

const route = useRoute();
const router = useRouter();
const authStore = useAuthStore();
const menuList = computed(() => authStore.showMenuListGet);
const activeMenu = computed(() => (route.meta.activeMenu ? route.meta.activeMenu : route.path) as string);

const handleClickMenu = (subItem: Menu.MenuOptions) => {
  if (subItem.meta.isLink) return window.open(subItem.meta.isLink, "_blank");
  router.push(subItem.path);
};

const logoImg = ref<HTMLElement | null>(null);
const logoText = ref<HTMLElement | null>(null);
const shouldRoll = ref(false);

onMounted(() => {
  if (logoImg.value && logoText.value) {
    const parentWidth = logoText.value.parentElement?.offsetWidth || 0;
    const imgWidth = logoImg.value.offsetWidth;
    const textWidth = logoText.value.offsetWidth;
    shouldRoll.value = textWidth > parentWidth - imgWidth;
  }
});
</script>

<style scoped lang="scss">
@import "./index.scss";
.logo-img {
  &.roll {
    margin-left: 6px;
  }
}

.roll-box {
  height: 55px;
  // border: 1px solid #ccc;
  border-radius: 6px;
  margin: 50px auto;
  width: calc(100% - 28px - 6px);
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
