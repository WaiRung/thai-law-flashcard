<template>
    <main class="main-content">
        <div class="diagram-container">
            <div class="header-section">
                <h1 class="page-title">Diagrams</h1>
                <p class="page-subtitle">
                    แผนภาพสรุปประเด็นกฎหมายสำหรับเปิดทบทวนอย่างรวดเร็ว แล้วขยายอ่านรายละเอียดได้ทันที
                </p>
                <div v-if="diagramCategories.length > 0" class="summary-row" aria-label="Diagram summary">
                    <p class="summary-pill">{{ diagramCategories.length }} หมวดหมู่</p>
                    <p class="summary-pill">{{ totalDiagramCount }} แผนภาพ</p>
                </div>
            </div>

            <div v-if="openError" class="error-banner" role="alert" aria-live="polite">
                <p class="error-title">เปิดแผนภาพไม่สำเร็จ</p>
                <p class="error-message">{{ openError }}</p>
            </div>

            <div v-if="diagramCategories.length === 0" class="empty-state">
                <p class="empty-message">ยังไม่มีแผนภาพในระบบ</p>
                <p class="empty-submessage">No diagrams are available right now.</p>
            </div>

            <div v-else class="categories-list">
                <section
                    v-for="category in diagramCategories"
                    :key="category.categoryId"
                    class="category-section"
                >
                    <div class="category-header">
                        <div>
                            <h2 class="category-title">{{ category.nameTh }}</h2>
                            <p class="category-subtitle">{{ category.nameEn }}</p>
                        </div>
                        <p class="category-count">{{ category.images.length }} ภาพ</p>
                    </div>

                    <div class="images-grid">
                        <article v-for="(image, index) in category.images" :key="index" class="image-card">
                            <button
                                type="button"
                                class="image-trigger"
                                :class="{ 'is-opening': openingImageKey === getCacheKey(category.categoryId, image.filename) }"
                                :disabled="openingImageKey === getCacheKey(category.categoryId, image.filename)"
                                :aria-busy="openingImageKey === getCacheKey(category.categoryId, image.filename)"
                                @click="openImageModal(category, image)"
                                :aria-label="`เปิดภาพ ${image.nameTh} แบบเต็มหน้าจอ`"
                            >
                                <div class="image-wrapper">
                                    <div
                                        v-if="isImageLoading(category.categoryId, index)"
                                        class="loading-overlay"
                                        role="status"
                                        aria-live="polite"
                                    >
                                        <div class="spinner"></div>
                                        <p class="loading-text">กำลังโหลด...</p>
                                    </div>
                                    <img
                                        :src="getImageUrlSync(category.categoryId, image.filename)"
                                        :alt="getImageAlt(image)"
                                        class="diagram-image"
                                        :class="{ 'image-loaded': !isImageLoading(category.categoryId, index) }"
                                        @load="handleImageLoad($event, category.categoryId, index)"
                                        @error="handleImageError($event, category.categoryId, index)"
                                    />
                                    <div class="image-overlay">
                                        <span class="image-overlay-label">แตะเพื่อขยาย</span>
                                    </div>
                                </div>
                            </button>
                            <div class="image-info">
                                <p class="image-title">{{ image.nameTh }}</p>
                                <p class="image-subtitle">{{ image.nameEn || "English title unavailable" }}</p>
                                <p
                                    v-if="openingImageKey === getCacheKey(category.categoryId, image.filename)"
                                    class="image-status"
                                >
                                    กำลังเปิดภาพขนาดเต็ม...
                                </p>
                            </div>
                        </article>
                    </div>
                </section>
            </div>
        </div>

        <ImageModal
            :is-open="isModalOpen"
            :image-url="selectedImage.url"
            :title="selectedImage.title"
            :subtitle="selectedImage.subtitle"
            @close="closeImageModal"
        />
    </main>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import diagramsConfig from "../config/diagrams.json";
import ImageModal from "../components/ImageModal.vue";
import { getCachedDiagramImage } from "../services/diagramService";
import type { DiagramCategory, DiagramImage } from "../types/diagram";

interface ImageLoadingState {
    [key: string]: boolean;
}

interface ImageUrlCache {
    [key: string]: string;
}

const SPINNER_MIN_DISPLAY_TIME = 100;

