<template>
    <main class="main-content" :aria-busy="isLoading ? 'true' : 'false'">
        <div v-if="isLoading" class="loading-skeleton" aria-hidden="true">
            <div class="skeleton-header">
                <div class="skeleton-title shimmer"></div>
                <div class="skeleton-subtitle shimmer"></div>
                <div class="skeleton-subtitle skeleton-subtitle-short shimmer"></div>
            </div>

            <div class="skeleton-grid">
                <div v-for="n in 4" :key="n" class="skeleton-card shimmer"></div>
            </div>

            <LoadingSpinner message="กำลังโหลดหมวดหมู่..." />
        </div>
        <template v-else>
            <!-- Warning Banner for Fallback -->
            <div
                v-if="error && isUsingFallback"
                class="warning-banner"
                role="status"
                aria-live="polite"
            >
                <span class="warning-icon">⚠️</span>
                <span class="warning-text">{{ error }}</span>
                <button
                    type="button"
                    class="warning-dismiss"
                    aria-label="ปิดข้อความแจ้งเตือน"
                    @click="dismissWarning"
                >
                    ปิด
                </button>
            </div>

            <CategorySelection
                :categories="categoryList"
                :high-scores="highScores"
                @select="selectCategory"
            />
        </template>
    </main>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";
import { useRouter } from "vue-router";
import CategorySelection from "../components/CategorySelection.vue";
import LoadingSpinner from "../components/LoadingSpinner.vue";
import { categoryStores } from "../data/categoryStores";
import { filterQuestions } from "../services/filterService";
import { getAllHighScores } from "../services/highScoreService";
import { useDataManager } from "../composables/useDataManager";
import { useHeader } from "../composables/useHeader";
import type { CategoryStore } from "../types/flashcard";
import type { HighScore } from "../types/quiz";

const router = useRouter();
const { setHeader, resetHeader } = useHeader();

// Category Management
const categories = ref<CategoryStore[]>([]);
const filteredCounts = ref<Record<string, number>>({});
const highScores = ref<Map<string, HighScore>>(new Map());
const isLoading = ref(false);
const error = ref<string | null>(null);
const isUsingFallback = ref(false);
let warningTimeout: ReturnType<typeof setTimeout> | null = null;

// Use data manager composable
const { loadCategories } = useDataManager();

const clearWarningTimer = () => {
    if (warningTimeout) {
        clearTimeout(warningTimeout);
        warningTimeout = null;
    }
};

const dismissWarning = () => {
    clearWarningTimer();
    error.value = null;
};

const showFallbackWarning = () => {
    error.value = "ไม่สามารถโหลดข้อมูลจาก API ได้ กำลังใช้ข้อมูลแบบออฟไลน์";
    clearWarningTimer();
    warningTimeout = setTimeout(() => {
        error.value = null;
        warningTimeout = null;
    }, 5000);
};

// Build category list from loaded categories
const categoryList = computed(() =>
    categories.value.map((store) => ({
        id: store.id,
        nameTh: store.nameTh,
        nameEn: store.nameEn,
        icon: store.icon,
        count: filteredCounts.value[store.id] ?? 0,
    })),
);

// Load categories on component mount
const loadCategoriesData = async () => {
    isLoading.value = true;
    error.value = null;
    isUsingFallback.value = false;

    try {
        // Use the composable to load categories
        const loadedCategories = await loadCategories();
        
        if (loadedCategories) {
            categories.value = loadedCategories;
        } else {
            // Fall back to static data if loading fails
            categories.value = categoryStores;
            isUsingFallback.value = true;
            showFallbackWarning();
        }
    } catch (err) {
        // Fall back to static data if API fails
        void err;
        categories.value = categoryStores;
        isUsingFallback.value = true;
        showFallbackWarning();
    } finally {
        isLoading.value = false;
    }
};

// Load high scores
const loadHighScores = async () => {
    try {
        highScores.value = await getAllHighScores();
    } catch (err) {
        void err;
        highScores.value = new Map();
    }
};

// Category Selection Method
const selectCategory = (categoryId: string) => {
    // Check if category has multiple data sources
    const category = categories.value.find(c => c.id === categoryId);
    
    if (category?.dataSources && category.dataSources.length > 1) {
        // Navigate to data source selection view
        router.push({ name: "quizlaw-datasources", params: { categoryId } });
    } else {
        // Navigate to quiz view using router
        router.push({ name: "quizlaw", params: { categoryId } });
    }
};

