<template>
    <main class="main-content">
        <LoadingSpinner v-if="isLoading" message="กำลังโหลดหมวดหมู่..." />
        <ErrorMessage
            v-else-if="loadError"
            :message="loadError"
            @retry="loadCategoriesWithSections"
        />
        <template v-else>
            <div class="sections-container">
                <div class="header-section">
                    <h2 class="title">เลือกหมวดกฎหมาย</h2>
                    <p class="subtitle">เลือกหมวดเพื่ออ่านรายการมาตราและแหล่งข้อมูลที่เกี่ยวข้อง</p>
                </div>

                <div
                    class="categories-list"
                    role="list"
                    aria-label="หมวดกฎหมายที่มีรายการมาตรา"
                >
                    <button
                        v-for="category in categoriesWithSections"
                        :key="category.categoryId"
                        @click="selectCategory(category.categoryId)"
                        type="button"
                        class="category-item"
                        role="listitem"
                        :aria-label="`${category.categoryName} มี ${category.sectionCount} มาตรา`"
                    >
                        <div class="category-icon" aria-hidden="true">📜</div>
                        <div class="category-content">
                            <div class="category-name">{{ category.categoryName }}</div>
                            <div class="category-count">{{ formatSectionCount(category.sectionCount) }}</div>
                        </div>
                        <div class="category-arrow" aria-hidden="true">
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                fill="none"
                                viewBox="0 0 24 24"
                                stroke="currentColor"
                                class="arrow-icon"
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
                    </button>
                </div>

                <div v-if="categoriesWithSections.length === 0" class="empty-state">
                    <h3 class="empty-title">ไม่พบหมวดกฎหมายในขณะนี้</h3>
                    <p class="empty-description">ลองรีเฟรชอีกครั้ง หรือกลับไปเลือกเมนูอื่นชั่วคราว</p>
                    <button
                        type="button"
                        class="empty-action"
                        @click="loadCategoriesWithSections"
                    >
                        โหลดหมวดหมู่อีกครั้ง
                    </button>
                </div>
            </div>
        </template>
    </main>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import { useRouter } from "vue-router";
import LoadingSpinner from "../components/LoadingSpinner.vue";
import ErrorMessage from "../components/ErrorMessage.vue";
import { getAllSections } from "../services/sectionService";
import { categoryStores } from "../data/categoryStores";
import { fetchCategories } from "../services/api";
import {
    getCategoriesCache,
    isCacheValid,
} from "../services/cache";
import { useHeader } from "../composables/useHeader";
import type { CategoryStore } from "../types/flashcard";

interface CategoryWithSectionCount {
    categoryId: string;
    categoryName: string;
    sectionCount: number;
}

const router = useRouter();
const { setHeader, resetHeader } = useHeader();
const isLoading = ref(true);
const loadError = ref<string | null>(null);
const categoriesWithSections = ref<CategoryWithSectionCount[]>([]);
const categories = ref<CategoryStore[]>([]);

// Load categories from cache or fallback to static data
const loadCategories = async () => {
    try {
        // First, try to load from cache
        const cachedCategories = await getCategoriesCache();
        const cacheIsValid = await isCacheValid();

        if (cachedCategories && cachedCategories.length > 0 && cacheIsValid) {
            categories.value = cachedCategories;
            return;
        }

        // Try to fetch from API
        const apiCategories = await fetchCategories();
        categories.value = apiCategories;
    } catch (err) {
        // Fall back to static data
        console.warn("Failed to load categories, using static data:", err);
        categories.value = categoryStores;
    }
};

const loadCategoriesWithSections = async () => {
    isLoading.value = true;
    loadError.value = null;

    try {
        // Load categories first
        await loadCategories();
        
        // Get all sections to count them per category
        const allSections = await getAllSections(categories.value);
        
        // Build category list with section counts
        categoriesWithSections.value = allSections.map((cat) => ({
            categoryId: cat.categoryId,
            categoryName: cat.categoryName,
            sectionCount: cat.sections.length,
        }));
    } catch (error) {
        console.error("Failed to load categories:", error);
        loadError.value = "ไม่สามารถโหลดหมวดกฎหมายได้ในขณะนี้ กรุณาลองอีกครั้ง";
    } finally {
        isLoading.value = false;
    }
};

const formatSectionCount = (count: number) => `${count} มาตรา`;

const selectCategory = (categoryId: string) => {
    // Check if category has multiple data sources
    const category = categories.value.find((c: CategoryStore) => c.id === categoryId);
    
    if (category?.dataSources && category.dataSources.length > 1) {
        // Navigate to data source selection view
        router.push({ name: "section-datasources", params: { categoryId } });
    } else {
        router.push({ name: "sections-detail", params: { categoryId } });
    }
};

onMounted(() => {
    // Set header to indicate Sections mode
    setHeader("รายการมาตรา", "Sections List");
    
    loadCategoriesWithSections();
});

