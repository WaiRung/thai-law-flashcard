<template>
    <main class="main-content">
        <LoadingSpinner v-if="isLoading" message="กำลังโหลดหมวดหมู่..." />
        <template v-else>
            <div v-if="isUsingFallback" class="warning-banner" role="status" aria-live="polite">
                <div class="warning-copy">
                    <p class="warning-title">กำลังใช้ข้อมูลออฟไลน์</p>
                    <p class="warning-text">{{ error }}</p>
                </div>
            </div>

            <CategorySelection
                :categories="categoryList"
                title="เลือกหมวดแฟลชการ์ด"
                subtitle="Choose a flashcard category"
                description="เลือกหมวดที่ต้องการทบทวนเพื่อเปิดชุดคำถามตามเนื้อหากฎหมายแต่ละกลุ่ม"
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

// Use data manager composable
const { loadCategories } = useDataManager();

// Build category list from loaded categories
const categoryList = computed(() =>
    categories.value.map((store: CategoryStore) => ({
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
            console.warn("Using static data as fallback");
            categories.value = categoryStores;
            isUsingFallback.value = true;
            error.value = "ไม่สามารถโหลดข้อมูลจาก API ได้ในขณะนี้ ระบบจะแสดงหมวดหมู่จากข้อมูลที่บันทึกไว้ในเครื่อง";
        }
    } catch (err) {
        // Fall back to static data if API fails
        console.warn("Failed to load categories, using static data:", err);
        categories.value = categoryStores;
        isUsingFallback.value = true;
        error.value = "ไม่สามารถโหลดข้อมูลจาก API ได้ในขณะนี้ ระบบจะแสดงหมวดหมู่จากข้อมูลที่บันทึกไว้ในเครื่อง";
    } finally {
        isLoading.value = false;
    }
};

// Category Selection Method
const selectCategory = (categoryId: string) => {
    // Check if category has multiple data sources
    const category = categories.value.find((c: CategoryStore) => c.id === categoryId);
    
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
    await Promise.all(categories.value.map(async (store: CategoryStore) => {
        const filtered = await filterQuestions(store.id, store.questions);
        filteredCounts.value[store.id] = filtered.length;
    }));
});

// Reset header on unmount
onUnmounted(() => {
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
    align-items: flex-start;
    gap: 0.75rem;
    padding: 1rem 1rem 1.125rem;
    background-color: var(--category-warning-bg);
    border: 1px solid rgba(245, 158, 11, 0.26);
    border-radius: 1rem;
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