const diagramCategories = ref<DiagramCategory[]>([]);
const baseUrl = diagramsConfig.baseUrl;
const imageLoadingStates = ref<ImageLoadingState>({});
const imageUrlCache = ref<ImageUrlCache>({});
const isModalOpen = ref(false);
const openError = ref("");
const openingImageKey = ref("");
const selectedImage = ref({
    url: "",
    title: "",
    subtitle: ""
});

const totalDiagramCount = computed(() => {
    return diagramCategories.value.reduce((total: number, category: DiagramCategory) => {
        return total + category.images.length;
    }, 0);
});

const FALLBACK_IMAGE_SVG = "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='400' height='300'%3E%3Crect width='400' height='300' fill='%23f3f4f6'/%3E%3Ctext x='50%25' y='50%25' font-family='Arial' font-size='16' fill='%236b7280' text-anchor='middle' dominant-baseline='middle'%3EImage not available%3C/text%3E%3C/svg%3E";

const getImageKey = (categoryId: string, imageIndex: number): string => {
    return `${categoryId}-${imageIndex}`;
};

const isImageLoading = (categoryId: string, imageIndex: number): boolean => {
    const key = getImageKey(categoryId, imageIndex);
    return imageLoadingStates.value[key] ?? true;
};

const getCacheKey = (categoryId: string, filename: string): string => {
    return `${categoryId}-${filename}`;
};

const getImageUrl = async (categoryId: string, categoryPath: string, filename: string): Promise<string> => {
    const key = getCacheKey(categoryId, filename);

    if (imageUrlCache.value[key]) {
        return imageUrlCache.value[key];
    }

    const cachedImage = await getCachedDiagramImage(categoryId, filename);
    if (cachedImage) {
        imageUrlCache.value[key] = cachedImage;
        return cachedImage;
    }

    const remoteUrl = `${baseUrl}/${categoryPath}/${filename}`;
    imageUrlCache.value[key] = remoteUrl;
    return remoteUrl;
};

const getImageUrlSync = (categoryId: string, filename: string): string => {
    const key = getCacheKey(categoryId, filename);
    return imageUrlCache.value[key] || FALLBACK_IMAGE_SVG;
};

const getImageAlt = (image: DiagramImage): string => {
    if (image.nameTh && image.nameEn) {
        return `${image.nameTh} (${image.nameEn})`;
    }

    return image.nameTh || image.nameEn || "Diagram image";
};

const handleImageLoad = async (event: Event, categoryId: string, imageIndex: number) => {
    const img = event.target as HTMLImageElement;
    const key = getImageKey(categoryId, imageIndex);

    try {
        await img.decode();
        await new Promise((resolve) => setTimeout(resolve, SPINNER_MIN_DISPLAY_TIME));
        imageLoadingStates.value[key] = false;
    } catch {
        imageLoadingStates.value[key] = false;
    }
};

const handleImageError = (event: Event, categoryId: string, imageIndex: number) => {
    const img = event.target as HTMLImageElement;
    img.src = FALLBACK_IMAGE_SVG;
    const key = getImageKey(categoryId, imageIndex);
    imageLoadingStates.value[key] = false;
};

const openImageModal = async (category: DiagramCategory, image: DiagramImage) => {
    const imageKey = getCacheKey(category.categoryId, image.filename);

    try {
        openError.value = "";
        openingImageKey.value = imageKey;

        selectedImage.value = {
            url: await getImageUrl(category.categoryId, category.categoryPath, image.filename),
            title: image.nameTh,
            subtitle: image.nameEn
        };
        isModalOpen.value = true;
    } catch {
        openError.value = "ไม่สามารถเปิดแผนภาพได้ในขณะนี้ กรุณาลองใหม่อีกครั้ง / Cannot open this diagram right now. Please try again.";
    } finally {
        openingImageKey.value = "";
    }
};

const closeImageModal = () => {
    isModalOpen.value = false;
};

const loadDiagrams = async () => {
    diagramCategories.value = diagramsConfig.diagrams.filter((category) => category.images.length > 0);

    const preloadPromises: Promise<unknown>[] = [];

    for (const category of diagramCategories.value) {
        for (let index = 0; index < category.images.length; index++) {
            const key = getImageKey(category.categoryId, index);
            imageLoadingStates.value[key] = true;

            const image = category.images[index];
            preloadPromises.push(getImageUrl(category.categoryId, category.categoryPath, image.filename));
        }
    }

    await Promise.all(preloadPromises);
};

