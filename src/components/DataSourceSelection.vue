<template>
    <div :class="['datasource-container', toneClass]">
        <div class="datasource-header">
            <h2 class="datasource-title">{{ categoryName }}</h2>
            <p class="datasource-subtitle">{{ resolvedSubtitle }}</p>
            <p v-if="description" class="datasource-description">{{ description }}</p>
            <div v-if="dataSources.length > 0" class="datasource-summary" aria-label="Data source summary">
                <p class="summary-pill">{{ dataSources.length }} แหล่งข้อมูล</p>
            </div>
        </div>

        <div v-if="dataSources.length > 0" class="datasource-grid">
            <button
                v-for="(dataSource, index) in dataSources"
                :key="index"
                @click="selectDataSource(index)"
                type="button"
                class="datasource-card"
                :aria-label="`เลือกแหล่งข้อมูล ${dataSource.nameTh || `แหล่งข้อมูล ${index + 1}`}`"
            >
                <div class="datasource-card-main">
                    <div class="datasource-icon" aria-hidden="true">📚</div>
                    <div class="datasource-copy">
                        <div class="datasource-name-th">{{ dataSource.nameTh || `แหล่งข้อมูล ${index + 1}` }}</div>
                        <div class="datasource-name-en">{{ dataSource.nameEn || `Data Source ${index + 1}` }}</div>
                    </div>
                    <div class="datasource-arrow" aria-hidden="true">
                        <svg
                            xmlns="http://www.w3.org/2000/svg"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor"
                            class="datasource-arrow-icon"
                            focusable="false"
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

                <div v-if="actionLabel" class="datasource-card-meta">
                    <span class="datasource-action-label">{{ actionLabel }}</span>
                </div>
            </button>
        </div>

        <div v-else class="datasource-empty-state" role="status" aria-live="polite">
            <p class="empty-title">ยังไม่พบแหล่งข้อมูลสำหรับหมวดนี้</p>
            <p class="empty-subtitle">No data sources are available right now.</p>
        </div>
    </div>
</template>

<script setup lang="ts">
import { computed } from "vue";
import type { DataSource } from "../types/flashcard";

interface Props {
    categoryName: string;
    dataSources: DataSource[];
    subtitle?: string;
    description?: string;
    actionLabel?: string;
    tone?: "blue" | "indigo";
}

const props = withDefaults(defineProps<Props>(), {
    subtitle: "เลือกแหล่งข้อมูล / Select Data Source",
    description: "",
    actionLabel: "",
    tone: "blue",
});

const resolvedSubtitle = computed(() => props.subtitle);
const description = computed(() => props.description);
const actionLabel = computed(() => props.actionLabel);
const toneClass = computed(() => `datasource-container--${props.tone}`);

const emit = defineEmits<{
    select: [dataSourceIndex: number];
}>();

const selectDataSource = (dataSourceIndex: number) => {
    emit("select", dataSourceIndex);
};
</script>

<style scoped>
.datasource-container {
    --source-ink: #1f2937;
    --source-body: #4b5563;
    --source-muted: #6b7280;
    --source-border: #e5e7eb;
    --source-surface: #ffffff;
    --source-surface-soft: #f9fafb;
    --source-accent: #3b82f6;
    --source-focus: rgba(37, 99, 235, 0.24);

    min-height: 0;
    display: flex;
    flex-direction: column;
    padding: 0.5rem 0 0.75rem;
    max-width: 760px;
    width: 100%;
    margin: 0 auto;
}

.datasource-container--indigo {
    --source-accent: #6366f1;
    --source-focus: rgba(79, 70, 229, 0.22);
}

.datasource-header {
    text-align: center;
    margin-bottom: 1.25rem;
    padding: 1.25rem 1rem 1.375rem;
    border-radius: 1rem;
    border: 1px solid rgba(229, 231, 235, 0.82);
    background: linear-gradient(180deg, rgba(255, 255, 255, 0.92) 0%, rgba(243, 244, 246, 0.92) 100%);
}

.datasource-title {
    font-size: clamp(1.5rem, 4vw, 2rem);
    font-weight: 700;
    color: var(--source-ink);
    margin: 0;
    margin-bottom: 0.5rem;
    line-height: 1.25;
    overflow-wrap: anywhere;
}

.datasource-subtitle {
    font-size: 1rem;
    color: var(--source-body);
    margin: 0;
    line-height: 1.6;
}

.datasource-description {
    margin: 0.75rem auto 0;
    max-width: 54ch;
    font-size: 0.95rem;
    line-height: 1.6;
    color: var(--source-muted);
}

