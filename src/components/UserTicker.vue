<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[生成成功]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  { text: '用户 138****9102 成功生成了《高中语文古诗词赏析》公开课优质教案' },
  { text: '用户 159****8231 成功合成了《初中数学二次函数》课堂教学反思与改进方案' },
  { text: '用户 186****3394 成功生成了《小学班主任期末德育工作总结与家校沟通记录》' },
  { text: '用户 135****6712 成功撰写了《基于新课标的跨学科教学研究》微课题申报书' },
  { text: '用户 177****4509 成功优化了《职业院校实训课课堂互动与随堂评价表》' },
]);
</script>