onMounted(async () => {
    await loadDiagrams();
});
</script>

<style scoped>
.main-content {
    --diagram-ink: #1f2937;
    --diagram-body: #4b5563;
    --diagram-muted: #6b7280;
    --diagram-border: #e5e7eb;
    --diagram-surface: #ffffff;
    --diagram-surface-soft: #f9fafb;
    --diagram-accent: #3b82f6;
    --diagram-accent-deep: #2563eb;
    --diagram-focus: rgba(37, 99, 235, 0.24);

    flex: 1;
    padding: 1.5rem 1rem 2rem;
    max-width: 1120px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.diagram-container {
    display: flex;
    flex-direction: column;
    gap: 2rem;
}

.header-section {
    text-align: center;
    padding: 1.25rem 1rem 1.75rem;
    border-radius: 1rem;
    border: 1px solid rgba(229, 231, 235, 0.82);
    background: linear-gradient(180deg, rgba(255, 255, 255, 0.75) 0%, rgba(243, 244, 246, 0.72) 100%);
}

.page-title {
    font-size: clamp(2rem, 3.8vw, 2.5rem);
    font-weight: 700;
    color: var(--diagram-ink);
    margin: 0 0 0.5rem 0;
}

.page-subtitle {
    font-size: 1.125rem;
    color: var(--diagram-body);
    line-height: 1.6;
    margin: 0 auto;
    max-width: 62ch;
}

.summary-row {
    display: flex;
    justify-content: center;
    gap: 0.75rem;
    flex-wrap: wrap;
    margin-top: 1rem;
}

.summary-pill {
    margin: 0;
    padding: 0.375rem 0.75rem;
    border-radius: 999px;
    border: 1px solid rgba(59, 130, 246, 0.28);
    background: rgba(59, 130, 246, 0.08);
    color: var(--diagram-accent-deep);
    font-size: 0.875rem;
    font-weight: 600;
}

.error-banner {
    border-radius: 1rem;
    border: 1px solid rgba(239, 68, 68, 0.18);
    background: rgba(254, 242, 242, 0.95);
    padding: 1rem 1rem 1.125rem;
}

.error-title {
    margin: 0 0 0.25rem 0;
    font-size: 1rem;
    font-weight: 700;
    color: #991b1b;
}

.error-message {
    margin: 0;
    font-size: 0.95rem;
    line-height: 1.55;
    color: #7f1d1d;
}

.empty-state {
    text-align: center;
    padding: 3rem 1rem;
    background: var(--diagram-surface);
    border-radius: 1rem;
    border: 1px solid var(--diagram-border);
}

.empty-message {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--diagram-ink);
    margin: 0 0 0.5rem 0;
}

.empty-submessage {
    font-size: 1rem;
    color: var(--diagram-body);
    margin: 0;
}

.categories-list {
    display: flex;
    flex-direction: column;
    gap: 2rem;
}

.category-section {
    background: var(--diagram-surface);
    border-radius: 1rem;
    padding: 2rem;
    border: 1px solid var(--diagram-border);
}

.category-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: 1rem;
    margin-bottom: 1.5rem;
    padding-bottom: 1rem;
    border-bottom: 1px solid var(--diagram-border);
}

.category-title {
    font-size: clamp(1.5rem, 2.3vw, 1.875rem);
    font-weight: 700;
    color: var(--diagram-ink);
    margin: 0 0 0.25rem 0;
}

.category-subtitle {
    font-size: 1rem;
    color: var(--diagram-body);
    margin: 0;
}

.category-count {
    margin: 0;
    padding: 0.32rem 0.7rem;
    border-radius: 999px;
    border: 1px solid rgba(59, 130, 246, 0.24);
    color: var(--diagram-accent-deep);
    font-size: 0.8125rem;
    font-weight: 600;
    white-space: nowrap;
    align-self: center;
}

.images-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1.5rem;
}

