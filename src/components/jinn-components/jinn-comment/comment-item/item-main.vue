<template>
  <!-- 评论内容 -->
  <div class="item-main" v-html="replaceCustomString(content)"></div>
</template>

<script setup lang="ts">
import { replaceCustomString } from '@/utils/FileUtils/EmotionFile';

const props = defineProps<{
  content: string; // 评论内容
}>();
</script>

<style lang="scss" scoped>
  .item-main {
    color: #000000;
    font-size: 14.5px;
    width: calc(100% - 40px);
    word-break: break-all;
  }

</style>
