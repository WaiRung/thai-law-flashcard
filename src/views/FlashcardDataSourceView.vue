<template>
    <main class="main-content">
        <LoadingSpinner v-if="isLoading" message="กำลังโหลดข้อมูล..." />
        <template v-else>
            <div v-if="isUsingFallback && !loadError" class="warning-banner" role="status" aria-live="polite">
                <div class="warning-copy">
                    <p class="warning-title">กำลังใช้ข้อมูลออฟไลน์</p>
                    <p class="warning-text">ไม่สามารถโหลดข้อมูลล่าสุดจาก API ได้ในขณะนี้ ระบบจะแสดงแหล่งข้อมูลจากข้อมูลที่บันทึกไว้ในเครื่อง</p>
                </div>
            </div>

            <section v-if="loadError" class="status-panel" role="alert" aria-live="assertive">
                <h2 class="status-title">ไม่สามารถเปิดหน้าแหล่งข้อมูลได้</h2>
                <p class="status-copy">{{ loadError }}</p>
                <div class="status-actions">
                    <button type="button" class="action-btn action-btn-primary" @click="loadCategoryData">
                        โหลดอีกครั้ง
                    </button>
                    <button type="button" class="action-btn action-btn-secondary" @click="goBackToCategories">
                        กลับไปหน้าเลือกหมวด
                    </button>
                </div>
            </section>

            <section v-else-if="category && !hasDataSources" class="status-panel" role="status" aria-live="polite">
                <h2 class="status-title">ยังไม่มีแหล่งข้อมูลให้เลือก</h2>
                <p class="status-copy">หมวดนี้ยังไม่มีข้อมูลแยกแหล่งอ้างอิงในตอนนี้ กรุณากลับไปเลือกหมวดอื่น</p>
                <div class="status-actions">
                    <button type="button" class="action-btn action-btn-secondary" @click="goBackToCategories">
                        กลับไปหน้าเลือกหมวด
                    </button>
                </div>
            </section>

            <DataSourceSelection
                v-else-if="category"
                :category-name="category.nameTh"
                :data-sources="category.dataSources || []"
                @select="selectDataSource"
            />

            <section v-else class="status-panel" role="status" aria-live="polite">
                <h2 class="status-title">ไม่พบหมวดหมู่ที่เลือก</h2>
                <p class="status-copy">ระบบไม่พบหมวดหมู่ที่ต้องการเปิด กรุณาเลือกหมวดใหม่อีกครั้ง</p>
                <div class="status-actions">
                    <button type="button" class="action-btn action-btn-secondary" @click="goBackToCategories">
                        กลับไปหน้าเลือกหมวด
                    </button>
                </div>
            </section>
        </template>
    </main>
</template>

