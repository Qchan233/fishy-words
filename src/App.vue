<template>
  <div class="app">
    <!-- 头部 -->
    <header class="header">
      <h1>🐟 小鱼背单词</h1>
      <p class="subtitle">每天进步一点点～</p>
    </header>

    <!-- 统计面板 -->
    <div class="stats-panel">
      <div class="stat-item">
        <span class="stat-label">今日学习</span>
        <span class="stat-value">{{ stats.todayLearned }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">掌握单词</span>
        <span class="stat-value">{{ stats.mastered }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">错题本</span>
        <span class="stat-value">{{ stats.mistakes }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">连续打卡</span>
        <span class="stat-value">{{ stats.streak }}天</span>
      </div>
    </div>

    <!-- 单词卡片 -->
    <div class="card-container">
      <div 
        class="word-card" 
        :class="{ 'flipped': isFlipped, 'known': isKnown, 'unknown': isUnknown }"
        @click="flipCard"
      >
        <div class="card-inner">
          <!-- 正面 -->
          <div class="card-front">
            <div class="word-text">{{ currentWord.word }}</div>
            <div class="pronunciation">{{ currentWord.pronunciation }}</div>
            <div class="pos-tag">{{ currentWord.pos }}</div>
            <div class="hint-text">点击翻转查看释义</div>
          </div>
          
          <!-- 背面 -->
          <div class="card-back">
            <div class="meanings">
              <div v-for="(meaning, idx) in currentWord.meanings" :key="idx" class="meaning-item">
                {{ meaning }}
              </div>
            </div>
            <div class="example-box">
              <p class="example-en">{{ currentWord.example }}</p>
              <p class="example-cn">{{ currentWord.exampleCn }}</p>
            </div>
            <div class="tags">
              <span v-for="tag in currentWord.tags" :key="tag" class="tag">{{ tag }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 操作按钮 -->
    <div class="action-buttons">
      <button class="btn btn-unknown" @click="markUnknown" :disabled="!isFlipped">
        ❌ 不认识
      </button>
      <button class="btn btn-known" @click="markKnown" :disabled="!isFlipped">
        ✅ 认识
      </button>
    </div>

    <!-- 进度条 -->
    <div class="progress-container">
      <div class="progress-bar">
        <div 
          class="progress-fill" 
          :style="{ width: progressPercentage + '%' }"
        ></div>
      </div>
      <div class="progress-text">
        {{ currentIndex + 1 }} / {{ wordList.length }}
      </div>
    </div>

    <!-- 完成提示 -->
    <div v-if="isCompleted" class="completion-modal">
      <div class="modal-content">
        <h2>🎉 太棒啦！</h2>
        <p>今天的学习完成啦～</p>
        <div class="completion-stats">
          <p>今日学习：<strong>{{ stats.todayLearned }}</strong> 词</p>
          <p>掌握单词：<strong>{{ stats.mastered }}</strong> 词</p>
        </div>
        <button class="btn btn-primary" @click="restart">
          再来一轮
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import wordLibrary from './data/words.js';

export default {
  name: 'App',
  data() {
    return {
      wordList: [],
      currentIndex: 0,
      isFlipped: false,
      isKnown: false,
      isUnknown: false,
      isCompleted: false,
      stats: {
        todayLearned: 0,
        mastered: 0,
        mistakes: 0,
        streak: 1,
        lastLearnDate: null
      }
    };
  },
  computed: {
    currentWord() {
      return this.wordList[this.currentIndex] || {};
    },
    progressPercentage() {
      return ((this.currentIndex + 1) / this.wordList.length) * 100;
    }
  },
  mounted() {
    this.loadStats();
    this.initWordList();
    this.checkStreak();
  },
  methods: {
    loadStats() {
      const saved = localStorage.getItem('fishyWordsStats');
      if (saved) {
        this.stats = JSON.parse(saved);
      }
    },
    saveStats() {
      localStorage.setItem('fishyWordsStats', JSON.stringify(this.stats));
    },
    checkStreak() {
      const today = new Date().toDateString();
      if (this.stats.lastLearnDate !== today) {
        const yesterday = new Date();
        yesterday.setDate(yesterday.getDate() - 1);
        if (this.stats.lastLearnDate !== yesterday.toDateString()) {
          this.stats.streak = 1;
        }
      }
    },
    initWordList() {
      // 随机打乱单词
      this.wordList = [...wordLibrary].sort(() => Math.random() - 0.5);
    },
    flipCard() {
      this.isFlipped = !this.isFlipped;
    },
    markKnown() {
      if (!this.isFlipped) return;
      
      this.isKnown = true;
      this.stats.todayLearned++;
      this.stats.mastered++;
      this.saveStats();
      
      setTimeout(() => {
        this.nextWord();
      }, 300);
    },
    markUnknown() {
      if (!this.isFlipped) return;
      
      this.isUnknown = true;
      this.stats.todayLearned++;
      this.stats.mistakes++;
      this.saveStats();
      
      // 加入错题本
      this.addToMistakes(this.currentWord);
      
      setTimeout(() => {
        this.nextWord();
      }, 300);
    },
    addToMistakes(word) {
      const mistakes = JSON.parse(localStorage.getItem('fishyWordsMistakes') || '[]');
      if (!mistakes.find(w => w.id === word.id)) {
        mistakes.push(word);
        localStorage.setItem('fishyWordsMistakes', JSON.stringify(mistakes));
      }
    },
    nextWord() {
      this.isFlipped = false;
      this.isKnown = false;
      this.isUnknown = false;
      
      if (this.currentIndex < this.wordList.length - 1) {
        this.currentIndex++;
      } else {
        this.completeLearning();
      }
    },
    completeLearning() {
      this.isCompleted = true;
      this.stats.lastLearnDate = new Date().toDateString();
      this.saveStats();
    },
    restart() {
      this.isCompleted = false;
      this.currentIndex = 0;
      this.initWordList();
    }
  }
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  padding: 20px;
}

.app {
  max-width: 500px;
  margin: 0 auto;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 20px;
}

.header h1 {
  font-size: 2em;
  margin-bottom: 5px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}

.subtitle {
  font-size: 1em;
  opacity: 0.9;
}

/* 统计面板 */
.stats-panel {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  margin-bottom: 20px;
}

.stat-item {
  background: rgba(255,255,255,0.2);
  backdrop-filter: blur(10px);
  border-radius: 12px;
  padding: 12px 8px;
  text-align: center;
  color: white;
}

.stat-label {
  display: block;
  font-size: 0.75em;
  opacity: 0.9;
  margin-bottom: 4px;
}

.stat-value {
  display: block;
  font-size: 1.5em;
  font-weight: bold;
}

/* 单词卡片 */
.card-container {
  perspective: 1000px;
  margin-bottom: 20px;
}

.word-card {
  width: 100%;
  height: 350px;
  cursor: pointer;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.word-card.flipped {
  transform: rotateY(180deg);
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transform-style: preserve-3d;
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 30px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.card-front {
  background: white;
  color: #333;
}

.card-back {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  transform: rotateY(180deg);
  text-align: center;
}

.word-text {
  font-size: 2.5em;
  font-weight: bold;
  color: #667eea;
  margin-bottom: 10px;
}

.pronunciation {
  font-size: 1.2em;
  color: #666;
  margin-bottom: 10px;
}

.pos-tag {
  background: #667eea;
  color: white;
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.9em;
  margin-bottom: 20px;
}

.hint-text {
  color: #999;
  font-size: 0.9em;
  margin-top: 30px;
}

.meanings {
  margin-bottom: 20px;
}

.meaning-item {
  font-size: 1.2em;
  color: #333;
  margin-bottom: 8px;
}

.example-box {
  background: white;
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 15px;
  width: 100%;
}

.example-en {
  color: #667eea;
  font-style: italic;
  margin-bottom: 5px;
}

.example-cn {
  color: #666;
  font-size: 0.95em;
}

.tags {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  justify-content: center;
}

.tag {
  background: #667eea;
  color: white;
  padding: 4px 10px;
  border-radius: 15px;
  font-size: 0.8em;
}

/* 操作按钮 */
.action-buttons {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
}

.btn {
  flex: 1;
  padding: 15px;
  border: none;
  border-radius: 12px;
  font-size: 1.1em;
  cursor: pointer;
  transition: all 0.3s;
  font-weight: bold;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-unknown {
  background: rgba(255,255,255,0.3);
  color: white;
  border: 2px solid white;
}

.btn-unknown:not(:disabled):hover {
  background: rgba(255,255,255,0.5);
}

.btn-known {
  background: white;
  color: #667eea;
}

.btn-known:not(:disabled):hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
}

.btn-primary {
  background: #667eea;
  color: white;
  margin-top: 15px;
  width: 100%;
}

.btn-primary:hover {
  background: #5568d3;
}

/* 进度条 */
.progress-container {
  text-align: center;
  color: white;
}

.progress-bar {
  background: rgba(255,255,255,0.3);
  border-radius: 10px;
  height: 8px;
  overflow: hidden;
  margin-bottom: 10px;
}

.progress-fill {
  background: white;
  height: 100%;
  transition: width 0.3s;
}

.progress-text {
  font-size: 0.9em;
  opacity: 0.9;
}

/* 完成提示 */
.completion-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 40px;
  border-radius: 20px;
  text-align: center;
  max-width: 400px;
  width: 90%;
}

.modal-content h2 {
  color: #667eea;
  margin-bottom: 15px;
}

.modal-content p {
  color: #666;
  margin-bottom: 20px;
}

.completion-stats {
  background: #f5f7fa;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.completion-stats p {
  margin: 10px 0;
  color: #333;
}

.completion-stats strong {
  color: #667eea;
  font-size: 1.2em;
}
</style>
