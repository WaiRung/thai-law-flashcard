<template>
    <main class="main-content">
        <LoadingSpinner v-if="isLoading" message="กำลังเตรียมคำถาม..." />

        <section v-else-if="!showResult && !currentQuestion" class="empty-state" aria-live="polite">
            <div class="empty-state-panel">
                <p class="empty-state-label">แบบทดสอบยังไม่พร้อมใช้งาน</p>
                <h2 class="empty-state-title">ไม่พบข้อคำถามสำหรับชุดนี้</h2>
                <p class="empty-state-body">
                    ลองกลับไปเลือกหมวดหมู่หรือแหล่งข้อมูลอื่น เพื่อเริ่มทำแบบทดสอบจากเนื้อหาที่มีข้อคำถามครบถ้วน
                </p>
                <div class="empty-state-actions">
                    <button type="button" class="secondary-button" @click="handleBack">
                        กลับไปเลือกหมวดหมู่
                    </button>
                    <button type="button" class="primary-button" @click="initializeQuiz">
                        โหลดแบบทดสอบอีกครั้ง
                    </button>
                </div>
            </div>
        </section>

        <template v-else-if="!showResult && currentQuestion">
            <section class="quiz-progress" aria-label="สถานะแบบทดสอบ">
                <div class="progress-header">
                    <div class="progress-copy">
                        <p class="progress-label">กำลังทำแบบทดสอบ</p>
                        <p class="progress-title">ข้อ {{ currentQuestionIndex + 1 }} จาก {{ totalQuestions }}</p>
                    </div>

                    <div
                        class="timer-display"
                        :class="{
                            'timer-warning': remainingTime <= 5,
                            'timer-danger': remainingTime <= 3,
                            'timer-paused': isPaused,
                        }"
                    >
                        <span class="timer-label">{{ isPaused ? 'หยุดเวลาไว้' : 'เวลาที่เหลือ' }}</span>
                        <span class="timer-value">{{ formattedTime }}</span>
                    </div>

                    <button
                        v-if="!isAnswered"
                        type="button"
                        class="pause-button"
                        :class="{ paused: isPaused }"
                        :aria-label="isPaused ? 'ดำเนินการต่อ' : 'หยุดชั่วคราว'"
                        @click="togglePause"
                    >
                        <span class="pause-text">{{ isPaused ? 'เล่นต่อ' : 'พักเวลา' }}</span>
                    </button>

                    <div class="score-display">
                        <span class="score-label">คะแนนสะสม</span>
                        <span class="score-value">{{ score }}/{{ totalQuestions }}</span>
                    </div>
                </div>

                <div class="bar-meta">
                    <span>ความคืบหน้า</span>
                    <span>{{ Math.round(progressPercentage) }}%</span>
                </div>
                <div class="progress-bar" aria-hidden="true">
                    <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
                </div>

                <div class="bar-meta">
                    <span>{{ isPaused ? 'ตัวจับเวลาถูกหยุดไว้' : 'ตัวจับเวลารายข้อ' }}</span>
                    <span>{{ formattedTime }}</span>
                </div>
                <div class="timer-bar" aria-hidden="true">
                    <div
                        class="timer-fill"
                        :class="{
                            'timer-warning': remainingTime <= 5,
                            'timer-danger': remainingTime <= 3,
                            'timer-paused': isPaused,
                        }"
                        :style="{ width: timerPercentage + '%' }"
                    ></div>
                </div>
            </section>

            <div
                v-if="isPaused"
                class="pause-overlay"
                role="dialog"
                aria-modal="true"
                aria-labelledby="pause-title"
            >
                <div class="pause-modal">
                    <p class="pause-label">หยุดชั่วคราว</p>
                    <h2 id="pause-title" class="pause-title">เวลาจะไม่เดินจนกว่าคุณจะพร้อม</h2>
                    <p class="pause-subtitle">กดปุ่มด้านล่างเพื่อกลับไปตอบคำถามข้อนี้ต่อทันที</p>
                    <button type="button" class="resume-button" @click="togglePause">
                        เล่นต่อ
                    </button>
                </div>
            </div>

            <template v-if="!isPaused">
                <QuizQuestion
                    :question="currentQuestion"
                    :question-number="currentQuestionIndex + 1"
                    :total-questions="totalQuestions"
                    :selected-answer="selectedAnswer"
                    :is-answered="isAnswered"
                    :is-correct="isCorrect"
                    :last-answer-score="lastAnswerScore"
                    @select="handleSelectAnswer"
                />

                <div v-if="isAnswered" class="next-button-container">
                    <button type="button" class="next-button" @click="handleNext">
                        {{ isLastQuestion ? 'ดูสรุปผล' : 'ไปข้อถัดไป' }}
                        <span class="next-arrow">→</span>
                    </button>
                </div>
            </template>
        </template>

        <QuizResult
            v-else-if="showResult"
            :result="quizResult"
            :category-id="categoryId"
            @play-again="handlePlayAgain"
            @back="handleBack"
        />
    </main>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import QuizQuestion from "../components/QuizQuestion.vue";
