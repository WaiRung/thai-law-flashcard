<template>
    <div class="category-container">
        <div class="category-header">
            <h2 class="category-title">{{ title }}</h2>
            <p class="category-subtitle">{{ subtitle }}</p>
            <p v-if="description" class="category-description">{{ description }}</p>
            <div v-if="categories.length > 0" class="category-summary" aria-label="Category summary">
                <p class="summary-pill">{{ categories.length }} หมวดหมู่</p>
                <p class="summary-pill">{{ totalQuestions }} คำถาม</p>
                <p v-if="completedCategories > 0" class="summary-pill summary-pill-score">
                    {{ completedCategories }} หมวดมีสถิติ
                </p>
            </div>
        </div>

        <div v-if="categories.length > 0" class="category-grid">
            <button
                v-for="category in categories"
                :key="category.id"
                @click="selectCategory(category.id)"
                type="button"
                class="category-card"
                :aria-label="`เลือกหมวด ${category.nameTh} (${category.nameEn}) จำนวน ${category.count} คำถาม`"
            >
                <div class="category-card-main">
                    <div class="category-icon" aria-hidden="true">{{ category.icon }}</div>
                    <div class="category-copy">
                        <div class="category-name-th">{{ category.nameTh }}</div>
                        <div class="category-name-en">{{ category.nameEn }}</div>
                    </div>
                    <div class="category-arrow" aria-hidden="true">
                        <svg
                            xmlns="http://www.w3.org/2000/svg"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor"
                            class="category-arrow-icon"
                        >
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                stroke-width="2"
                                d="M9 5l7 7-7 7"
                            />
                        </svg>
                    </div>
                </div>

                <div class="category-meta">
                    <div class="category-count">{{ category.count }} คำถาม</div>
                    <div v-if="getHighScore(category.id)" class="category-high-score">
                        <span class="high-score-icon" aria-hidden="true">🏆</span>
                        <span class="high-score-text">คะแนนสูงสุด {{ getHighScore(category.id)?.percentage }}%</span>
                    </div>
                </div>
            </button>
        </div>

        <div v-else class="category-empty-state">
            <p class="empty-title">ยังไม่พบหมวดหมู่สำหรับการทบทวน</p>
            <p class="empty-subtitle">No law categories are available right now.</p>
        </div>
    </div>
</template>

<script setup lang="ts">
import { computed } from "vue";
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
    title?: string;
    subtitle?: string;
    description?: string;
}

const props = defineProps<Props>();

const title = computed(() => props.title ?? "เลือกหมวดกฎหมาย");
const subtitle = computed(() => props.subtitle ?? "Select law category");
const description = computed(() => {
    return props.description ?? "เลือกหมวดที่ต้องการทบทวนเพื่อเริ่มทำแบบฝึกแบบต่อเนื่อง";
});

const totalQuestions = computed(() => {
    return props.categories.reduce((total: number, category: Category) => total + category.count, 0);
});

const completedCategories = computed(() => {
    return props.categories.filter((category: Category) => props.highScores?.has(category.id)).length;
});

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
    gap: 1.5rem;
    padding: 0.25rem 0;
    max-width: 100%;
    margin: 0 auto;
}

.category-header {
    text-align: center;
    padding: 1.25rem 1rem 1.5rem;
    border-radius: 1rem;
    border: 1px solid rgba(229, 231, 235, 0.82);
    background: linear-gradient(180deg, rgba(255, 255, 255, 0.92) 0%, rgba(243, 244, 246, 0.92) 100%);
}

.category-title {
    font-size: clamp(1.75rem, 3.8vw, 2.25rem);
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

.category-description {
    margin: 0.75rem auto 0;
    max-width: 54ch;
    font-size: 0.95rem;
    line-height: 1.6;
    color: var(--category-muted);
}

.category-summary {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 0.75rem;
    margin-top: 1rem;
}

.summary-pill {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-height: 2rem;
    padding: 0.35rem 0.75rem;
    border-radius: 999px;
    border: 1px solid rgba(59, 130, 246, 0.16);
    background: rgba(59, 130, 246, 0.08);
    color: var(--category-accent-deep);
    font-size: 0.9rem;
    font-weight: 600;
}

.summary-pill-score {
    border-color: rgba(245, 158, 11, 0.22);
    background: rgba(245, 158, 11, 0.12);
    color: #92400e;
}

.category-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1rem;
    width: 100%;
}

