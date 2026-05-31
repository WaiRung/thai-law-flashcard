<template>
    <main class="main-content" ref="flashcardViewRef">
        <LoadingSpinner v-if="isLoading" message="กำลังโหลดแฟลชการ์ด..." />

        <section v-else-if="loadError" class="status-panel" role="alert" aria-live="assertive">
            <h2 class="status-title">ไม่สามารถเปิดหน้าแฟลชการ์ดได้</h2>
            <p class="status-copy">{{ loadError }}</p>
            <div class="status-actions">
                <button type="button" class="action-btn action-btn-primary" @click="initializePage">
                    โหลดอีกครั้ง
                </button>
                <button type="button" class="action-btn action-btn-secondary" @click="goBackToCategories">
                    กลับไปหน้าเลือกหมวด
                </button>
            </div>
        </section>

        <section v-else-if="!currentCard" class="status-panel" role="status" aria-live="polite">
            <h2 class="status-title">ยังไม่มีแฟลชการ์ดในหมวดนี้</h2>
            <p class="status-copy">ไม่พบข้อมูลคำถามที่พร้อมใช้งานในหมวดที่เลือก กรุณาเลือกหมวดหรือแหล่งข้อมูลอื่น</p>
            <div class="status-actions">
                <button type="button" class="action-btn action-btn-secondary" @click="goBackToCategories">
                    กลับไปหน้าเลือกหมวด
                </button>
            </div>
        </section>

        <template v-else>
            <ProgressBar
                :current-index="currentIndex"
                :total-cards="totalCards"
                :completed-count="completedCards.size"
            />

            <div class="flashcard-wrapper">
                <FlashCard
                    :card="currentCard"
                    :isFlipped="isFlipped"
                    :hasDescription="currentCardHasDescription"
                    :sectionId="currentCard.id"
                    @flip="toggleFlip"
                    @showDescription="showDescriptionModal"
                />
            </div>

            <DescriptionModal
                :isOpen="isDescriptionModalOpen"
                :sectionId="currentCard.id"
                :descriptions="currentDescriptions"
                @close="closeDescriptionModal"
            />

            <FlashcardControls
                :current-index="currentIndex"
                :total-cards="totalCards"
                @previous="previousCard"
                @next="nextCard"
                @shuffle="shuffleCards"
                @reset="resetProgress"
            />
        </template>
    </main>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from "vue";
import { useRouter, useRoute } from "vue-router";
import FlashCard from "../components/FlashCard.vue";
import ProgressBar from "../components/ProgressBar.vue";
import FlashcardControls from "../components/FlashcardControls.vue";
import DescriptionModal from "../components/DescriptionModal.vue";
import LoadingSpinner from "../components/LoadingSpinner.vue";
import { categoryStores } from "../data/categoryStores";
import { fetchCategories } from "../services/api";
import {
    getCategoriesCache,
    isCacheValid,
    getDescriptionsCache,
} from "../services/cache";
import { filterQuestionsByDataSource } from "../services/filterService";
import type {
    Flashcard,
    CategoryStore,
} from "../types/flashcard";
import type { DescriptionCache } from "../types/description";
import { useHeader } from "../composables/useHeader";

const router = useRouter();
const route = useRoute();
const { setHeader, resetHeader } = useHeader();

// Get categoryId and dataSourceIndex from route params
const categoryId = computed(() => route.params.categoryId as string);
const dataSourceIndex = computed(() => {
    const index = route.params.dataSourceIndex;
    return index !== undefined ? parseInt(index as string, 10) : undefined;
});

// State
const cards = ref<Flashcard[]>([]);
const currentIndex = ref(0);
const isFlipped = ref(false);
const completedCards = ref(new Set<string>());
const categories = ref<CategoryStore[]>([]);
const descriptionsCache = ref<DescriptionCache>({});
const isDescriptionModalOpen = ref(false);
const currentDescriptions = ref<Array<{ content: string }>>([]);
const isLoading = ref(true);
const loadError = ref<string | null>(null);