import QuizResult from "../components/QuizResult.vue";
import LoadingSpinner from "../components/LoadingSpinner.vue";
import { categoryStores } from "../data/categoryStores";
import { fetchCategories } from "../services/api";
import { getCategoriesCache, isCacheValid } from "../services/cache";
import { filterQuestionsByDataSource } from "../services/filterService";
import { generateQuizQuestions } from "../services/quizService";
import { calculateCountdownTime, calculateAnswerScore, formatTime } from "../services/scoreService";
import { useHeader } from "../composables/useHeader";
import type { QuizQuestion as QuizQuestionType, QuizResult as QuizResultType, QuizAnswerScore } from "../types/quiz";
import type { CategoryStore } from "../types/flashcard";

const router = useRouter();
const route = useRoute();
const { setHeader, resetHeader } = useHeader();

const QUIZ_QUESTION_COUNT = 20;

const categoryId = computed(() => route.params.categoryId as string);
const dataSourceIndex = computed(() => {
    const index = route.params.dataSourceIndex;
    return index !== undefined ? parseInt(index as string, 10) : undefined;
});

const isLoading = ref(true);
const questions = ref<QuizQuestionType[]>([]);
const currentQuestionIndex = ref(0);
const score = ref(0);
const selectedAnswer = ref<string | null>(null);
const isAnswered = ref(false);
const isCorrect = ref<boolean | null>(null);
const showResult = ref(false);
const categories = ref<CategoryStore[]>([]);

const remainingTime = ref(0);
const totalTimeForQuestion = ref(0);
const timerInterval = ref<ReturnType<typeof setInterval> | null>(null);
const timerStartTime = ref(0);
const totalScore = ref(0);
const totalTimeBonus = ref(0);
const lastAnswerScore = ref<QuizAnswerScore | null>(null);

const isPaused = ref(false);
const pauseStartTimestamp = ref(0);

const roundToTwo = (value: number): number => Math.round(value * 100) / 100;

const currentQuestion = computed(() => questions.value[currentQuestionIndex.value]);
const totalQuestions = computed(() => questions.value.length);
const isLastQuestion = computed(() => currentQuestionIndex.value >= totalQuestions.value - 1);
const answeredCount = computed(() => currentQuestionIndex.value + (isAnswered.value ? 1 : 0));
const progressPercentage = computed(() => {
    if (totalQuestions.value === 0) return 0;
    return (answeredCount.value / totalQuestions.value) * 100;
});
const timerPercentage = computed(() => {
    if (totalTimeForQuestion.value === 0) return 0;
    return (remainingTime.value / totalTimeForQuestion.value) * 100;
});
const formattedTime = computed(() => formatTime(remainingTime.value));

const quizResult = computed<QuizResultType>(() => ({
    totalQuestions: totalQuestions.value,
    correctAnswers: score.value,
    score: score.value,
    percentage: totalQuestions.value > 0 ? Math.round((score.value / totalQuestions.value) * 100) : 0,
    timeBonus: roundToTwo(totalTimeBonus.value),
    totalScore: roundToTwo(totalScore.value),
    maxScore: totalQuestions.value * 2,
}));

const startTimer = () => {
    stopTimer();
    if (!currentQuestion.value) return;

    const questionTime = calculateCountdownTime(currentQuestion.value.question);
    totalTimeForQuestion.value = questionTime;
    remainingTime.value = questionTime;
    timerStartTime.value = Date.now();

    timerInterval.value = setInterval(() => {
        if (!isAnswered.value && !isPaused.value) {
            const elapsed = (Date.now() - timerStartTime.value) / 1000;
            const remaining = totalTimeForQuestion.value - elapsed;
            remainingTime.value = Math.max(0, remaining);

            if (remainingTime.value <= 0) {
                handleTimeUp();
            }
        }
    }, 100);
};

