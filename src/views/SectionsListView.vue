<template>
    <main class="main-content">
        <LoadingSpinner v-if="isLoading" message="กำลังโหลดรายการมาตรา..." />
        <ErrorMessage
            v-else-if="loadError"
            :message="loadError"
            @retry="loadSections"
        />
        <template v-else>
            <div class="sections-container">
                <div class="header-section">
                    <h2 class="title">รายการมาตรา</h2>
                    <p class="subtitle">อ่านเนื้อหามาตราและเปิดคำอธิบายเพิ่มเติมได้จากรายการด้านล่าง</p>
                </div>

                <template v-if="hasSections">
                    <div
                        v-for="category in categorySections"
                        :key="category.categoryId"
                        class="category-section"
                    >
                        <h3 class="category-title">{{ category.categoryName }}</h3>
                        <div class="sections-list">
                            <div
                                v-for="section in category.sections"
                                :key="section.id"
                                class="section-item"
                            >
                                <div class="section-header-row">
                                    <div class="section-header">{{ section.id }}</div>
                                    <button
                                        v-if="hasDescription(section)"
                                        class="info-button-list"
                                        @click="showDescriptionModal(section)"
                                        type="button"
                                        aria-label="ดูคำอธิบายเพิ่มเติม"
                                    >
                                        <svg
                                            xmlns="http://www.w3.org/2000/svg"
                                            viewBox="0 0 24 24"
                                            fill="none"
                                            stroke="currentColor"
                                            stroke-width="2"
                                            stroke-linecap="round"
                                            stroke-linejoin="round"
                                            class="info-icon"
                                            focusable="false"
                                        >
                                            <circle cx="12" cy="12" r="10"></circle>
                                            <line x1="12" y1="16" x2="12" y2="12"></line>
                                            <line x1="12" y1="8" x2="12.01" y2="8"></line>
                                        </svg>
                                    </button>
                                </div>
                                <div v-if="section.title && !section.id.includes('อนุ')" class="section-title">
                                    {{ section.title }}
                                </div>
                                <div class="section-answer">{{ cleanAnswerForDisplay(section.answer, section) }}</div>
                            </div>
                        </div>
                    </div>
                </template>

                <section
                    v-else
                    class="empty-state"
                    role="status"
                    aria-live="polite"
                >
                    <h3 class="empty-title">{{ emptyTitle }}</h3>
                    <p class="empty-description">{{ emptyDescription }}</p>
                    <div class="empty-actions">
                        <button
                            type="button"
                            class="empty-action empty-action--primary"
                            @click="goToSectionsCategories"
                        >
                            กลับไปเลือกหมวดกฎหมาย
                        </button>
                        <button
                            type="button"
                            class="empty-action empty-action--secondary"
                            @click="loadSections"
                        >
                            โหลดรายการมาตราอีกครั้ง
                        </button>
                    </div>
                </section>
            </div>

            <!-- Description Modal -->
            <DescriptionModal
                :isOpen="isDescriptionModalOpen"
                :sectionId="currentSectionId"
                :descriptions="currentDescriptions"
                @close="closeDescriptionModal"
            />
        </template>
    </main>
</template>

<script setup lang="ts">
import { computed, ref, onMounted, onUnmounted } from "vue";
import { useRouter } from "vue-router";
import LoadingSpinner from "../components/LoadingSpinner.vue";
import DescriptionModal from "../components/DescriptionModal.vue";
import ErrorMessage from "../components/ErrorMessage.vue";
import { getCategoryDataSourceSections } from "../services/sectionService";
import { categoryStores } from "../data/categoryStores";
import { fetchCategories } from "../services/api";
import {
    getCategoriesCache,
    isCacheValid,
} from "../services/cache";
import { useHeader } from "../composables/useHeader";
import type { CategoryStore } from "../types/flashcard";
import type { DescriptionContent } from "../types/description";

interface SectionContent {
    id: string;
    question: string;
    answer: string;
    title?: string; // Optional title for whole sections
    descriptions?: DescriptionContent[]; // Optional descriptions
}

interface CategorySections {
    categoryId: string;
    categoryName: string;
    sections: SectionContent[];
}

// Accept categoryId and optional dataSourceIndex as props
const props = defineProps<{
    categoryId: string;
    dataSourceIndex?: string;
}>();

const router = useRouter();
const { setHeader, resetHeader } = useHeader();
const isLoading = ref(true);
const loadError = ref<string | null>(null);
const emptyStateKind = ref<"not-found" | "no-sections">("not-found");
const categorySections = ref<CategorySections[]>([]);
const categories = ref<CategoryStore[]>([]);
const isDescriptionModalOpen = ref(false);
const currentSectionId = ref("");
const currentDescriptions = ref<DescriptionContent[]>([]);
const hasSections = computed(() => categorySections.value.some((category: CategorySections) => category.sections.length > 0));
const emptyTitle = computed(() =>
    emptyStateKind.value === "not-found" ? "ไม่พบหมวดกฎหมายที่เลือก" : "ยังไม่พบข้อมูลมาตราในหมวดนี้"
);
const emptyDescription = computed(() =>
    emptyStateKind.value === "not-found"
        ? "อาจเป็นลิงก์ที่ไม่ถูกต้อง หรือข้อมูลถูกปรับปรุงแล้ว"
        : "หมวดนี้อาจยังไม่มีข้อมูลพร้อมใช้งาน ลองโหลดอีกครั้งในภายหลัง"
);