<script setup lang="ts">
import { computed, ref, onMounted, onUnmounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import DataSourceSelection from "../components/DataSourceSelection.vue";
import LoadingSpinner from "../components/LoadingSpinner.vue";
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
const category = ref<CategoryStore | null>(null);
const loadError = ref<string | null>(null);
const isUsingFallback = ref(false);

const hasDataSources = computed(() => {
    return Boolean(category.value?.dataSources && category.value.dataSources.length > 0);
});

const goBackToCategories = () => {
    router.push({ name: "flashcard-categories" });
};

const loadCategoryData = async () => {
    isLoading.value = true;
    loadError.value = null;
    isUsingFallback.value = false;

    try {
        const loadedCategories = await loadCategories();
        const categories = loadedCategories || categoryStores;

        if (!loadedCategories) {
            isUsingFallback.value = true;
        }

        category.value = categories.find((c) => c.id === categoryId) || null;

        if (!category.value) {
            loadError.value = "ระบบไม่พบหมวดหมู่ที่คุณเลือก อาจมีการเปลี่ยนแปลงข้อมูลหรือลิงก์หมดอายุ";
            return;
        }

        if (!category.value.dataSources || category.value.dataSources.length === 0) {
            return;
        }

        if (category.value.dataSources.length === 1) {
            selectDataSource(0);
            return;
        }

        setHeader(category.value.nameTh, category.value.nameEn);
    } catch (err) {
        loadError.value = "ไม่สามารถโหลดข้อมูลหมวดหมู่ได้ในขณะนี้ กรุณาลองใหม่อีกครั้ง";
    } finally {
        isLoading.value = false;
    }
};

const selectDataSource = (dataSourceIndex: number) => {
    router.push({
        name: "flashcards",
        params: { categoryId, dataSourceIndex: String(dataSourceIndex) },
    });
};

onMounted(async () => {
    setHeader("เลือกแหล่งข้อมูล", "Select data source");
    await loadCategoryData();
});

onUnmounted(() => {
    resetHeader();
});
</script>

<style scoped>
.main-content {
    --flashcard-warning-bg: #fef3c7;
    --flashcard-warning-border: #f59e0b;
    --flashcard-warning-ink: #78350f;
    --flashcard-status-ink: #1f2937;
    --flashcard-status-body: #4b5563;
    --flashcard-status-border: #e5e7eb;
    --flashcard-status-surface: #ffffff;
    --flashcard-accent: #4f46e5;
    --flashcard-accent-deep: #4338ca;
    --flashcard-focus: rgba(79, 70, 229, 0.24);

    flex: 1;
    padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
    max-width: 760px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.warning-banner {
    display: flex;
    align-items: flex-start;
    gap: 0.75rem;
    padding: 1rem 1rem 1.125rem;
    background-color: var(--flashcard-warning-bg);
    border: 1px solid rgba(245, 158, 11, 0.26);
    border-radius: 1rem;
    margin-bottom: 1rem;
    color: var(--flashcard-warning-ink);
    animation: slide-in 0.24s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.warning-copy {
    flex: 1;
}

.warning-title {
    margin: 0 0 0.25rem;
    font-size: 0.95rem;
    font-weight: 700;
    line-height: 1.4;
}

.warning-text {
    font-size: 0.875rem;
    font-weight: 600;
    line-height: 1.5;
    margin: 0;
}

.status-panel {
    border: 1px solid var(--flashcard-status-border);
    border-radius: 1rem;
    background: var(--flashcard-status-surface);
    padding: 1.5rem 1.125rem;
    margin-top: 0.25rem;
}

.status-title {
    margin: 0;
    color: var(--flashcard-status-ink);
    font-size: 1.25rem;
    font-weight: 700;
    line-height: 1.35;
}

.status-copy {
    margin: 0.625rem 0 0;
    color: var(--flashcard-status-body);
    font-size: 0.95rem;
    line-height: 1.6;
    max-width: 65ch;
}

.status-actions {
    display: flex;
    gap: 0.75rem;
    flex-wrap: wrap;
    margin-top: 1.125rem;
}

.action-btn {
    min-height: 2.75rem;
    border-radius: 0.75rem;
    border: 1px solid transparent;
    padding: 0.625rem 1rem;
    font-size: 0.95rem;
    font-weight: 600;
    cursor: pointer;
    transition:
        background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        border-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.action-btn-primary {
    color: #ffffff;
    background: linear-gradient(135deg, var(--flashcard-accent) 0%, var(--flashcard-accent-deep) 100%);
    box-shadow: 0 4px 12px rgba(79, 70, 229, 0.24);
}

.action-btn-secondary {
    color: var(--flashcard-status-ink);
    background: #f9fafb;
    border-color: var(--flashcard-status-border);
}

@media (hover: hover) {
    .action-btn-primary:hover {
        transform: translateY(-1px);
        box-shadow: 0 7px 16px rgba(79, 70, 229, 0.3);
    }

    .action-btn-secondary:hover {
        background-color: #f3f4f6;
        border-color: #d1d5db;
    }
}

.action-btn:focus-visible {
    outline: 3px solid var(--flashcard-focus);
    outline-offset: 2px;
}

.action-btn:active {
    transform: translateY(0);
}

@keyframes slide-in {
    from {
        opacity: 0;
        transform: translateY(-8px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@media (max-width: 640px) {
    .main-content {
        padding: 0.875rem max(0.75rem, env(safe-area-inset-right, 0px)) 1rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .status-panel {
        padding: 1.25rem 1rem;
    }

    .status-title {
        font-size: 1.125rem;
    }

    .status-copy {
        font-size: 0.9375rem;
    }

    .status-actions {
        flex-direction: column;
    }

    .action-btn {
        width: 100%;
    }
}

@media (prefers-reduced-motion: reduce) {
    .warning-banner {
        animation: none;
    }

    .action-btn {
        transition: none;
    }

    .action-btn:hover,
    .action-btn:active {
        transform: none;
    }
}
</style>