const stopTimer = () => {
    if (timerInterval.value) {
        clearInterval(timerInterval.value);
        timerInterval.value = null;
    }
};

const togglePause = () => {
    if (isAnswered.value) return;

    if (isPaused.value) {
        const pauseDuration = (Date.now() - pauseStartTimestamp.value) / 1000;
        timerStartTime.value = timerStartTime.value + pauseDuration * 1000;
        isPaused.value = false;
    } else {
        pauseStartTimestamp.value = Date.now();
        isPaused.value = true;
    }
};

const handleTimeUp = () => {
    if (isAnswered.value) return;

    selectedAnswer.value = null;
    isAnswered.value = true;
    isCorrect.value = false;
    lastAnswerScore.value = {
        baseScore: 0,
        timeBonus: 0,
        totalPoints: 0,
    };
    stopTimer();
};

const loadCategories = async () => {
    try {
        const cachedCategories = await getCategoriesCache();
        const cacheIsValid = await isCacheValid();

        if (cachedCategories && cachedCategories.length > 0 && cacheIsValid) {
            categories.value = cachedCategories;
            return;
        }

        categories.value = await fetchCategories();
    } catch (err) {
        void err;
        categories.value = categoryStores;
    }
};

const initializeQuiz = async () => {
    isLoading.value = true;
    stopTimer();

    await loadCategories();

    const selectedStore = categories.value.find((store) => store.id === categoryId.value);
    if (!selectedStore) {
        router.replace("/");
        return;
    }

    if (dataSourceIndex.value !== undefined && selectedStore.dataSources) {
        const dataSource = selectedStore.dataSources[dataSourceIndex.value];
        if (dataSource) {
            const title = dataSource.nameTh || selectedStore.nameTh;
            const subtitle = dataSource.nameEn || selectedStore.nameEn;
            setHeader(title, subtitle);
        } else {
            setHeader(selectedStore.nameTh, selectedStore.nameEn);
        }
    } else {
        setHeader(selectedStore.nameTh, selectedStore.nameEn);
    }

    const filteredQuestions = await filterQuestionsByDataSource(
        categoryId.value,
        selectedStore.questions,
        dataSourceIndex.value,
    );

    questions.value = generateQuizQuestions(filteredQuestions, QUIZ_QUESTION_COUNT);
    currentQuestionIndex.value = 0;
    score.value = 0;
    totalScore.value = 0;
    totalTimeBonus.value = 0;
    selectedAnswer.value = null;
    isAnswered.value = false;
    isCorrect.value = null;
    showResult.value = false;
    lastAnswerScore.value = null;
    isPaused.value = false;
    isLoading.value = false;

    startTimer();
};

const handleSelectAnswer = (answer: string) => {
    if (isAnswered.value || !currentQuestion.value) return;

    stopTimer();
    selectedAnswer.value = answer;
    isAnswered.value = true;
    isCorrect.value = answer === currentQuestion.value.correctAnswer;

    const answerScore = calculateAnswerScore(
        isCorrect.value,
        remainingTime.value,
        totalTimeForQuestion.value,
    );

    lastAnswerScore.value = answerScore;

    if (isCorrect.value) {
        score.value++;
    }

    totalScore.value += answerScore.totalPoints;
    totalTimeBonus.value += answerScore.timeBonus;
};

const handleNext = () => {
    if (isLastQuestion.value) {
        stopTimer();
        showResult.value = true;
    } else {
        currentQuestionIndex.value++;
        selectedAnswer.value = null;
        isAnswered.value = false;
        isCorrect.value = null;
        lastAnswerScore.value = null;
        startTimer();
    }
};

const handlePlayAgain = () => {
    void initializeQuiz();
};

const handleBack = () => {
    stopTimer();
    void router.push({ name: "quizlaw-categories" });
};

onMounted(() => {
    void initializeQuiz();
});

onUnmounted(() => {
    stopTimer();
    resetHeader();
});
</script>

