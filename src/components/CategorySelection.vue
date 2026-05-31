<template>
    <div class="category-container">
        <div class="category-header">
            <h2 class="category-title">เลือกหมวดกฎหมาย</h2>
            <p class="category-subtitle">Select Law Category</p>
        </div>

        <div class="category-grid">
            <button
                v-for="category in categories"
                :key="category.id"
                @click="selectCategory(category.id)"
                type="button"
                class="category-card"
                :aria-label="`เลือกหมวด ${category.nameTh} (${category.nameEn}) จำนวน ${category.count} คำถาม`"
            >
                <div class="category-icon">{{ category.icon }}</div>
                <div class="category-name-th">{{ category.nameTh }}</div>
                <div class="category-name-en">{{ category.nameEn }}</div>
                <div class="category-count">{{ category.count }} คำถาม</div>
                
                <!-- High Score Display -->
                <div v-if="getHighScore(category.id)" class="category-high-score">
                    <span class="high-score-icon">🏆</span>
                    <span class="high-score-text">{{ getHighScore(category.id)?.percentage }}%</span>
                </div>
            </button>
        </div>
    </div>
</template>

<script setup lang="ts">
import type { HighScore } from "../types/quiz";

interface Category {
    id: string;
    nameTh: string;
    nameEn: string;
    icon: string;
    count: number;
}

interface Props {
    categories: Category[];
    highScores?: Map<string, HighScore>;
}

const props = defineProps<Props>();

const emit = defineEmits<{
    select: [categoryId: string];
}>();

const selectCategory = (categoryId: string) => {
    emit("select", categoryId);
};

const getHighScore = (categoryId: string): HighScore | undefined => {
    return props.highScores?.get(categoryId);
};
</script>

<style scoped>
.category-container {
    --category-ink: #1f2937;
    --category-body: #4b5563;
    --category-muted: #6b7280;
    --category-border: #e5e7eb;
    --category-surface: #ffffff;
    --category-surface-soft: #f9fafb;
    --category-accent: #3b82f6;
    --category-accent-deep: #1d4ed8;
    --category-focus: rgba(59, 130, 246, 0.28);

    display: flex;
    flex-direction: column;
    padding: 0.5rem 0;
    max-width: 100%;
    margin: 0 auto;
}

.category-header {
    text-align: center;
    margin-bottom: 1.75rem;
}

.category-title {
    font-size: 1.875rem;
    font-weight: 700;
    color: var(--category-ink);
    margin: 0;
    margin-bottom: 0.5rem;
    line-height: 1.25;
    overflow-wrap: anywhere;
}

.category-subtitle {
    font-size: 1rem;
    color: var(--category-body);
    margin: 0;
    line-height: 1.6;
}

.category-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1rem;
    width: 100%;
}

.category-card {
    background: var(--category-surface);
    border: 2px solid var(--category-border);
    border-radius: 1rem;
    padding: 1.5rem 1.25rem;
    cursor: pointer;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        border-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    min-height: 200px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
    -webkit-tap-highlight-color: transparent;
    animation: categoryEnter 0.3s cubic-bezier(0.2, 0.8, 0.2, 1) both;
}

.category-card:hover {
    background: var(--category-surface-soft);
    transform: translateY(-4px);
    box-shadow: 0 8px 16px rgba(59, 130, 246, 0.2);
    border-color: var(--category-accent);
}

.category-card:active {
    transform: translateY(-2px);
}

.category-card:focus-visible {
    outline: 3px solid var(--category-focus);
    outline-offset: 2px;
    border-color: var(--category-accent-deep);
    box-shadow: 0 0 0 4px rgba(37, 99, 235, 0.16);
}

.category-card:nth-child(1) {
    animation-delay: 0.03s;
}

.category-card:nth-child(2) {
    animation-delay: 0.06s;
}

.category-card:nth-child(3) {
    animation-delay: 0.09s;
}

.category-card:nth-child(4) {
    animation-delay: 0.12s;
}

.category-icon {
    font-size: 3rem;
    margin-bottom: 1rem;
    transition: transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.category-card:hover .category-icon {
    transform: translateY(-1px);
}

.category-name-th {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--category-ink);
    margin-bottom: 0.5rem;
    line-height: 1.4;
    overflow-wrap: anywhere;
}

.category-name-en {
    font-size: 0.875rem;
    color: var(--category-muted);
    margin-bottom: 1rem;
    line-height: 1.5;
    overflow-wrap: anywhere;
}

.category-count {
    font-size: 0.875rem;
    color: var(--category-accent-deep);
    font-weight: 600;
    padding: 0.375rem 0.75rem;
    background-color: #eff6ff;
    border-radius: 9999px;
}

.category-high-score {
    display: flex;
    align-items: center;
    gap: 0.375rem;
    margin-top: 0.75rem;
    padding: 0.375rem 0.75rem;
    background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%);
    border-radius: 9999px;
    border: 1px solid #f59e0b;
}

.high-score-icon {
    font-size: 0.875rem;
}

.high-score-text {
    font-size: 0.875rem;
    font-weight: 700;
    color: #b45309;
}

@keyframes categoryEnter {
    from {
        opacity: 0;
        transform: translateY(8px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@media (prefers-reduced-motion: reduce) {
    .category-card,
    .category-icon {
        animation: none;
        transition: none;
    }

    .category-card:hover,
    .category-card:active,
    .category-card:hover .category-icon {
        transform: none;
    }
}

@media (max-width: 640px) {
    .category-container {
        padding: 0.25rem 0;
    }

    .category-title {
        font-size: 1.5rem;
    }

    .category-subtitle {
        font-size: 1rem;
    }

    .category-grid {
        grid-template-columns: 1fr;
        gap: 1rem;
    }

    .category-card {
        padding: 1.5rem 1rem;
        min-height: 176px;
    }

    .category-icon {
        font-size: 2.5rem;
    }

    .category-name-th {
        font-size: 1.125rem;
    }
}
</style>
