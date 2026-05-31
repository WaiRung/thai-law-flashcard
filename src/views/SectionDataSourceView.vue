<template>
    <main class="main-content">
        <LoadingSpinner v-if="isLoading" message="กำลังโหลดข้อมูล..." />
        <ErrorMessage
            v-else-if="loadError"
            :message="loadError"
            @retry="loadCategoryData"
        />
        <template v-else-if="category">
            <DataSourceSelection
                :category-name="category.nameTh"
                :data-sources="category.dataSources || []"
                subtitle="เลือกแหล่งข้อมูลเพื่อเปิดรายการมาตรา"
                description="ระบบจะแสดงแหล่งข้อมูลทั้งหมดที่มีสำหรับหมวดนี้ เพื่อให้เลือกชุดเนื้อหาที่ต้องการอ่าน"
                action-label="เลือกแหล่งข้อมูล"
                @select="selectDataSource"
            />
        </template>
        <section v-else class="not-found-state" role="status" aria-live="polite">
            <h2 class="not-found-title">ไม่พบหมวดกฎหมายที่เลือก</h2>
            <p class="not-found-description">
                อาจเป็นลิงก์ที่ไม่ถูกต้อง หรือข้อมูลถูกปรับปรุงแล้ว
            </p>
            <div class="not-found-actions">
                <button type="button" class="action-button action-button--primary" @click="goToSectionsList">
                    กลับไปเลือกหมวดกฎหมาย
                </button>
                <button type="button" class="action-button action-button--secondary" @click="loadCategoryData">
                    โหลดข้อมูลอีกครั้ง
                </button>
            </div>
        </section>
    </main>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import DataSourceSelection from "../components/DataSourceSelection.vue";
import LoadingSpinner from "../components/LoadingSpinner.vue";
import ErrorMessage from "../components/ErrorMessage.vue";
import { categoryStores } from "../data/categoryStores";
import { useDataManager } from "../composables/useDataManager";
import { useHeader } from "../composables/useHeader";
import type { CategoryStore } from "../types/flashcard";

const router = useRouter();
const route = useRoute();
const { setHeader, resetHeader } = useHeader();
const { loadCategories } = useDataManager();

const categoryId = route.params.categoryId as string;
const isLoading = ref(true);
const loadError = ref<string | null>(null);
const category = ref<CategoryStore | null>(null);

const loadCategoryData = async () => {
    isLoading.value = true;
    loadError.value = null;

    try {
        const loadedCategories = await loadCategories();
        const categories = loadedCategories || categoryStores;

        category.value = categories.find((c) => c.id === categoryId) || null;

        if (!category.value) {
            console.warn(`Category not found: ${categoryId}`);
            return;
        }

        // Set header
        setHeader(category.value.nameTh, category.value.nameEn);
    } catch (err) {
        console.error("Failed to load category:", err);
        loadError.value = "ไม่สามารถโหลดข้อมูลหมวดกฎหมายได้ในขณะนี้ กรุณาลองอีกครั้ง";
    } finally {
        isLoading.value = false;
    }
};

const goToSectionsList = () => {
    router.push({ name: "sections-list" });
};

const selectDataSource = (dataSourceIndex: number) => {
    router.push({
        name: "sections-detail",
        params: { categoryId, dataSourceIndex: String(dataSourceIndex) },
    });
};

onMounted(async () => {
    setHeader("เลือกแหล่งข้อมูล", "Section Data Sources");
    await loadCategoryData();
});

onUnmounted(() => {
    resetHeader();
});
</script>

<style scoped>
.main-content {
    --section-source-ink: #1f2937;
    --section-source-body: #4b5563;
    --section-source-border: #e5e7eb;
    --section-source-surface: #ffffff;
    --section-source-soft: #f8fafc;
    --section-source-accent: #3b82f6;
    --section-source-focus: rgba(37, 99, 235, 0.24);

    flex: 1;
    padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
    max-width: 600px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.not-found-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.75rem;
    text-align: center;
    padding: 2rem 1rem;
    background: var(--section-source-soft);
    border: 1px solid var(--section-source-border);
    border-radius: 1rem;
}

.not-found-title {
    margin: 0;
    color: var(--section-source-ink);
    font-size: 1.125rem;
    line-height: 1.4;
}

.not-found-description {
    margin: 0;
    color: var(--section-source-body);
    font-size: 0.9375rem;
    line-height: 1.6;
    max-width: 48ch;
}

.not-found-actions {
    width: 100%;
    margin-top: 0.25rem;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 0.75rem;
}

.action-button {
    min-height: 44px;
    padding: 0.75rem 1rem;
    border-radius: 0.75rem;
    border: 1px solid transparent;
    font-size: 0.9375rem;
    font-weight: 600;
    cursor: pointer;
    transition:
        background-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        border-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        box-shadow 0.2s cubic-bezier(0.25, 1, 0.5, 1);
}

.action-button--primary {
    background: var(--section-source-accent);
    color: #ffffff;
}

.action-button--secondary {
    background: var(--section-source-surface);
    border-color: var(--section-source-accent);
    color: var(--section-source-accent);
}

@media (hover: hover) {
    .action-button--primary:hover {
        background: #2563eb;
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.25);
    }

    .action-button--secondary:hover {
        background: #eff6ff;
    }
}

.action-button:focus-visible {
    outline: 3px solid var(--section-source-focus);
    outline-offset: 2px;
}

.action-button:active {
    box-shadow: none;
}

@media (max-width: 640px) {
    .main-content {
        padding: 0.875rem max(0.75rem, env(safe-area-inset-right, 0px)) 1rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .not-found-state {
        padding: 1.5rem 0.875rem;
    }

    .not-found-actions {
        flex-direction: column;
    }

    .action-button {
        width: 100%;
    }
}

@media (prefers-reduced-motion: reduce) {
    .action-button {
        transition: none;
    }
}
</style>