<style scoped>
.main-content {
    --quiz-indigo: #6366f1;
    --quiz-indigo-deep: #4f46e5;
    --quiz-surface: #ffffff;
    --quiz-surface-soft: #f9fafb;
    --quiz-canvas: #f3f4f6;
    --quiz-border: #dbe3ee;
    --quiz-border-strong: #c7d2fe;
    --quiz-text: #1f2937;
    --quiz-body: #4b5563;
    --quiz-muted: #6b7280;
    --quiz-warning: #b45309;
    --quiz-warning-soft: #fef3c7;
    --quiz-danger: #dc2626;
    --quiz-danger-soft: #fee2e2;
    flex: 1;
    padding: 1.25rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.5rem max(0.75rem, env(safe-area-inset-left, 0px));
    max-width: 600px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
}

.empty-state {
    display: flex;
    flex: 1;
    align-items: center;
}

.empty-state-panel,
.quiz-progress,
.pause-modal {
    background: var(--quiz-surface);
    border: 1px solid var(--quiz-border);
    border-radius: 1rem;
    box-shadow: 0 10px 30px rgba(15, 23, 42, 0.08);
}

.empty-state-panel {
    width: 100%;
    padding: 1.5rem;
    text-align: left;
}

.empty-state-label,
.progress-label,
.pause-label {
    margin: 0 0 0.375rem;
    font-size: 0.875rem;
    font-weight: 600;
    color: var(--quiz-indigo-deep);
}

.empty-state-title,
.progress-title,
.pause-title {
    margin: 0;
    color: var(--quiz-text);
}

.empty-state-title {
    font-size: 1.375rem;
    line-height: 1.35;
}

.empty-state-body,
.pause-subtitle {
    margin: 0.75rem 0 0;
    font-size: 0.95rem;
    line-height: 1.6;
    color: var(--quiz-body);
}

.empty-state-actions {
    display: flex;
    flex-wrap: wrap;
    gap: 0.75rem;
    margin-top: 1.25rem;
}

.quiz-progress {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    padding: 1rem;
}

.progress-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 0.75rem;
    flex-wrap: wrap;
}

.progress-copy {
    display: flex;
    flex-direction: column;
    gap: 0.125rem;
}

.progress-title {
    font-size: 1.125rem;
    font-weight: 700;
    line-height: 1.35;
}

.timer-display {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
    gap: 0.125rem;
    padding: 0.625rem 0.875rem;
    background: var(--quiz-surface-soft);
    border: 1px solid var(--quiz-border);
    border-radius: 1rem;
    color: var(--quiz-text);
    transition: background-color 0.25s ease, border-color 0.25s ease, color 0.25s ease;
    min-height: 44px;
}

.timer-label {
    font-size: 0.75rem;
    color: var(--quiz-muted);
}

.timer-display.timer-warning {
    background: var(--quiz-warning-soft);
    border-color: #fcd34d;
    color: var(--quiz-warning);
}

.timer-display.timer-danger {
    background: var(--quiz-danger-soft);
    border-color: #fca5a5;
    color: var(--quiz-danger);
    animation: pulse 0.5s ease-in-out infinite;
}

@keyframes pulse {
    0%,
    100% {
        transform: scale(1);
    }

    50% {
        transform: scale(1.03);
    }
}

.timer-value {
    min-width: 2rem;
    text-align: center;
    font-size: 1.125rem;
    font-weight: 700;
}

.score-display {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    justify-content: center;
    gap: 0.25rem;
    margin-left: auto;
    min-height: 44px;
    color: var(--quiz-muted);
    font-size: 0.875rem;
    font-weight: 500;
}

.score-value {
    color: var(--quiz-indigo-deep);
    font-size: 1rem;
    font-weight: 700;
}

.bar-meta {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.75rem;
    font-size: 0.8125rem;
    color: var(--quiz-muted);
}

.progress-bar {
    height: 8px;
    background: var(--quiz-canvas);
    border-radius: 9999px;
    overflow: hidden;
}

.progress-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--quiz-indigo) 0%, var(--quiz-indigo-deep) 100%);
    border-radius: 9999px;
    transition: width 0.25s ease;
}

.timer-bar {
    height: 4px;
    background: var(--quiz-canvas);
    border-radius: 9999px;
    overflow: hidden;
}