.datasource-summary {
    display: flex;
    justify-content: center;
    margin-top: 0.875rem;
}

.summary-pill {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-height: 2rem;
    padding: 0.35rem 0.75rem;
    border-radius: 999px;
    border: 1px solid color-mix(in srgb, var(--source-accent) 18%, white);
    background: color-mix(in srgb, var(--source-accent) 8%, white);
    color: color-mix(in srgb, var(--source-accent) 82%, black);
    font-size: 0.9rem;
    font-weight: 600;
    margin: 0;
}

.datasource-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1rem;
    width: 100%;
}

.datasource-card {
    background: var(--source-surface);
    border: 1px solid var(--source-border);
    border-radius: 1rem;
    padding: 1.125rem 1rem;
    cursor: pointer;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        border-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
    display: flex;
    flex-direction: column;
    align-items: stretch;
    gap: 0.875rem;
    text-align: left;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
    min-height: 88px;
    -webkit-tap-highlight-color: transparent;
    width: 100%;
}

@media (hover: hover) {
    .datasource-card:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 14px color-mix(in srgb, var(--source-accent) 20%, transparent);
        border-color: var(--source-accent);
        background: var(--source-surface-soft);
    }

    .datasource-card:hover .datasource-arrow-icon {
        transform: translateX(4px);
        color: var(--source-accent);
    }
}

.datasource-card:active {
    transform: translateY(0);
    background: var(--source-surface-soft);
}

.datasource-card:focus-visible {
    outline: 3px solid var(--source-focus);
    outline-offset: 2px;
}

.datasource-card-main {
    display: flex;
    align-items: center;
    gap: 0.875rem;
}

.datasource-icon {
    font-size: 1.875rem;
    width: 2.75rem;
    height: 2.75rem;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border-radius: 0.875rem;
    background: color-mix(in srgb, var(--source-accent) 8%, white);
    flex-shrink: 0;
}

.datasource-copy {
    flex: 1;
    min-width: 0;
}

.datasource-arrow {
    flex-shrink: 0;
    color: #94a3b8;
}

.datasource-arrow-icon {
    width: 1.25rem;
    height: 1.25rem;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.datasource-card-meta {
    display: flex;
    align-items: center;
    margin-top: auto;
}

.datasource-action-label {
    display: inline-flex;
    align-items: center;
    min-height: 2rem;
    padding: 0.35rem 0.75rem;
    border-radius: 999px;
    border: 1px solid color-mix(in srgb, var(--source-accent) 18%, white);
    background: color-mix(in srgb, var(--source-accent) 8%, white);
    color: color-mix(in srgb, var(--source-accent) 82%, black);
    font-size: 0.85rem;
    font-weight: 600;
    line-height: 1.2;
}

.datasource-name-th {
    font-size: 1.125rem;
    font-weight: 600;
    color: var(--source-ink);
    margin-bottom: 0.25rem;
    line-height: 1.4;
    overflow-wrap: anywhere;
}

.datasource-name-en {
    font-size: 0.875rem;
    color: var(--source-muted);
    line-height: 1.45;
    overflow-wrap: anywhere;
}

.datasource-empty-state {
    border-radius: 1rem;
    border: 1px solid var(--source-border);
    background: var(--source-surface);
    padding: 1.75rem 1.125rem;
    text-align: center;
}

.empty-title {
    margin: 0 0 0.5rem;
    font-size: 1.125rem;
    font-weight: 700;
    color: var(--source-ink);
}

.empty-subtitle {
    margin: 0;
    font-size: 0.9375rem;
    line-height: 1.55;
    color: var(--source-body);
}

@media (max-width: 640px) {
    .datasource-container {
        padding: 0.25rem 0 0.5rem;
    }

    .datasource-header {
        padding: 1rem 0.875rem 1.125rem;
    }

    .datasource-title {
        font-size: 1.5rem;
    }

    .datasource-subtitle {
        font-size: 1rem;
    }

    .datasource-grid {
        grid-template-columns: 1fr;
        gap: 1rem;
    }

    .datasource-card {
        padding: 1rem;
    }

    .datasource-icon {
        width: 2.5rem;
        height: 2.5rem;
        font-size: 1.625rem;
    }

    .datasource-name-th {
        font-size: 1.05rem;
    }
}

@media (prefers-reduced-motion: reduce) {
    .datasource-card,
    .datasource-arrow-icon {
        transition: none;
    }

    .datasource-card:hover,
    .datasource-card:active,
    .datasource-card:hover .datasource-arrow-icon {
        transform: none;
    }
}
</style>