.category-card {
    background: var(--category-surface);
    border: 1px solid var(--category-border);
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
    align-items: stretch;
    text-align: left;
    gap: 1rem;
    min-height: 168px;
    -webkit-tap-highlight-color: transparent;
}

@media (hover: hover) {
    .category-card:hover {
        background: var(--category-surface-soft);
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.15);
        border-color: var(--category-accent);
    }

    .category-card:hover .category-arrow-icon {
        transform: translateX(4px);
        color: var(--category-accent);
    }
}

.category-card:active {
    transform: translateY(0);
}

.category-card:focus-visible {
    outline: 3px solid var(--category-focus);
    outline-offset: 2px;
    border-color: var(--category-accent-deep);
    box-shadow: 0 0 0 4px rgba(37, 99, 235, 0.12);
}

.category-card-main {
    display: flex;
    align-items: flex-start;
    gap: 1rem;
}

.category-icon {
    font-size: 2rem;
    width: 3rem;
    height: 3rem;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border-radius: 0.875rem;
    background: rgba(59, 130, 246, 0.08);
    flex-shrink: 0;
}

.category-copy {
    flex: 1;
    min-width: 0;
}

.category-arrow {
    flex-shrink: 0;
    color: #94a3b8;
}

.category-arrow-icon {
    width: 1.25rem;
    height: 1.25rem;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.category-name-th {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--category-ink);
    margin-bottom: 0.375rem;
    line-height: 1.4;
    overflow-wrap: anywhere;
}

.category-name-en {
    font-size: 0.875rem;
    color: var(--category-body);
    margin-bottom: 0;
    line-height: 1.5;
    overflow-wrap: anywhere;
}

.category-meta {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 0.625rem;
    margin-top: auto;
}

.category-count {
    font-size: 0.875rem;
    color: var(--category-accent-deep);
    font-weight: 600;
    padding: 0.375rem 0.75rem;
    background-color: #eff6ff;
    border: 1px solid rgba(59, 130, 246, 0.12);
    border-radius: 9999px;
}

.category-high-score {
    display: flex;
    align-items: center;
    gap: 0.375rem;
    padding: 0.375rem 0.75rem;
    background: rgba(254, 243, 199, 0.72);
    border-radius: 9999px;
    border: 1px solid rgba(245, 158, 11, 0.24);
}

.high-score-icon {
    font-size: 0.875rem;
}

.high-score-text {
    font-size: 0.875rem;
    font-weight: 700;
    color: #b45309;
}

.category-empty-state {
    border-radius: 1rem;
    border: 1px solid var(--category-border);
    background: var(--category-surface);
    padding: 2rem 1.25rem;
    text-align: center;
}

.empty-title {
    margin: 0 0 0.5rem;
    font-size: 1.125rem;
    font-weight: 700;
    color: var(--category-ink);
}

.empty-subtitle {
    margin: 0;
    font-size: 0.9375rem;
    line-height: 1.55;
    color: var(--category-body);
}

@media (prefers-reduced-motion: reduce) {
    .category-card,
    .category-arrow-icon {
        transition: none;
    }

    .category-card:hover,
    .category-card:active,
    .category-card:hover .category-arrow-icon {
        transform: none;
    }
}

@media (max-width: 640px) {
    .category-container {
        padding: 0.25rem 0;
    }

    .category-header {
        padding: 1rem 0.875rem 1.25rem;
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
        padding: 1.125rem 1rem;
        min-height: 0;
    }

    .category-icon {
        width: 2.75rem;
        height: 2.75rem;
        font-size: 1.75rem;
    }

    .category-name-th {
        font-size: 1.125rem;
    }

    .category-card-main {
        gap: 0.875rem;
    }
}
</style>
