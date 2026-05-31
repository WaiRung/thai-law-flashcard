<template>
    <main class="main-content">
        <LoadingSpinner v-if="isLoading" message="กำลังโหลดหมวดหมู่..." />
        <template v-else>
            <!-- Warning Banner for Fallback -->
            <div v-if="error && isUsingFallback" class="warning-banner" role="status" aria-live="polite">
                <span class="warning-icon">⚠️</span>
                <span class="warning-text">{{ error }}</span>
            </div>

            <CategorySelection
                :categories="categoryList"
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
import { useDataManager } from "../composables/useDataManager";
import { useHeader } from "../composables/useHeader";
import type { CategoryStore } from "../types/flashcard";

const router = useRouter();
const { setHeader, resetHeader } = useHeader();

// Category Management
const categories = ref<CategoryStore[]>([]);
const filteredCounts = ref<Record<string, number>>({});
const isLoading = ref(false);
const error = ref<string | null>(null);
const isUsingFallback = ref(false);
let warningTimeout: ReturnType<typeof setTimeout> | null = null;

// Use data manager composable
const { loadCategories } = useDataManager();

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

    if (warningTimeout) {
        clearTimeout(warningTimeout);
        warningTimeout = null;
    }

    try {
        // Use the composable to load categories
        const loadedCategories = await loadCategories();
        
        if (loadedCategories) {
            categories.value = loadedCategories;
        } else {
            // Fall back to static data if loading fails
            console.warn("Using static data as fallback");
            categories.value = categoryStores;
            isUsingFallback.value = true;
            error.value = "ไม่สามารถโหลดข้อมูลจาก API ได้ กำลังใช้ข้อมูลแบบออฟไลน์";
            
            // Clear fallback warning after a short delay to reduce visual noise.
            warningTimeout = setTimeout(() => {
                error.value = null;
            }, 5000);
        }
    } catch (err) {
        // Fall back to static data if API fails
        console.warn("Failed to load categories, using static data:", err);
        categories.value = categoryStores;
        isUsingFallback.value = true;
        error.value = "ไม่สามารถโหลดข้อมูลจาก API ได้ กำลังใช้ข้อมูลแบบออฟไลน์";
        
        warningTimeout = setTimeout(() => {
            error.value = null;
        }, 5000);
    } finally {
        isLoading.value = false;
    }
};

// Category Selection Method
const selectCategory = (categoryId: string) => {
    // Check if category has multiple data sources
    const category = categories.value.find(c => c.id === categoryId);
    
    if (category?.dataSources && category.dataSources.length > 1) {
        // Navigate to data source selection view
        router.push({ name: "flashcard-datasources", params: { categoryId } });
    } else {
        // Navigate to flashcard view using router
        router.push({ name: "flashcards", params: { categoryId } });
    }
};

// Initialize categories on mount
onMounted(async () => {
    // Set header to indicate Flashcards mode
    setHeader("Flashcards", "เลือกหมวดหมู่");

    // Load categories
    await loadCategoriesData();
    
    // Calculate filtered counts for each category
    await Promise.all(categories.value.map(async (store) => {
        const filtered = await filterQuestions(store.id, store.questions);
        filteredCounts.value[store.id] = filtered.length;
    }));
});

// Reset header on unmount
onUnmounted(() => {
    if (warningTimeout) {
        clearTimeout(warningTimeout);
        warningTimeout = null;
    }

    resetHeader();
});
</script>

<style scoped>
.main-content {
    --category-warning-bg: #fef3c7;
    --category-warning-border: #f59e0b;
    --category-warning-ink: #78350f;

    flex: 1;
    padding: 1.5rem 1rem;
    max-width: 600px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.warning-banner {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0.875rem 1rem;
    background-color: var(--category-warning-bg);
    border: 1px solid var(--category-warning-border);
    border-radius: 0.5rem;
    margin-bottom: 1.5rem;
    color: var(--category-warning-ink);
    animation: slideIn 0.28s cubic-bezier(0.2, 0.8, 0.2, 1);
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

.warning-icon {
    font-size: 1.25rem;
    flex-shrink: 0;
}

.warning-text {
    font-size: 0.875rem;
    font-weight: 600;
    line-height: 1.5;
    flex: 1;
}

@media (prefers-reduced-motion: reduce) {
    .warning-banner {
        animation: none;
    }
}

@media (max-width: 640px) {
    .main-content {
        padding: 1rem 0.75rem;
    }
}
</style>