// Touch gesture state
const touchStartX = ref(0);
const touchStartY = ref(0);
const touchEndX = ref(0);
const touchEndY = ref(0);

// Touch gesture constants
const MIN_SWIPE_DISTANCE = 50;
const MAX_VERTICAL_DISTANCE = 100;

// Ref for flashcard view element
const flashcardViewRef = ref<HTMLElement | null>(null);

// Computed
const currentCard = computed(() => cards.value[currentIndex.value]);
const totalCards = computed(() => cards.value.length);
const currentCardHasDescription = computed(() => {
    if (!currentCard.value) return false;
    const description = descriptionsCache.value[currentCard.value.id];
    return description && description.descriptions && description.descriptions.length > 0;
});

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
        if (import.meta.env.DEV) {
            console.warn("Failed to load categories, using static data:", err);
        }
        categories.value = categoryStores;
    }
};

// Load flashcards for the selected category
const loadFlashcards = async () => {
    loadError.value = null;

    const selectedStore = categories.value.find(
        (store: CategoryStore) => store.id === categoryId.value,
    );
    
    if (!selectedStore) {
        loadError.value = "ไม่พบหมวดหมู่ที่เลือก กรุณากลับไปเลือกหมวดใหม่อีกครั้ง";
        setHeader("ไม่พบหมวดหมู่", "Category not found");
        return;
    }

    // Set header title and subtitle based on the selected category or data source
    if (dataSourceIndex.value !== undefined && selectedStore.dataSources) {
        const dataSource = selectedStore.dataSources[dataSourceIndex.value];
        if (dataSource) {
            const title = dataSource.nameTh || selectedStore.nameTh;
            const subtitle = dataSource.nameEn || selectedStore.nameEn;
            setHeader(title, subtitle);
        } else {
            setHeader(selectedStore.nameTh, selectedStore.nameEn);
        }
    } else {
        setHeader(selectedStore.nameTh, selectedStore.nameEn);
    }

    // Apply question filtering based on allowed IDs and data source
    const filteredQuestions = await filterQuestionsByDataSource(
        categoryId.value,
        selectedStore.questions,
        dataSourceIndex.value
    );

    cards.value = [...filteredQuestions];
    currentIndex.value = 0;
    isFlipped.value = false;
    completedCards.value.clear();

    if (cards.value.length === 0) {
        setHeader("ยังไม่มีข้อมูลแฟลชการ์ด", "No flashcards available");
    }
};

const initializePage = async () => {
    isLoading.value = true;
    loadError.value = null;

    try {
        await loadCategories();
        await loadFlashcards();
        descriptionsCache.value = await getDescriptionsCache();
    } catch {
        loadError.value = "ไม่สามารถโหลดข้อมูลแฟลชการ์ดได้ในขณะนี้ กรุณาลองใหม่อีกครั้ง";
        setHeader("เกิดข้อผิดพลาด", "Something went wrong");
    } finally {
        isLoading.value = false;
    }
};

const goBackToCategories = () => {
    router.push({ name: "flashcard-categories" });
};

// Methods
const toggleFlip = () => {
    isFlipped.value = !isFlipped.value;
    if (isFlipped.value) {
        completedCards.value.add(currentCard.value.id);
    }
};

const nextCard = () => {
    if (currentIndex.value < totalCards.value - 1) {
        currentIndex.value++;
        isFlipped.value = false;
    }
};

const previousCard = () => {
    if (currentIndex.value > 0) {
        currentIndex.value--;
        isFlipped.value = false;
    }
};

const shuffleCards = () => {
    const shuffled = [...cards.value];
    for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
    }
    cards.value = shuffled;
    currentIndex.value = 0;
    isFlipped.value = false;
};

const resetProgress = () => {
    completedCards.value.clear();
    currentIndex.value = 0;
    isFlipped.value = false;
};

const showDescriptionModal = () => {
    if (!currentCard.value) return;
    const description = descriptionsCache.value[currentCard.value.id];
    if (description && description.descriptions) {
        currentDescriptions.value = description.descriptions;
        isDescriptionModalOpen.value = true;
    }
};

