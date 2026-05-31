<template>
    <div class="quiz-result-container">
        <div class="result-card">
            <p class="result-kicker">สรุปผลแบบทดสอบ</p>
            <h2 class="result-title">{{ resultTitle }}</h2>
            <p class="result-subtitle">ตรวจทานผลลัพธ์แล้วเลือกทำต่อหรือกลับไปเลือกหมวดใหม่</p>

            <div v-if="isNewHighScore" class="new-high-score-badge">
                <span class="badge-text">สถิติใหม่!</span>
            </div>

            <div class="score-display">
                <div class="score-circle" :class="scoreClass">
                    <span class="score-percentage">{{ result.percentage }}%</span>
                </div>
            </div>

            <div class="total-score-container">
                <div class="total-score-value">{{ result.correctAnswers }} / {{ result.totalQuestions }}</div>
                <div class="total-score-label">ตอบถูกจากทั้งหมด</div>
                <div v-if="result.timeBonus > 0" class="bonus-score-display">
                    <span class="bonus-score-value">+{{ result.timeBonus.toFixed(2) }}</span>
                    <span class="bonus-score-label">โบนัสความเร็ว</span>
                </div>
            </div>

            <div v-if="highScore" class="high-score-container">
                <div class="high-score-header">
                    <span class="high-score-title">สถิติสูงสุด</span>
                </div>
                <div class="high-score-details">
                    <div class="high-score-value">{{ highScore.score }} / {{ highScore.totalQuestions }} ({{ highScore.percentage }}%)</div>
                    <div v-if="highScoreBonus > 0" class="high-score-bonus">
                        <span>+{{ highScoreBonus.toFixed(2) }} โบนัส</span>
                    </div>
                    <div class="high-score-date">{{ formattedHighScoreDate }}</div>
                </div>
            </div>

            <div class="stats-container">
                <div class="stat-item">
                    <span class="stat-value correct">{{ result.correctAnswers }}</span>
                    <span class="stat-label">ถูกต้อง</span>
                </div>
                <div class="stat-divider"></div>
                <div class="stat-item">
                    <span class="stat-value incorrect">{{ result.totalQuestions - result.correctAnswers }}</span>
                    <span class="stat-label">ไม่ถูกต้อง</span>
                </div>
                <div class="stat-divider"></div>
                <div class="stat-item">
                    <span class="stat-value total">{{ result.totalQuestions }}</span>
                    <span class="stat-label">ทั้งหมด</span>
                </div>
            </div>

            <div class="action-buttons">
                <button type="button" class="action-button primary" @click="handlePlayAgain">
                    เล่นอีกครั้ง
                </button>
                <button type="button" class="action-button secondary" @click="handleBack">
                    เลือกหมวดหมู่
                </button>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import type { QuizResult, HighScore } from "../types/quiz";
import { checkAndSaveHighScore, getHighScore, formatThaiDate } from "../services/highScoreService";

interface Props {
    result: QuizResult;
    categoryId: string;
}

const props = defineProps<Props>();

const emit = defineEmits<{
    playAgain: [];
    back: [];
}>();

const highScore = ref<HighScore | null>(null);
const isNewHighScore = ref(false);

const resultTitle = computed(() => {
    if (props.result.percentage >= 80) return "ยอดเยี่ยม!";
    if (props.result.percentage >= 60) return "ดีมาก!";
    if (props.result.percentage >= 40) return "พอใช้";
    return "ลองอีกครั้ง!";
});

const scoreClass = computed(() => {
    if (props.result.percentage >= 80) return "excellent";
    if (props.result.percentage >= 60) return "good";
    if (props.result.percentage >= 40) return "fair";
    return "needs-improvement";
});

const formattedHighScoreDate = computed(() => {
    if (!highScore.value) return "";
    return formatThaiDate(highScore.value.achievedAt);
});

const highScoreBonus = computed(() => {
    if (!highScore.value) return 0;
    return Math.max(0, highScore.value.totalScore - highScore.value.score);
});

const handlePlayAgain = () => {
    emit("playAgain");
};

const handleBack = () => {
    emit("back");
};

onMounted(async () => {
    isNewHighScore.value = await checkAndSaveHighScore(
        props.categoryId,
        props.result.correctAnswers,
        props.result.percentage,
        props.result.totalQuestions,
        props.result.totalScore,
    );

    highScore.value = await getHighScore(props.categoryId);
});
</script>

<style scoped>
.quiz-result-container {
    --quiz-indigo: #6366f1;
    --quiz-indigo-deep: #4f46e5;
    --quiz-surface: #ffffff;
    --quiz-surface-soft: #f9fafb;
    --quiz-border: #e5e7eb;
    --quiz-text: #1f2937;
    --quiz-body: #4b5563;
    --quiz-muted: #6b7280;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    padding: 1rem;
}

.result-card {
    width: 100%;
    max-width: 400px;
    padding: 2rem;
    background: var(--quiz-surface);
    border: 1px solid var(--quiz-border);
    border-radius: 1.5rem;
    box-shadow: 0 18px 38px rgba(15, 23, 42, 0.1);
    text-align: center;
}

.result-kicker {
    margin: 0 0 0.5rem;
    font-size: 0.875rem;
    font-weight: 600;
    color: var(--quiz-indigo-deep);
}