.image-card {
    display: flex;
    flex-direction: column;
    background: var(--diagram-surface-soft);
    border-radius: 0.875rem;
    overflow: hidden;
    border: 1px solid var(--diagram-border);
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        border-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.image-card:hover {
    transform: translateY(-2px);
    border-color: rgba(59, 130, 246, 0.5);
    box-shadow: 0 6px 16px rgba(59, 130, 246, 0.14);
}

.image-card:focus-within {
    border-color: var(--diagram-accent);
    box-shadow: 0 0 0 3px var(--diagram-focus);
}

.image-trigger {
    border: none;
    padding: 0;
    margin: 0;
    background: none;
    cursor: pointer;
    width: 100%;
    text-align: left;
}

.image-trigger:disabled {
    cursor: wait;
}

.image-trigger.is-opening .image-wrapper {
    opacity: 0.92;
}

.image-trigger:focus-visible {
    outline: none;
}

.image-wrapper {
    width: 100%;
    height: 220px;
    overflow: hidden;
    background: var(--diagram-surface);
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    transition: opacity 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.image-trigger:hover .image-wrapper {
    opacity: 0.9;
}

.image-overlay {
    position: absolute;
    inset: auto 0 0 0;
    display: flex;
    justify-content: flex-end;
    padding: 0.75rem;
    background: linear-gradient(180deg, rgba(15, 23, 42, 0) 0%, rgba(15, 23, 42, 0.4) 100%);
    opacity: 0;
    transition: opacity 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

@media (hover: hover) {
    .image-trigger:hover .image-overlay,
    .image-card:focus-within .image-overlay {
        opacity: 1;
    }
}

.image-overlay-label {
    display: inline-flex;
    align-items: center;
    min-height: 2rem;
    padding: 0.35rem 0.7rem;
    border-radius: 999px;
    background: rgba(15, 23, 42, 0.72);
    color: #f8fafc;
    font-size: 0.8125rem;
    font-weight: 600;
}

.loading-overlay {
    position: absolute;
    inset: 0;
    background: rgba(255, 255, 255, 0.94);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    z-index: 1;
    gap: 0.75rem;
}

.spinner {
    width: 2.5rem;
    height: 2.5rem;
    border: 3px solid var(--diagram-border);
    border-top-color: var(--diagram-accent);
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}

.loading-text {
    font-size: 0.875rem;
    color: var(--diagram-muted);
    margin: 0;
}

.diagram-image {
    width: 100%;
    height: 100%;
    object-fit: contain;
    padding: 1rem;
    opacity: 0;
    transition: opacity 0.22s ease-out;
}

.diagram-image.image-loaded {
    opacity: 1;
}

.image-info {
    padding: 0.9rem 1rem 1rem;
    border-top: 1px solid rgba(229, 231, 235, 0.72);
}

.image-title {
    font-size: 1rem;
    font-weight: 600;
    color: var(--diagram-ink);
    margin: 0 0 0.25rem 0;
    line-height: 1.4;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
    min-height: 2.8em;
}

.image-subtitle {
    font-size: 0.875rem;
    color: var(--diagram-body);
    margin: 0;
    line-height: 1.45;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
    min-height: 2.6em;
}

.image-status {
    margin: 0.5rem 0 0;
    font-size: 0.8125rem;
    font-weight: 600;
    line-height: 1.45;
    color: var(--diagram-accent-deep);
}

@media (max-width: 768px) {
    .main-content {
        padding: 1rem 0.75rem 1.5rem;
    }

    .header-section {
        padding: 1rem 0.75rem 1.25rem;
    }

    .page-title {
        font-size: 2rem;
    }

    .page-subtitle {
        font-size: 1rem;
    }

    .category-section {
        padding: 1.5rem;
    }

    .category-title {
        font-size: 1.4rem;
    }

    .images-grid {
        grid-template-columns: 1fr;
    }

    .category-header {
        align-items: center;
    }

    .image-wrapper {
        height: 200px;
    }
}

@media (max-width: 640px) {
    .page-title {
        font-size: 1.75rem;
    }

    .category-title {
        font-size: 1.25rem;
    }

    .category-section {
        padding: 1rem;
    }

    .category-header {
        flex-direction: column;
        align-items: flex-start;
    }

    .category-count {
        align-self: flex-start;
    }
}

@media (prefers-reduced-motion: reduce) {
    .image-card,
    .image-wrapper,
    .diagram-image,
    .spinner,
    .image-overlay {
        transition: none !important;
        animation: none !important;
        transform: none !important;
    }
}
</style>