/**
 * Clean answer text by removing redundant section ID
 * For full sections (with title and no "อนุ" in ID): Keep subsection IDs like (1), (2)
 * For individual subsection flashcards: Remove subsection ID prefix
 * @param answer - The original answer text
 * @param section - The section object to determine if it's a full section
 * @returns Cleaned answer text
 */
const cleanAnswerForDisplay = (answer: string, section: SectionContent): string => {
    const lines = answer.split('\n');
    const cleanedLines: string[] = [];
    const isShowBracketForSubsection = (
        section.title && !section.id.includes('อนุ')) ||
        section.id.includes('วรรค');

    for (let i = 0; i < lines.length; i++) {
        const line = lines[i];

        // Skip the first line if it's a section ID (e.g., "มาตรา 1", "มาตรา 1 วรรค 2")
        if (i === 0 && line.trim().startsWith('มาตรา')) {
            continue;
        }

        // Handle subsection IDs based on section type
        const subsectionMatch = line.match(/^(\s*)\((\d+)\)\s+(.+)$/);
        if (subsectionMatch) {
            const [, spaces, id, content] = subsectionMatch;

            if (isShowBracketForSubsection) {
                // For full sections: Keep the subsection ID
                cleanedLines.push(`${spaces}(${id}) ${content}`);
            } else {
                // For individual subsection flashcards: Remove the ID
                cleanedLines.push(`${spaces}${content}`);
            }
        } else {
            cleanedLines.push(line);
        }
    }

    return cleanedLines.join('\n').trim();
};

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

const hasDescription = (section: SectionContent): boolean => {
    return section.descriptions !== undefined && section.descriptions.length > 0;
};

const showDescriptionModal = (section: SectionContent) => {
    if (section.descriptions && section.descriptions.length > 0) {
        currentSectionId.value = section.id;
        currentDescriptions.value = section.descriptions;
        isDescriptionModalOpen.value = true;
    }
};

const closeDescriptionModal = () => {
    isDescriptionModalOpen.value = false;
};

const loadSections = async () => {
    isLoading.value = true;
    loadError.value = null;
    emptyStateKind.value = "not-found";

    try {
        // Load categories first
        await loadCategories();

        // Parse dataSourceIndex if provided
        const dataSourceIdx = props.dataSourceIndex !== undefined
            ? parseInt(props.dataSourceIndex, 10)
            : undefined;

        // Get sections for the specific category and data source
        const categorySection = await getCategoryDataSourceSections(
            props.categoryId,
            dataSourceIdx,
            categories.value
        );

        if (categorySection) {
            categorySections.value = [categorySection];
            // Set header to show the current category/data source name
            setHeader(categorySection.categoryName, "รายการมาตรา");

            if (categorySection.sections.length === 0) {
                emptyStateKind.value = "no-sections";
            }
        } else {
            categorySections.value = [];
            emptyStateKind.value = "not-found";
        }
    } catch (error) {
        console.error("Failed to load sections:", error);
        loadError.value = "ไม่สามารถโหลดรายการมาตราได้ในขณะนี้ กรุณาลองอีกครั้ง";
    } finally {
        isLoading.value = false;
    }
};

const goToSectionsCategories = () => {
    router.push({ name: "sections-list" });
};

onMounted(() => {
    setHeader("รายการมาตรา", "Sections List");
    loadSections();
});

// Reset header on unmount
onUnmounted(() => {
    resetHeader();
});
</script>

<style scoped>
.main-content {
    --sections-ink: #1f2937;
    --sections-body: #374151;
    --sections-muted: #6b7280;
    --sections-border: #e5e7eb;
    --sections-surface: #ffffff;
    --sections-surface-soft: #f9fafb;
    --sections-accent: #3b82f6;
    --sections-accent-strong: #2563eb;
    --sections-accent-soft: #eff6ff;
    --sections-success-ink: #065f46;
    --sections-success-border: #86efac;
    --sections-success-bg: #f0fdf4;
    --sections-info-ink: #0f766e;
    --sections-info-border: #99f6e4;
    --sections-info-bg: #ecfeff;
    --sections-focus: rgba(37, 99, 235, 0.24);

    flex: 1;
    padding: 1.25rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.5rem max(0.75rem, env(safe-area-inset-left, 0px));
    max-width: 860px;
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
    font-size: clamp(1.625rem, 4.2vw, 2rem);
    font-weight: 700;
    color: var(--sections-ink);
    margin: 0 0 0.5rem 0;
    line-height: 1.25;
    overflow-wrap: anywhere;
}

.subtitle {
    font-size: 1rem;
    color: var(--sections-body);
    margin: 0;
    line-height: 1.6;
}

