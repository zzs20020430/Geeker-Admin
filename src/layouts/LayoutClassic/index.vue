<!-- 经典布局 -->
<template>
  <el-container class="layout">
    <el-header>
      <div class="header-lf mask-image">
        <div class="logo flx-center">
          <img ref="logoImg" :class="['logo-img', { roll: shouldRoll }]" src="@/assets/images/logo.svg" alt="logo" />
          <div ref="logoText" :class="['logo-text', { 'roll-box': shouldRoll }]" class="logo-text">
            <div class="text">{{ title }}</div>
          </div>
        </div>
        <ToolBarLeft />
      </div>
      <div class="header-ri">
        <ToolBarRight />
      </div>
    </el-header>
    <el-container class="classic-content">
      <el-aside>
        <div class="aside-box" :style="{ width: isCollapse ? '65px' : '210px' }">
          <el-scrollbar>
            <el-menu
              :router="false"
              :default-active="activeMenu"
              :collapse="isCollapse"
              :unique-opened="accordion"
              :collapse-transition="false"
            >
              <SubMenu :menu-list="menuList" />
            </el-menu>
          </el-scrollbar>
        </div>
      </el-aside>
      <el-container class="classic-main">
        <Main />
      </el-container>
    </el-container>
  </el-container>
</template>

<script setup lang="ts" name="layoutClassic">
import { computed, onMounted, ref } from "vue";
import { useRoute } from "vue-router";
import { useAuthStore } from "@/stores/modules/auth";
import { useGlobalStore } from "@/stores/modules/global";
import Main from "@/layouts/components/Main/index.vue";
import SubMenu from "@/layouts/components/Menu/SubMenu.vue";
import ToolBarLeft from "@/layouts/components/Header/ToolBarLeft.vue";
import ToolBarRight from "@/layouts/components/Header/ToolBarRight.vue";

const title = import.meta.env.VITE_GLOB_APP_TITLE;

const route = useRoute();
const authStore = useAuthStore();
const globalStore = useGlobalStore();
const accordion = computed(() => globalStore.accordion);
const isCollapse = computed(() => globalStore.isCollapse);
const menuList = computed(() => authStore.showMenuListGet);
const activeMenu = computed(() => (route.meta.activeMenu ? route.meta.activeMenu : route.path) as string);

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