// Reset header on unmount
onUnmounted(() => {
    resetHeader();
});
</script>

<style scoped>
.main-content {
    --section-cat-ink: #1f2937;
    --section-cat-body: #4b5563;
    --section-cat-muted: #6b7280;
    --section-cat-border: #e5e7eb;
    --section-cat-surface: #ffffff;
    --section-cat-surface-soft: #f9fafb;
    --section-cat-accent: #3b82f6;
    --section-cat-focus: rgba(37, 99, 235, 0.24);
    --section-cat-arrow: #64748b;
    --section-cat-empty-bg: #f8fafc;

    flex: 1;
    padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
    max-width: 600px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.sections-container {
    display: flex;
    flex-direction: column;
    gap: 2rem;
}

.header-section {
    text-align: center;
    padding: 0.75rem 0;
}

.title {
    font-size: clamp(1.5rem, 4vw, 2rem);
    font-weight: 700;
    color: var(--section-cat-ink);
    margin: 0 0 0.5rem 0;
    line-height: 1.25;
    overflow-wrap: anywhere;
}

.subtitle {
    font-size: 1rem;
    color: var(--section-cat-body);
    margin: 0;
    line-height: 1.6;
}

.categories-list {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.category-item {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1.25rem;
    background: var(--section-cat-surface);
    border: 1px solid var(--section-cat-border);
    border-radius: 1rem;
    cursor: pointer;
    transition:
        transform 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        background-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        border-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        box-shadow 0.2s cubic-bezier(0.25, 1, 0.5, 1);
    text-align: left;
    width: 100%;
    min-height: 88px;
    -webkit-tap-highlight-color: transparent;
}

@media (hover: hover) {
    .category-item:hover {
        background: var(--section-cat-surface-soft);
        border-color: var(--section-cat-accent);
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.15);
    }
}

.category-item:active {
    transform: translateY(0);
    background: var(--section-cat-surface-soft);
}

.category-item:focus-visible {
    outline: 3px solid var(--section-cat-focus);
    outline-offset: 2px;
}

.category-icon {
    font-size: 2.5rem;
    flex-shrink: 0;
}

.category-content {
    flex: 1;
}

.category-name {
    font-size: 1.25rem;
    font-weight: 700;
    color: var(--section-cat-ink);
    margin-bottom: 0.25rem;
    overflow-wrap: anywhere;
}

.category-count {
    font-size: 0.875rem;
    color: var(--section-cat-body);
}

.category-arrow {
    flex-shrink: 0;
}

.arrow-icon {
    width: 1.5rem;
    height: 1.5rem;
    color: var(--section-cat-arrow);
    transition:
        color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        transform 0.2s cubic-bezier(0.25, 1, 0.5, 1);
}

@media (hover: hover) {
    .category-item:hover .arrow-icon {
        color: var(--section-cat-accent);
        transform: translateX(4px);
    }
}

.empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.75rem;
    text-align: center;
    padding: 2rem 1rem;
    background: var(--section-cat-empty-bg);
    border: 1px solid var(--section-cat-border);
    border-radius: 1rem;
}

.empty-title {
    margin: 0;
    color: var(--section-cat-ink);
    font-size: 1.125rem;
    line-height: 1.4;
}

.empty-description {
    margin: 0;
    color: var(--section-cat-body);
    font-size: 0.9375rem;
    line-height: 1.6;
}

.empty-action {
    margin-top: 0.25rem;
    padding: 0.75rem 1rem;
    border: 1px solid var(--section-cat-accent);
    background: var(--section-cat-surface);
    color: var(--section-cat-accent);
    border-radius: 0.75rem;
    font-size: 0.9375rem;
    font-weight: 600;
    cursor: pointer;
    transition:
        background-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        border-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        box-shadow 0.2s cubic-bezier(0.25, 1, 0.5, 1);
}

@media (hover: hover) {
    .empty-action:hover {
        background: var(--section-cat-accent);
        color: #ffffff;
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.2);
    }
}

.empty-action:focus-visible {
    outline: 3px solid var(--section-cat-focus);
    outline-offset: 2px;
}

.empty-action:active {
    background: #dbeafe;
}

@media (max-width: 640px) {
    .main-content {
        padding: 0.875rem max(0.75rem, env(safe-area-inset-right, 0px)) 1rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .title {
        font-size: 1.5rem;
    }

    .subtitle {
        font-size: 1rem;
    }

    .category-item {
        padding: 1rem;
        min-height: 76px;
    }

    .category-icon {
        font-size: 2rem;
    }

    .category-name {
        font-size: 1.125rem;
    }

    .category-count {
        font-size: 0.8125rem;
    }
}

@media (prefers-reduced-motion: reduce) {
    .category-item,
    .arrow-icon,
    .empty-action {
        transition: none;
    }
}
</style>