.result-title {
    margin: 0 0 0.25rem;
    font-size: 1.75rem;
    font-weight: 700;
    color: var(--quiz-text);
}

.result-subtitle {
    margin: 0 0 1rem;
    font-size: 1rem;
    line-height: 1.55;
    color: var(--quiz-body);
}

.new-high-score-badge {
    display: inline-flex;
    align-items: center;
    padding: 0.5rem 1rem;
    margin-bottom: 1rem;
    background: linear-gradient(135deg, #fbbf24 0%, #f59e0b 100%);
    border-radius: 9999px;
}

.badge-text {
    font-size: 0.875rem;
    font-weight: 700;
    color: #1f2937;
}

.score-display {
    margin-bottom: 1.5rem;
}

.score-circle {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 120px;
    height: 120px;
    border: 6px solid;
    border-radius: 50%;
}

.score-circle.excellent {
    border-color: #10b981;
    background: linear-gradient(135deg, #d1fae5 0%, #a7f3d0 100%);
}

.score-circle.good {
    border-color: #3b82f6;
    background: linear-gradient(135deg, #dbeafe 0%, #bfdbfe 100%);
}

.score-circle.fair {
    border-color: #f59e0b;
    background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%);
}

.score-circle.needs-improvement {
    border-color: #ef4444;
    background: linear-gradient(135deg, #fee2e2 0%, #fecaca 100%);
}

.score-percentage {
    font-size: 2rem;
    font-weight: 700;
    color: var(--quiz-text);
}

.total-score-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-bottom: 1.5rem;
    padding: 1rem;
    background: linear-gradient(135deg, #fffbeb 0%, #fef3c7 100%);
    border: 1px solid #fbbf24;
    border-radius: 1rem;
}

.total-score-value {
    font-size: 2.5rem;
    font-weight: 800;
    color: #b45309;
}

.total-score-label {
    font-size: 0.875rem;
    font-weight: 600;
    color: #92400e;
}

.bonus-score-display {
    display: flex;
    align-items: center;
    gap: 0.375rem;
    margin-top: 0.5rem;
    padding: 0.375rem 0.75rem;
    background: rgba(255, 255, 255, 0.6);
    border-radius: 1rem;
}

.bonus-score-value {
    font-size: 0.875rem;
    font-weight: 700;
    color: #059669;
}

.bonus-score-label {
    font-size: 0.75rem;
    color: var(--quiz-muted);
}

.high-score-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-bottom: 1.5rem;
    padding: 1rem;
    background: linear-gradient(135deg, #f5f3ff 0%, #ede9fe 100%);
    border: 1px solid #c4b5fd;
    border-radius: 1rem;
}

.high-score-header {
    display: flex;
    align-items: center;
    margin-bottom: 0.5rem;
}

.high-score-title {
    font-size: 0.875rem;
    font-weight: 600;
    color: #6d28d9;
}

.high-score-details {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.high-score-value {
    font-size: 1.25rem;
    font-weight: 700;
    color: #5b21b6;
}

.high-score-bonus {
    margin-top: 0.25rem;
    font-size: 0.75rem;
    color: #059669;
}

.high-score-date {
    margin-top: 0.25rem;
    font-size: 0.75rem;
    color: #7c3aed;
}

.stats-container {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    margin-bottom: 2rem;
    padding: 1rem;
    background: var(--quiz-surface-soft);
    border: 1px solid var(--quiz-border);
    border-radius: 1rem;
}

.stat-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.125rem;
}

.stat-value {
    font-size: 1.5rem;
    font-weight: 700;
}

.stat-value.correct {
    color: #10b981;
}

.stat-value.incorrect {
    color: #ef4444;
}

.stat-value.total {
    color: #6b7280;
}

.stat-label {
    font-size: 0.75rem;
    color: #6b7280;
}

.stat-divider {
    width: 1px;
    height: 40px;
    align-self: center;
    background: #e5e7eb;
}

.action-buttons {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
}

.action-button {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 1rem 1.5rem;
    border: none;
    border-radius: 0.75rem;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.2s ease, box-shadow 0.2s ease, background-color 0.2s ease;
}

.action-button.primary {
    background: linear-gradient(135deg, var(--quiz-indigo) 0%, var(--quiz-indigo-deep) 100%);
    color: #ffffff;
}

.action-button.primary:hover {
    transform: translateY(-1px);
    box-shadow: 0 10px 20px rgba(79, 70, 229, 0.25);
}

.action-button.secondary {
    background: var(--quiz-surface-soft);
    border: 1px solid var(--quiz-border);
    color: var(--quiz-body);
}

.action-button.secondary:hover {
    background: #e5e7eb;
}

.action-button:focus-visible {
    outline: 3px solid rgba(99, 102, 241, 0.25);
    outline-offset: 2px;
}

@media (max-width: 640px) {
    .result-card {
        padding: 1.5rem;
    }

    .result-title {
        font-size: 1.5rem;
    }

    .score-circle {
        width: 100px;
        height: 100px;
    }

    .score-percentage {
        font-size: 1.75rem;
    }

    .stats-container {
        gap: 0.75rem;
    }

    .stat-value {
        font-size: 1.25rem;
    }

    .stat-divider {
        display: none;
    }
}

@media (prefers-reduced-motion: reduce) {
    .action-button {
        transition: none;
    }
}
</style>
