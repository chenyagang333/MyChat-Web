<template>
  <div class="navs" :class="headerUp ? 'headerUp' : 'headerDown'">
    <div class="nav" v-for="(i, index) in 4" :key="index">
      <span
        :class="nowNav == index ? 'select-nav' : ''"
        @click="toggleNav(index)"
        >{{ $t(lang + "navs" + i) }}
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useRoute, useRouter } from "vue-router";
import { onMounted, ref } from "vue";

defineProps<{
  headerUp: boolean;
}>();

const router = useRouter();
const route = useRoute();

const lang = "Layout1."; // 基本参数管理

//#region 导航
const nowNav = ref<number>(0);
const navMap = ["/", "create-show"];
const toggleNav = (i: number) => {
  nowNav.value = i;
  router.push(navMap[i]);
};
//#endregion

onMounted(() => {
  console.log("454557878 :>> ", 454557878);
});
</script>

<style lang="scss" scoped>
.navs {
  margin-left: 20px;
  display: flex;
  align-items: center;
  user-select: none;
  .nav {
    margin-right: 20px;
    cursor: pointer;
    height: 35px;
    font-size: 18px;
    line-height: 35px;
    position: relative;
    $nav-color: var(--jinn-text-color1);

    @media (max-width: 1450px) {
      margin-right: 10px;
    }
    &::after {
      transition: width 0.1s;
      position: absolute;
      content: "";
      bottom: 0;
      left: 0;
      height: 2px;
      width: 0%;
      background-color: $nav-color;
    }
    &:hover {
      color: $nav-color;
    }
    &:hover::after {
      width: 100%;
    }
    .select-nav {
      color: $nav-color;
    }
  }
}
.headerUp {
  .nav {
    color: white;
  }
}
.headerDown {
  .nav {
    color: var(--jinn-color2);
  }
}
</style>
