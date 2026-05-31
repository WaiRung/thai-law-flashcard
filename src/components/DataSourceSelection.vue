<template>
    <div class="datasource-container">
        <div class="datasource-header">
            <h2 class="datasource-title">{{ categoryName }}</h2>
            <p class="datasource-subtitle">เลือกแหล่งข้อมูล / Select Data Source</p>
        </div>

        <div class="datasource-grid">
            <button
                v-for="(dataSource, index) in dataSources"
                :key="index"
                @click="selectDataSource(index)"
                type="button"
                class="datasource-card"
                :aria-label="`เลือกแหล่งข้อมูล ${dataSource.nameTh || `แหล่งข้อมูล ${index + 1}`}`"
            >
                <div class="datasource-icon">📚</div>
                <div class="datasource-name-th">{{ dataSource.nameTh || `แหล่งข้อมูล ${index + 1}` }}</div>
                <div class="datasource-name-en">{{ dataSource.nameEn || `Data Source ${index + 1}` }}</div>
            </button>
        </div>
    </div>
</template>

<script setup lang="ts">
import type { DataSource } from "../types/flashcard";

interface Props {
    categoryName: string;
    dataSources: DataSource[];
}

defineProps<Props>();

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

.datasource-header {
    text-align: center;
    margin-bottom: 1.5rem;
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
    font-size: 1.125rem;
    color: var(--source-body);
    margin: 0;
    line-height: 1.6;
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
    padding: 1.5rem 1rem;
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
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
    min-height: 148px;
    -webkit-tap-highlight-color: transparent;
    width: 100%;
}

@media (hover: hover) {
    .datasource-card:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 14px rgba(59, 130, 246, 0.16);
        border-color: var(--source-accent);
        background: var(--source-surface-soft);
    }
}

.datasource-card:active {
    transform: translateY(-2px);
}

.datasource-card:focus-visible {
    outline: 3px solid var(--source-focus);
    outline-offset: 2px;
}

.datasource-icon {
    font-size: 2.5rem;
    margin-bottom: 0.875rem;
}

.datasource-name-th {
    font-size: 1.125rem;
    font-weight: 600;
    color: var(--source-ink);
    margin-bottom: 0.375rem;
    line-height: 1.4;
    overflow-wrap: anywhere;
}

.datasource-name-en {
    font-size: 0.875rem;
    color: var(--source-muted);
    line-height: 1.45;
    overflow-wrap: anywhere;
}

@media (max-width: 640px) {
    .datasource-container {
        padding: 0.25rem 0 0.5rem;
    }

    .datasource-subtitle {
        font-size: 1rem;
    }

    .datasource-grid {
        grid-template-columns: 1fr;
        gap: 1rem;
    }

    .datasource-card {
        padding: 1.5rem 1rem;
        min-height: 132px;
    }

    .datasource-icon {
        font-size: 2.25rem;
    }
}

@media (prefers-reduced-motion: reduce) {
    .datasource-card {
        transition: none;
    }
}
</style>