.category-section {
    background: var(--sections-surface);
    border: 1px solid var(--sections-border);
    border-radius: 1rem;
    padding: 1.25rem;
}

.category-title {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--sections-ink);
    margin: 0 0 1rem 0;
    padding-bottom: 0.75rem;
    border-bottom: 1px solid var(--sections-border);
    overflow-wrap: anywhere;
}

.sections-list {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
}

.section-item {
    padding: 1.25rem;
    background: var(--sections-surface-soft);
    border: 1px solid var(--sections-border);
    border-radius: 0.75rem;
    transition:
        background-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        border-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        box-shadow 0.2s cubic-bezier(0.25, 1, 0.5, 1);
}

@media (hover: hover) {
    .section-item:hover {
        background: #f3f4f6;
        border-color: var(--sections-accent);
        box-shadow: 0 2px 8px rgba(59, 130, 246, 0.15);
    }
}

.section-header-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.5rem;
    margin-bottom: 0.75rem;
}

.section-header {
    font-size: 1rem;
    font-weight: 700;
    color: var(--sections-accent);
    overflow-wrap: anywhere;
}

.info-button-list {
    width: 44px;
    height: 44px;
    border-radius: 50%;
    background: var(--sections-info-bg);
    color: var(--sections-info-ink);
    border: 1px solid var(--sections-info-border);
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition:
        transform 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        background-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        border-color 0.2s cubic-bezier(0.25, 1, 0.5, 1),
        box-shadow 0.2s cubic-bezier(0.25, 1, 0.5, 1);
    padding: 0;
    flex-shrink: 0;
    -webkit-tap-highlight-color: transparent;
}

@media (hover: hover) {
    .info-button-list:hover {
        transform: scale(1.04);
        background: #ccfbf1;
        border-color: #5eead4;
        box-shadow: 0 4px 10px rgba(13, 148, 136, 0.24);
    }
}

.info-button-list:active {
    transform: scale(0.95);
}

.info-button-list:focus-visible {
    outline: 3px solid var(--sections-focus);
    outline-offset: 2px;
}

.info-button-list .info-icon {
    width: 18px;
    height: 18px;
    color: currentColor;
}

.section-title {
    font-size: 1rem;
    font-weight: 700;
    color: var(--sections-success-ink);
    margin-bottom: 0.75rem;
    padding: 0.5rem 0.625rem;
    background: var(--sections-success-bg);
    border: 1px solid var(--sections-success-border);
    border-radius: 0.5rem;
}

.section-answer {
    font-size: 0.9375rem;
    color: var(--sections-body);
    line-height: 1.8;
    white-space: pre-wrap;
    word-wrap: break-word;
}

.empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.75rem;
    text-align: center;
    padding: 2rem 1rem;
    background: var(--sections-surface-soft);
    border: 1px solid var(--sections-border);
    border-radius: 1rem;
}

.empty-title {
    margin: 0;
    color: var(--sections-ink);
    font-size: 1.125rem;
    line-height: 1.4;
}

.empty-description {
    margin: 0;
    color: var(--sections-body);
    font-size: 0.9375rem;
    line-height: 1.6;
    max-width: 48ch;
}

.empty-actions {
    width: 100%;
    margin-top: 0.25rem;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 0.75rem;
}

.empty-action {
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

.empty-action--primary {
    background: var(--sections-accent);
    color: #ffffff;
}

.empty-action--secondary {
    background: var(--sections-surface);
    color: var(--sections-accent);
    border-color: var(--sections-accent);
}

@media (hover: hover) {
    .empty-action--primary:hover {
        background: var(--sections-accent-strong);
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.25);
    }

    .empty-action--secondary:hover {
        background: var(--sections-accent-soft);
    }
}

.empty-action:focus-visible {
    outline: 3px solid var(--sections-focus);
    outline-offset: 2px;
}

@media (max-width: 640px) {
    .main-content {
        padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .title {
        font-size: 1.5rem;
    }

    .subtitle {
        font-size: 1rem;
    }

    .empty-state {
        padding: 1.5rem 0.875rem;
    }

    .empty-actions {
        flex-direction: column;
    }

    .empty-action {
        width: 100%;
    }

    .category-section {
        padding: 1rem;
    }

    .sections-list {
        gap: 1rem;
    }

    .section-item {
        padding: 1rem;
    }

    .section-header-row {
        margin-bottom: 0.5rem;
    }

    .section-header {
        font-size: 0.875rem;
    }

    .info-button-list {
        width: 42px;
        height: 42px;
    }

    .info-button-list .info-icon {
        width: 16px;
        height: 16px;
    }

    .section-title {
        font-size: 0.875rem;
        margin-bottom: 0.5rem;
        padding: 0.375rem;
    }

    .section-answer {
        font-size: 0.875rem;
    }
}

@media (prefers-reduced-motion: reduce) {
    .section-item,
    .info-button-list,
    .empty-action {
        transition: none;
    }

    .info-button-list:active {
        transform: none;
    }
}
</style>
