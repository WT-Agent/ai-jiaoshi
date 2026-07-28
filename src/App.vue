<template>
  <div class="app-container">
    <!-- 成功提示 -->
    <div v-if="copied" class="top-success-toast">
      复制成功
    </div>
    <!-- 常驻悬浮分享按钮 (H5 / 移动端与桌面端通用) -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg class="share-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享教研神器</span>
    </button>

    <header>
      <h1>{{ appTitle }}</h1>
      <p>新课标教学设计 · 课堂教学反思 · 班主任德育总结 · 教育科研课题申报</p>
    </header>

    <!-- 动态广播轮播 -->
    <UserTicker />

    <!-- 核心操作区卡片 -->
    <main ref="inputCardRef" class="glass-card input-group">
      <!-- 4 种预设类型选择 -->
      <div class="selector-group">
        <label class="selector-label">选择教研类型</label>
        <div class="style-selector">
          <button 
            v-for="ttype in teacherTypeOptions" 
            :key="ttype"
            class="style-option"
            :class="{ active: activeTeacherType === ttype }"
            @click="activeTeacherType = ttype"
          >
            {{ ttype }}
          </button>
        </div>
      </div>

      <!-- 2 组属性: 教学学段 & 学科门类 -->
      <div class="options-row" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem;">
        <div class="selector-group">
          <label class="selector-label">教学学段</label>
          <div class="style-selector">
            <button 
              v-for="stage in schoolStageOptions" 
              :key="stage"
              class="style-option"
              :class="{ active: selectedSchoolStage === stage }"
              @click="selectedSchoolStage = stage"
            >
              {{ stage }}
            </button>
          </div>
        </div>

        <div class="selector-group">
          <label class="selector-label">学科门类</label>
          <div class="style-selector">
            <button 
              v-for="subject in subjectCategoryOptions" 
              :key="subject"
              class="style-option"
              :class="{ active: selectedSubjectCategory === subject }"
              @click="selectedSubjectCategory = subject"
            >
              {{ subject }}
            </button>
          </div>
        </div>
      </div>

      <!-- 输入框 -->
      <div class="selector-group">
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <label class="selector-label">输入教学主题、课文题目、反思难点或课题方向</label>
          <div style="display: flex; gap: 0.5rem;">
            <button v-if="userInput" class="text-link-btn" @click="userInput = ''">清空输入</button>
            <button class="text-link-btn" @click="showTeacherGuideModal = true">新课标设计指南</button>
          </div>
        </div>
        <textarea 
          v-model="userInput" 
          placeholder="请输入您的教学主题、课文题目、反思难点或课题方向...（例如：初中语文《背影》公开课教案，重点突出情境教学与细节描写赏析，学段为初中教育。）"
          style="min-height: 120px;"
        ></textarea>
        <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-secondary);">
          <span>字符数: {{ userInput.length }} 字</span>
          <span>建议明确课文题目、学情特征、教学目标或教研反思痛点</span>
        </div>
      </div>

      <!-- 操作按钮区 -->
      <div style="display: flex; gap: 0.75rem;">
        <button 
          class="action-btn" 
          :disabled="loading || !userInput.trim()"
          @click="handleGenerate"
        >
          {{ loading ? '正在精准分析教学目标与生成方案中...' : '开始生成教师教学与教研方案' }}
        </button>
        <button class="icon-btn" style="padding: 0 1rem; border-radius: 10px;" @click="toggleHistoryDrawer">
          历史方案 ({{ historyList.length }})
        </button>
      </div>

      <!-- 异常提示 -->
      <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
        {{ errorMsg }}
      </div>
    </main>

    <!-- 生成结果卡片 -->
    <section v-if="result || loading" class="glass-card">
      <div class="result-header">
        <span class="result-title">教师教学与教研反思方案</span>
        <div class="button-actions">
          <button v-if="result" class="icon-btn" @click="copyText">
            {{ copied ? '已复制方案' : '复制教学方案' }}
          </button>
          <button v-if="result" class="icon-btn" @click="resetResult">
            重置
          </button>
        </div>
      </div>

      <!-- 加载中骨架屏 -->
      <div v-if="loading" class="skeleton">
        <div class="skeleton-line" style="width: 85%"></div>
        <div class="skeleton-line" style="width: 95%"></div>
        <div class="skeleton-line" style="width: 70%"></div>
        <div class="skeleton-line" style="width: 90%"></div>
        <div class="skeleton-line" style="width: 60%"></div>
      </div>

      <!-- 渲染结果 -->
      <div v-else-if="result">
        <!-- AI 共识打分可视化看板 -->
        <div v-if="aiScores" class="scores-container" style="margin-bottom: 1.5rem; padding: 1.25rem; background: rgba(0,0,0,0.25); border-radius: 12px; border: 1px solid rgba(255,255,255,0.06);">
          <div style="font-weight: 700; font-size: 0.95rem; margin-bottom: 1rem; color: #a5b4fc; display: flex; justify-content: space-between; align-items: center;">
            <span>AI 教学法理与课堂设计评估看板</span>
            <span style="font-size: 0.8rem; font-weight: normal; color: var(--text-secondary);">综合教研质量分: {{ getAverageScoreFromMap(aiScores) }} / 100</span>
          </div>
          <div class="metrics-grid" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1rem;">
            <div v-for="metric in metricsList" :key="metric.key" class="metric-item">
              <div style="display: flex; justify-content: space-between; font-size: 0.8rem; margin-bottom: 0.3rem;">
                <span style="color: var(--text-secondary);">{{ metric.label }}</span>
                <span style="font-weight: bold; color: var(--accent-color);">{{ aiScores[metric.key] || 90 }} 分</span>
              </div>
              <div class="bar-bg" style="height: 6px; background: rgba(255,255,255,0.08); border-radius: 3px; overflow: hidden;">
                <div class="bar-fill" :style="{ width: (aiScores[metric.key] || 90) + '%', background: 'var(--primary-gradient)', height: '100%', borderRadius: '3px', transition: 'width 0.5s ease' }"></div>
              </div>
            </div>
          </div>
        </div>

        <div class="output-content">{{ displayResultText }}</div>
      </div>
    </section>

    <!-- 历史记录面板 -->
    <section v-if="showHistory" class="glass-card" style="margin-top: 1rem;">
      <div class="result-header">
        <span class="result-title">本地教研方案历史记录</span>
        <button class="icon-btn" @click="showHistory = false">关闭记录</button>
      </div>

      <div v-if="historyList.length === 0" style="text-align: center; color: var(--text-secondary); padding: 1.5rem; font-size: 0.85rem;">
        暂无历史教研记录，立即开始生成教学方案吧！
      </div>

      <div v-else class="history-grid" style="display: flex; flex-direction: column; gap: 0.75rem; max-height: 320px; overflow-y: auto;">
        <div v-for="item in historyList" :key="item.id" class="history-item" style="padding: 1rem; background: rgba(0,0,0,0.2); border-radius: 10px; border: 1px solid var(--card-border);">
          <div style="display: flex; justify-content: space-between; font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 0.4rem;">
            <span>{{ item.timestamp }} · [{{ item.teacherType }} / {{ item.schoolStage }} / {{ item.subjectCategory }}]</span>
            <span style="color: var(--primary-color);">评分: {{ getAverageScore(item) }}</span>
          </div>
          <div style="font-size: 0.85rem; font-weight: bold; margin-bottom: 0.4rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; color: var(--text-primary);">
            主题/需求: {{ item.input }}
          </div>
          <div style="display: flex; gap: 0.5rem;">
            <button class="icon-btn" style="font-size: 0.75rem;" @click="applyHistory(item)">套用场景</button>
            <button class="icon-btn" style="font-size: 0.75rem;" @click="viewHistoryOutput(item)">查看方案全文</button>
          </div>
        </div>
      </div>
    </section>

    <!-- 教师教研模版 Showcase -->
    <NomadsShowcase
      @apply-template="handleApplyTemplate"
    />

    <!-- 新课标核心素养与教学设计规范指南 Modal -->
    <div v-if="showTeacherGuideModal" class="modal-overlay" @click.self="showTeacherGuideModal = false">
      <div class="modal-content" style="max-width: 480px;">
        <h3>新课标核心素养与教学设计规范指南</h3>
        <p style="text-align: left; font-size: 0.825rem; margin-bottom: 1rem; color: var(--text-secondary);">
          提升公开课教案质量与教研成果落地的核心准则：
        </p>
        <div class="modal-scroll-area" style="text-align: left; font-size: 0.825rem;">
          <div v-for="(rule, idx) in teacherGuideRules" :key="idx" style="margin-bottom: 0.75rem; padding: 0.5rem 0.75rem; background: rgba(255,255,255,0.03); border-radius: 8px; border: 1px solid rgba(255,255,255,0.05);">
            <div style="color: var(--accent-color); font-weight: bold; margin-bottom: 0.2rem;">{{ rule.title }}</div>
            <div style="color: var(--text-primary); margin-bottom: 0.2rem;">设计要点: {{ rule.advice }}</div>
            <div style="color: var(--text-secondary); font-size: 0.775rem;">避坑提示: {{ rule.avoid }}</div>
          </div>
        </div>
        <button class="modal-btn" style="margin-top: 1rem;" @click="showTeacherGuideModal = false">关闭</button>
      </div>
    </div>

    <!-- 微信 H5 悬浮分享引导 Modal -->
    <div v-if="showShareGuide" class="modal-overlay" @click.self="showShareGuide = false">
      <div class="modal-content">
        <h3>分享教师教学与教研反思专家</h3>
        <p>扫码关注或将链接分享给同行教师与教研员，助力高效备课与课题研究。</p>
        
        <div class="qr-code-placeholder">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" width="100%" height="100%">
            <rect width="100" height="100" fill="white"/>
            <rect x="5" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="9" width="17" height="17" fill="white"/>
            <rect x="13" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="70" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="74" y="9" width="17" height="17" fill="white"/>
            <rect x="78" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="5" y="70" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="74" width="17" height="17" fill="white"/>
            <rect x="13" y="78" width="9" height="9" fill="#110e24"/>
            <rect x="35" y="10" width="8" height="8" fill="#110e24"/>
            <rect x="48" y="5" width="6" height="12" fill="#110e24"/>
            <rect x="60" y="15" width="5" height="5" fill="#110e24"/>
            <rect x="35" y="35" width="10" height="10" fill="#110e24"/>
            <rect x="50" y="45" width="15" height="8" fill="#110e24"/>
            <rect x="40" y="70" width="8" height="16" fill="#110e24"/>
            <rect x="55" y="65" width="10" height="10" fill="#110e24"/>
            <rect x="75" y="40" width="12" height="12" fill="#110e24"/>
            <rect x="75" y="75" width="15" height="15" fill="#110e24"/>
            <rect x="45" y="80" width="8" height="8" fill="#110e24"/>
          </svg>
        </div>

        <div style="font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 1.5rem;">
          微信号: <span style="color: var(--primary-color); font-weight: bold;">{{ wechatId }}</span>
        </div>

        <button class="modal-btn" @click="showShareGuide = false">关闭</button>
      </div>
    </div>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
      <a href="https://api.wuxian.xyz/sign-up?aff=OyRY" target="_blank" rel="noopener noreferrer" class="footer-link-btn">API 平台</a>
      <a href="https://www.kutuyun.com/aff/IPJKCKWF" target="_blank" rel="noopener noreferrer" class="footer-link-btn">酷兔云</a>
      <a href="https://bandwagonhost.com/aff.php?aff=48115" target="_blank" rel="noopener noreferrer" class="footer-link-btn">搬瓦工</a>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的教学教研数据隐私与未公开发表课题保密。您在本应用中输入的教学主题与教研论文提纲仅用于实时大模型生成，系统不会在云端永久存储或泄露您的教研成果。</p>
          <p>为了记录您的免费生成额度，本应用会在您的浏览器本地（localStorage）记录试用次数与解锁状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用网腾无限 AI 教师教学与教研反思专家。本工具生成的教案设计、课堂反思及课题申报提纲仅供教师与教研人员备课参考。</p>
          <p>在应用于教学实践或正式提交课题申报书前，请教师结合具体班级学情、校本要求与最新教育政策进行二次调整与核对。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信交流" class="contact-qr-img" />
              <span class="contact-qr-label">微信交流</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉联系" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉联系</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import NomadsShowcase from './components/NomadsShowcase.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