const closeDescriptionModal = () => {
    isDescriptionModalOpen.value = false;
};

// Touch gesture handlers
const handleTouchStart = (e: TouchEvent) => {
    touchStartX.value = e.touches[0].clientX;
    touchStartY.value = e.touches[0].clientY;
};

const handleTouchMove = (e: TouchEvent) => {
    touchEndX.value = e.touches[0].clientX;
    touchEndY.value = e.touches[0].clientY;
};

const handleTouchEnd = () => {
    const deltaX = touchEndX.value - touchStartX.value;
    const deltaY = Math.abs(touchEndY.value - touchStartY.value);

    // Only process swipe if vertical movement is within acceptable range
    if (deltaY <= MAX_VERTICAL_DISTANCE) {
        // Right swipe - back to categories (use back to preserve history)
        if (deltaX > MIN_SWIPE_DISTANCE) {
            router.back();
        }
        // Left swipe - next card
        else if (deltaX < -MIN_SWIPE_DISTANCE && currentIndex.value < totalCards.value - 1) {
            nextCard();
        }
    }

    // Reset touch coordinates
    touchStartX.value = 0;
    touchStartY.value = 0;
    touchEndX.value = 0;
    touchEndY.value = 0;
};

// Helper function to add touch event listeners
const addTouchListeners = () => {
    if (flashcardViewRef.value) {
        flashcardViewRef.value.addEventListener('touchstart', handleTouchStart, { passive: true });
        flashcardViewRef.value.addEventListener('touchmove', handleTouchMove, { passive: true });
        flashcardViewRef.value.addEventListener('touchend', handleTouchEnd, { passive: true });
    }
};

// Helper function to remove touch event listeners
const removeTouchListeners = () => {
    if (flashcardViewRef.value) {
        flashcardViewRef.value.removeEventListener('touchstart', handleTouchStart);
        flashcardViewRef.value.removeEventListener('touchmove', handleTouchMove);
        flashcardViewRef.value.removeEventListener('touchend', handleTouchEnd);
    }
};

// Watch for category changes in route params
watch(
    [
        () => route.params.categoryId,
        () => route.params.dataSourceIndex,
    ],
    async ([newCategoryId, newDataSourceIndex], [oldCategoryId, oldDataSourceIndex]) => {
        if (
            categories.value.length > 0
            && newCategoryId
            && (newCategoryId !== oldCategoryId || newDataSourceIndex !== oldDataSourceIndex)
        ) {
            isLoading.value = true;
            await loadFlashcards();
            isLoading.value = false;
        }
    },
);

watch(() => categories.value.length, async (newLength: number) => {
    if (newLength > 0 && categoryId.value) {
        await loadFlashcards();
    }
});

// Initialize on mount
onMounted(async () => {
    await initializePage();
    
    // Add touch listeners after DOM update
    await nextTick();
    addTouchListeners();
});

// Cleanup event listeners on unmount
onUnmounted(() => {
    removeTouchListeners();
    resetHeader();
});
</script>

<style scoped>
.main-content {
    --flashcard-status-ink: #1f2937;
    --flashcard-status-body: #4b5563;
    --flashcard-status-border: #e5e7eb;
    --flashcard-status-surface: #ffffff;
    --flashcard-accent: #4f46e5;
    --flashcard-accent-deep: #4338ca;
    --flashcard-focus: rgba(79, 70, 229, 0.24);

    flex: 1;
    padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
    max-width: 600px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.flashcard-wrapper {
    width: 100%;
    margin-bottom: 1.5rem;
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

.action-btn:focus-visible {
    outline: none;
    box-shadow: 0 0 0 3px var(--flashcard-focus);
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

@media (prefers-reduced-motion: reduce) {
    .action-btn {
        transition: none;
    }

    .action-btn-primary:hover {
        transform: none;
    }
}

@media (max-width: 640px) {
    .main-content {
        padding: 0.875rem max(0.75rem, env(safe-area-inset-right, 0px)) 1rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .flashcard-wrapper {
        width: 100%;
    }
}
</style>