// Initialize categories on mount
onMounted(async () => {
    // Set header to indicate QuizLaw mode
    setHeader("QuizLaw", "เลือกหมวดหมู่");

    // Load categories and high scores in parallel
    await Promise.all([
        loadCategoriesData(),
        loadHighScores(),
    ]);
    
    // Calculate filtered counts for each category
    await Promise.all(categories.value.map(async (store) => {
        const filtered = await filterQuestions(store.id, store.questions);
        filteredCounts.value[store.id] = filtered.length;
    }));
});

// Reset header on unmount
onUnmounted(() => {
    clearWarningTimer();
    resetHeader();
});
</script>

<style scoped>
.main-content {
    flex: 1;
    padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
    max-width: 600px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.loading-skeleton {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.skeleton-header {
    border: 1px solid rgba(229, 231, 235, 0.82);
    border-radius: 1rem;
    padding: 1.25rem 1rem 1.5rem;
    background: linear-gradient(180deg, rgba(255, 255, 255, 0.92) 0%, rgba(243, 244, 246, 0.92) 100%);
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.skeleton-title {
    height: 2rem;
    border-radius: 0.625rem;
    width: 62%;
}

.skeleton-subtitle {
    height: 0.95rem;
    border-radius: 0.5rem;
    width: 78%;
}

.skeleton-subtitle-short {
    width: 54%;
}

.skeleton-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1rem;
}

.skeleton-card {
    height: 168px;
    border-radius: 1rem;
    border: 1px solid #e5e7eb;
}

.shimmer {
    background: linear-gradient(
        90deg,
        rgba(229, 231, 235, 0.88) 0%,
        rgba(243, 244, 246, 0.98) 45%,
        rgba(229, 231, 235, 0.88) 100%
    );
    background-size: 220% 100%;
    animation: skeletonShimmer 1.15s cubic-bezier(0.16, 1, 0.3, 1) infinite;
}

.warning-banner {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0.875rem 1rem;
    background-color: #fffbeb;
    border: 1px solid #fbbf24;
    border-radius: 0.5rem;
    margin-bottom: 1.5rem;
    color: #92400e;
    animation: slideIn 0.3s ease-out;
    line-height: 1.55;
}

@keyframes slideIn {
    from {
        opacity: 0;
        transform: translateY(-10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes skeletonShimmer {
    from {
        background-position: 100% 0;
    }
    to {
        background-position: -100% 0;
    }
}

.warning-icon {
    font-size: 1.25rem;
    flex-shrink: 0;
}

.warning-text {
    font-size: 0.9rem;
    font-weight: 500;
    flex: 1;
    overflow-wrap: anywhere;
}

.warning-dismiss {
    border: 1px solid rgba(146, 64, 14, 0.28);
    border-radius: 999px;
    background: rgba(255, 255, 255, 0.45);
    color: #78350f;
    font-size: 0.8125rem;
    line-height: 1;
    font-weight: 600;
    padding: 0.425rem 0.7rem;
    cursor: pointer;
    transition: background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

@media (hover: hover) {
    .warning-dismiss:hover {
        background: rgba(255, 255, 255, 0.72);
    }
}

.warning-dismiss:focus-visible {
    outline: 2px solid rgba(146, 64, 14, 0.35);
    outline-offset: 2px;
}

@media (max-width: 640px) {
    .main-content {
        padding: 0.875rem max(0.75rem, env(safe-area-inset-right, 0px)) 1rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .skeleton-header {
        padding: 1rem 0.875rem 1.25rem;
    }

    .skeleton-title {
        height: 1.75rem;
        width: 72%;
    }

    .skeleton-grid {
        grid-template-columns: 1fr;
    }

    .skeleton-card {
        height: 144px;
    }

    .warning-banner {
        gap: 0.5rem;
        margin-bottom: 1rem;
        padding: 0.75rem;
    }

    .warning-text {
        font-size: 0.8125rem;
    }
}

@media (prefers-reduced-motion: reduce) {
    .warning-banner {
        animation: none;
    }

    .shimmer,
    .warning-dismiss {
        animation: none;
        transition: none;
    }
}
</style>