// 配置参数
const appTitle = ref(appConfig.title || '网腾无限AI - 教师教学与教研反思专家');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '教师教学与教研反思专家');

const inputCardRef = ref<HTMLElement | null>(null);
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);

const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showTeacherGuideModal = ref(false);

// 解析 Cookie
const getCookie = (name: string): string | null => {
  const nameEQ = name + "=";
  const ca = document.cookie.split(';');
  for (let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) === ' ') c = c.substring(1, c.length);
    if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
  }
  return null;
};

// 用户登录状态
const userToken = ref(getCookie('wuxian_session'));
const isLoggedIn = computed(() => !!userToken.value);
const authUsesCount = ref(parseInt(localStorage.getItem('auth_uses') || '0', 10));

// 4 种预设类型
const teacherTypeOptions = [
  '公开课优质教案与教学设计',
  '课后教学反思与评课记录',
  '班主任工作总结与德育反思',
  '教学论文与课题开题结题'
];
const activeTeacherType = ref(teacherTypeOptions[0]);

// 2 组属性: 教学学段 & 学科门类
const schoolStageOptions = ['小学教育', '初中教育', '高中教育', '职业与高等教育'];
const selectedSchoolStage = ref('高中教育');

const subjectCategoryOptions = ['语文人文类', '数学理科类', '英语外语类', '艺体综合类'];
const selectedSubjectCategory = ref('语文人文类');

