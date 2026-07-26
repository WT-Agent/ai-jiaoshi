<template>
  <section class="nomads-showcase-section">
    <div class="showcase-header">
      <div class="header-left">
        <h2 class="showcase-title">教师教学与教研实战模板库 (Nomads Showcase)</h2>
        <p class="showcase-subtitle">精选公开课教案、评课反思、班主任总结与课题申报高频场景，点击“一键套用”快速生成</p>
      </div>
      <span class="showcase-badge">已收录 {{ showcaseItems.length }} 个实战模板</span>
    </div>

    <div class="showcase-grid">
      <div 
        v-for="item in showcaseItems" 
        :key="item.id" 
        class="glass-card showcase-card"
      >
        <div class="card-header">
          <span class="scenario-tag">{{ item.tag }}</span>
          <span class="usage-count">{{ item.usageCount }} 次应用</span>
        </div>

        <div class="card-content">
          <h3 class="item-title">{{ item.title }}</h3>
          <p class="item-prompt">“{{ item.prompt }}”</p>
        </div>

        <div class="card-action">
          <button class="apply-btn" @click="applyTemplate(item)">
            <span>一键套用</span>
            <svg class="arrow-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const emit = defineEmits<{
  (e: 'apply-template', payload: { prompt: string; teacherType?: string; schoolStage?: string; subjectCategory?: string }): void;
}>();

export interface ShowcaseItem {
  id: string;
  tag: string;
  title: string;
  prompt: string;
  teacherType?: string;
  schoolStage?: string;
  subjectCategory?: string;
  usageCount: string;
}

const showcaseItems = computed<ShowcaseItem[]>(() => [
  {
    id: 'teach-1',
    tag: '公开课教案',
    title: '语文公开课获奖教学设计',
    prompt: '请为高中语文《荷塘月色》设计一堂展示课教案，包含核心素养定位、情境化导入、细节探究研讨及课后延伸作业。',
    teacherType: '公开课优质教案与教学设计',
    schoolStage: '高中教育',
    subjectCategory: '语文人文类',
    usageCount: '28.5k'
  },
  {
    id: 'teach-2',
    tag: '教学反思',
    title: '课堂互动失效深刻反思',
    prompt: '初中数学《一次函数应用题》课堂提问冷场、学生参与度低，请帮我分析教学设计漏洞并给出深层课后反思与改进策略。',
    teacherType: '课后教学反思与评课记录',
    schoolStage: '初中教育',
    subjectCategory: '数学理科类',
    usageCount: '21.3k'
  },
  {
    id: 'teach-3',
    tag: '德育总结',
    title: '班主任期末工作与德育总结',
    prompt: '请撰写一份小学五年级班主任期末工作总结，涵盖班风学风建设、后进生转化案例、家校共育成果及下学期改进计划。',
    teacherType: '班主任工作总结与德育反思',
    schoolStage: '小学教育',
    subjectCategory: '语文人文类',
    usageCount: '34.8k'
  },
  {
    id: 'teach-4',
    tag: '课题申报',
    title: '教育教学微课题申报方案',
    prompt: '请设计一个关于“双减背景下初中英语单元作业分层设计实效性研究”的微课题申报书，包含课题立项依据、研究内容及预期成果。',
    teacherType: '教学论文与课题开题结题',
    schoolStage: '初中教育',
    subjectCategory: '英语外语类',
    usageCount: '19.2k'
  },
  {
    id: 'teach-5',
    tag: '评课记录',
    title: '专家级公开课评课与观摩记录',
    prompt: '请针对一堂高中物理《牛顿第二定律》优质示范课，撰写一份包含教学亮点、细节瑕疵与重建建议的高质量评课意见。',
    teacherType: '课后教学反思与评课记录',
    schoolStage: '高中教育',
    subjectCategory: '数学理科类',
    usageCount: '16.7k'
  },
  {
    id: 'teach-6',
    tag: '教研论文',
    title: '教学创新论文开题与写作提纲',
    prompt: '请围绕“项目式学习（PBL）在高中化学实验教学中的应用”撰写一份教研论文写作大纲与核心创新点论述。',
    teacherType: '教学论文与课题开题结题',
    schoolStage: '高中教育',
    subjectCategory: '数学理科类',
    usageCount: '25.4k'
  }
]);

function applyTemplate(item: ShowcaseItem) {
  emit('apply-template', {
    prompt: item.prompt,
    teacherType: item.teacherType,
    schoolStage: item.schoolStage,
    subjectCategory: item.subjectCategory
  });
}
</script>

<style scoped>
.nomads-showcase-section {
  margin-top: 2rem;
  width: 100%;
}

.showcase-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 1.25rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--card-border);
}

.showcase-title {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text-primary);
  background: var(--primary-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.showcase-subtitle {
  font-size: 0.825rem;
  color: var(--text-secondary);
  margin-top: 0.25rem;
}

.showcase-badge {
  font-size: 0.75rem;
  color: #a5b4fc;
  background: rgba(99, 102, 241, 0.12);
  border: 1px solid rgba(99, 102, 241, 0.25);
  padding: 4px 10px;
  border-radius: 20px;
}

.showcase-grid {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  gap: 1.25rem;
}

@media (min-width: 640px) {
  .showcase-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .showcase-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.showcase-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
  padding: 1.25rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid var(--card-border);
  border-radius: 14px;
  transition: all 0.25s ease;
}

.showcase-card:hover {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(99, 102, 241, 0.4);
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.scenario-tag {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(168, 85, 247, 0.15);
  color: #c084fc;
  border: 1px solid rgba(168, 85, 247, 0.3);
}

.usage-count {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.card-content {
  margin-bottom: 1rem;
  flex: 1;
}

.item-title {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0.4rem;
}

.item-prompt {
  font-size: 0.825rem;
  color: var(--text-secondary);
  line-height: 1.45;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  font-style: italic;
}

.card-action {
  padding-top: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.04);
}

.apply-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 0.5rem 1rem;
  background: rgba(99, 102, 241, 0.1);
  border: 1px solid rgba(99, 102, 241, 0.3);
  border-radius: 8px;
  color: #a5b4fc;
  font-size: 0.825rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.showcase-card:hover .apply-btn {
  background: var(--primary-gradient);
  border-color: transparent;
  color: white;
}

.arrow-icon {
  width: 14px;
  height: 14px;
  transition: transform 0.2s ease;
}

.apply-btn:hover .arrow-icon {
  transform: translateX(3px);
}
</style>