.timer-fill {
    height: 100%;
    background: linear-gradient(90deg, #10b981 0%, #059669 100%);
    border-radius: 9999px;
    transition: width 0.1s linear;
}

.timer-fill.timer-warning {
    background: linear-gradient(90deg, #f59e0b 0%, #d97706 100%);
}

.timer-fill.timer-danger {
    background: linear-gradient(90deg, #ef4444 0%, #dc2626 100%);
}

.timer-display.timer-paused {
    background: #eef2ff;
    border-color: var(--quiz-border-strong);
    color: #4338ca;
}

.timer-fill.timer-paused {
    background: linear-gradient(90deg, #818cf8 0%, #6366f1 100%);
}

.pause-button,
.secondary-button,
.primary-button,
.resume-button,
.next-button {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    min-height: 44px;
    padding: 0.875rem 1.25rem;
    border-radius: 0.875rem;
    font-size: 0.9375rem;
    font-weight: 600;
    cursor: pointer;
    -webkit-tap-highlight-color: transparent;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        border-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.pause-button,
.secondary-button {
    background: var(--quiz-surface-soft);
    border: 1px solid var(--quiz-border);
    color: var(--quiz-body);
}

.primary-button,
.resume-button,
.next-button {
    background: linear-gradient(135deg, var(--quiz-indigo) 0%, var(--quiz-indigo-deep) 100%);
    border: 1px solid transparent;
    color: #ffffff;
    box-shadow: 0 10px 20px rgba(79, 70, 229, 0.18);
}

@media (hover: hover) {
    .pause-button:hover,
    .secondary-button:hover {
        background: #e5e7eb;
        border-color: #d1d5db;
    }

    .primary-button:hover,
    .resume-button:hover,
    .next-button:hover {
        transform: translateY(-1px);
        box-shadow: 0 12px 24px rgba(79, 70, 229, 0.24);
    }
}

.pause-button.paused {
    background: #dcfce7;
    border-color: #86efac;
    color: #166534;
}

.pause-button.paused:hover {
    background: #bbf7d0;
}

.pause-button:focus-visible,
.secondary-button:focus-visible,
.primary-button:focus-visible,
.resume-button:focus-visible,
.next-button:focus-visible {
    outline: 3px solid rgba(99, 102, 241, 0.28);
    outline-offset: 2px;
}

.pause-text {
    white-space: nowrap;
}

.pause-overlay {
    position: fixed;
    inset: 0;
    z-index: 100;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(15, 23, 42, 0.55);
    backdrop-filter: blur(4px);
}

.pause-modal {
    width: min(90%, 320px);
    padding: 2rem;
    text-align: center;
    animation: scaleIn 0.2s ease-out;
}

@keyframes scaleIn {
    0% {
        transform: scale(0.96);
        opacity: 0;
    }

    100% {
        transform: scale(1);
        opacity: 1;
    }
}

.pause-title {
    font-size: 1.5rem;
    font-weight: 700;
}

.resume-button {
    width: 100%;
    margin-top: 1.5rem;
}

.next-button-container {
    display: flex;
    justify-content: center;
    margin-top: 1rem;
}

.next-arrow {
    font-size: 1.25rem;
    transition: transform 0.2s ease;
}

.next-button:hover .next-arrow {
    transform: translateX(4px);
}

@media (max-width: 640px) {
    .main-content {
        padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
        gap: 1rem;
    }

    .progress-header {
        align-items: stretch;
    }

    .timer-display,
    .pause-button,
    .score-display {
        flex: 1 1 calc(50% - 0.5rem);
    }

    .score-display {
        order: 3;
        flex-basis: 100%;
        margin-left: 0;
        align-items: flex-start;
    }

    .next-button {
        width: 100%;
    }

    .pause-button {
        flex: 1 1 auto;
    }

    .pause-modal {
        padding: 1.5rem;
    }

    .pause-title {
        font-size: 1.25rem;
    }

    .empty-state-actions {
        flex-direction: column-reverse;
    }

    .empty-state-actions > * {
        width: 100%;
    }
}

@media (prefers-reduced-motion: reduce) {
    .timer-display,
    .pause-button,
    .secondary-button,
    .primary-button,
    .resume-button,
    .next-button,
    .progress-fill,
    .timer-fill {
        transition: none;
    }

    .timer-display.timer-danger,
    .pause-modal {
        animation: none;
    }
}
</style>