// 评估指标列表
const metricsList = [
  { key: 'pedagogicalRigor', label: '教学法理严谨度' },
  { key: 'studentCentricity', label: '以生为本导向度' },
  { key: 'curriculumDesignClarity', label: '课堂设计条理性' },
  { key: 'teachingReflectionDepth', label: '教学反思深刻度' },
  { key: 'teacherProfessionalism', label: '教师专业素养值' }
];

const aiScores = ref<Record<string, number> | null>(null);

// 历史记录定义
interface HistoryItem {
  id: string;
  timestamp: string;
  teacherType: string;
  schoolStage: string;
  subjectCategory: string;
  input: string;
  aiScores: Record<string, number> | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

const HISTORY_KEY = 'jiaoshi_history_records';

const loadHistory = () => {
  try {
    const raw = localStorage.getItem(HISTORY_KEY);
    if (raw) {
      historyList.value = JSON.parse(raw);
    }
  } catch (e) {
    console.error('加载历史记录失败', e);
  }
};

const saveHistory = (item: HistoryItem) => {
  try {
    historyList.value.unshift(item);
    if (historyList.value.length > 20) {
      historyList.value = historyList.value.slice(0, 20);
    }
    localStorage.setItem(HISTORY_KEY, JSON.stringify(historyList.value));
  } catch (e) {
    console.error('保存历史记录失败', e);
  }
};

onMounted(() => {
  loadHistory();
});

// 新课标核心素养与教学设计规范指南
const teacherGuideRules = [
  { 
    title: '三维目标与核心素养对齐', 
    advice: '将知识传授升华为学科核心素养培育，明确学生在情境中解决实际问题的能力。', 
    avoid: '避免教学目标泛泛而谈，切忌缺少具体可衡量的行为动词。' 
  },
  { 
    title: '以学生为中心的课堂互动', 
    advice: '设计梯度式提问与小组探究活动，确保学生主导学习过程与思考留白。', 
    avoid: '切忌“满堂灌”与机械问答，避免互动流于表面形式。' 
  },
  { 
    title: '基于真实学情的课后反思', 
    advice: '直面课堂生成性问题与冷场节点，分析教法选择与学生思维障碍的深层矛盾。', 
    avoid: '避免反思流于流水账式记录，缺少针对性的再教设计。' 
  },
  { 
    title: '教研课题与论文切入点', 
    advice: '聚焦教学真实痛点（如“双减”减负增效、作业分层、跨学科主题学习）进行小切口深钻研。', 
    avoid: '切忌课题题目宽泛无边，缺乏实证数据与可落地操作路径。' 
  }
];

// 计算平均分
const getAverageScoreFromMap = (map: Record<string, number> | null): number => {
  if (!map) return 90;
  const values = Object.values(map);
  if (values.length === 0) return 90;
  const sum = values.reduce((acc, curr) => acc + curr, 0);
  return Math.round(sum / values.length);
};

const getAverageScore = (item: HistoryItem): number => {
  return getAverageScoreFromMap(item.aiScores);
};

// 提取展示结果 (去除打分标签)
const displayResultText = computed(() => {
  if (!result.value) return '';
  return result.value.replace(/\[JIAOSHI_SCORES\](.*?)\[\/JIAOSHI_SCORES\]/s, '').trim();
});

// 判断是否达到免费次数上限
const isLimitReached = computed(() => {
  if (isLoggedIn.value) {
    return authUsesCount.value >= 15;
  }
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

// 获取 API 端点
const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: `专家类别：${promptTopic.value}，教研类型：${activeTeacherType.value}，教学学段：${selectedSchoolStage.value}，学科门类：${selectedSubjectCategory.value}，具体要求：${userInput.value}`,
        style: activeTeacherType.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      const rawText = data.result || '';
      
      // 解析 AI 评分
      const scoreMatch = rawText.match(/\[JIAOSHI_SCORES\](.*?)\[\/JIAOSHI_SCORES\]/s);
      let parsedScores: Record<string, number> | null = null;

      if (scoreMatch && scoreMatch[1]) {
        const scoreStr = scoreMatch[1].trim();
        const parsedMap: Record<string, number> = {};
        scoreStr.split(',').forEach(pair => {
          const [k, v] = pair.split(':');
          if (k && v) {
            const num = parseInt(v.trim(), 10);
            if (!isNaN(num)) {
              parsedMap[k.trim()] = num;
            }
          }
        });
        parsedScores = parsedMap;
        aiScores.value = parsedMap;
        result.value = rawText.replace(/\[JIAOSHI_SCORES\](.*?)\[\/JIAOSHI_SCORES\]/s, '').trim();
      } else {
        parsedScores = {
          pedagogicalRigor: 92,
          studentCentricity: 95,
          curriculumDesignClarity: 90,
          teachingReflectionDepth: 88,
          teacherProfessionalism: 94
        };
        aiScores.value = parsedScores;
        result.value = rawText.trim();
      }

      // 保存至历史记录
      const historyItem: HistoryItem = {
        id: Date.now().toString(),
        timestamp: new Date().toLocaleString(),
        teacherType: activeTeacherType.value,
        schoolStage: selectedSchoolStage.value,
        subjectCategory: selectedSubjectCategory.value,
        input: userInput.value,
        aiScores: parsedScores,
        output: result.value
      };
      saveHistory(historyItem);

      // 次数计数更新
      if (isLoggedIn.value) {
        const nextAuthUses = authUsesCount.value + 1;
        localStorage.setItem('auth_uses', nextAuthUses.toString());
        authUsesCount.value = nextAuthUses;
      } else {
        const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
        localStorage.setItem('free_uses', (currentUses + 1).toString());
      }
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleApplyTemplate = (payload: { prompt: string; teacherType?: string; schoolStage?: string; subjectCategory?: string }) => {
  userInput.value = payload.prompt;
  if (payload.teacherType) activeTeacherType.value = payload.teacherType;
  if (payload.schoolStage) selectedSchoolStage.value = payload.schoolStage;
  if (payload.subjectCategory) selectedSubjectCategory.value = payload.subjectCategory;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const toggleHistoryDrawer = () => {
  showHistory.value = !showHistory.value;
};

const applyHistory = (item: HistoryItem) => {
  userInput.value = item.input;
  if (item.teacherType) activeTeacherType.value = item.teacherType;
  if (item.schoolStage) selectedSchoolStage.value = item.schoolStage;
  if (item.subjectCategory) selectedSubjectCategory.value = item.subjectCategory;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const viewHistoryOutput = (item: HistoryItem) => {
  result.value = item.output;
  aiScores.value = item.aiScores || null;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const resetResult = () => {
  result.value = '';
  aiScores.value = null;
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(result.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>

<style scoped>
.options-row {
  margin-bottom: 0.5rem;
}

.metrics-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 1rem;
}

.text-link-btn {
  background: none;
  border: none;
  color: var(--primary-color);
  font-size: 0.75rem;
  cursor: pointer;
  padding: 0;
  text-decoration: underline;
}

.text-link-btn:hover {
  color: #a5b4fc;
}

.qr-code-placeholder {
  width: 140px;
  height: 140px;
  margin: 1rem auto;
  padding: 8px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
</style>
