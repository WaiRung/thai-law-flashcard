<template>
    <main class="main-content">
        <div class="document-container">
            <div class="header-section">
                <h1 class="page-title">Documents</h1>
                <p class="page-subtitle">
                    เอกสารอ้างอิงฉบับ PDF สำหรับอ่าน ทบทวน และเปิดต่อได้สะดวกบนมือถือและแท็บเล็ต
                </p>
                <div v-if="documentCategories.length > 0" class="summary-row" aria-label="Document summary">
                    <p class="summary-pill">{{ documentCategories.length }} หมวดหมู่</p>
                    <p class="summary-pill">{{ totalDocumentCount }} เอกสาร</p>
                </div>
            </div>

            <div v-if="openError" class="error-banner" role="alert" aria-live="polite">
                <p class="error-title">เปิดเอกสารไม่สำเร็จ</p>
                <p class="error-message">{{ openError }}</p>
            </div>

            <div v-if="documentCategories.length === 0" class="empty-state">
                <p class="empty-message">ยังไม่มีเอกสารในระบบ</p>
                <p class="empty-submessage">No documents are available right now.</p>
            </div>

            <div v-else class="categories-list">
                <section
                    v-for="category in documentCategories"
                    :key="category.categoryId"
                    class="category-section"
                >
                    <div class="category-header">
                        <div>
                            <h2 class="category-title">{{ category.nameTh }}</h2>
                            <p class="category-subtitle">{{ category.nameEn }}</p>
                        </div>
                        <p class="category-count">{{ category.files.length }} ไฟล์</p>
                    </div>

                    <div class="files-grid">
                        <button
                            v-for="(file, index) in category.files"
                            :key="index"
                            type="button"
                            class="file-card"
                            :class="{ 'is-opening': openingDocumentKey === getFileKey(category.categoryId, file.filename) }"
                            :disabled="openingDocumentKey === getFileKey(category.categoryId, file.filename)"
                            :aria-busy="openingDocumentKey === getFileKey(category.categoryId, file.filename)"
                            @click="openDocument(category, file)"
                            :aria-label="`เปิดเอกสาร ${file.nameTh}`"
                        >
                            <div class="file-icon" aria-hidden="true">
                                <svg
                                    xmlns="http://www.w3.org/2000/svg"
                                    fill="none"
                                    viewBox="0 0 24 24"
                                    stroke="currentColor"
                                    class="file-icon-svg"
                                >
                                    <path
                                        stroke-linecap="round"
                                        stroke-linejoin="round"
                                        stroke-width="1.75"
                                        d="M7.5 3.75h6.879a2.25 2.25 0 0 1 1.59.659l2.622 2.621a2.25 2.25 0 0 1 .659 1.591V19.5A2.25 2.25 0 0 1 17 21.75H7.5A2.25 2.25 0 0 1 5.25 19.5V6A2.25 2.25 0 0 1 7.5 3.75Z"
                                    />
                                    <path
                                        stroke-linecap="round"
                                        stroke-linejoin="round"
                                        stroke-width="1.75"
                                        d="M14.25 3.75V8.25H18.75"
                                    />
                                    <path
                                        stroke-linecap="round"
                                        stroke-linejoin="round"
                                        stroke-width="1.75"
                                        d="M8.25 12.75h7.5M8.25 16.5h5.25"
                                    />
                                </svg>
                            </div>
                            <div class="file-info">
                                <p class="file-title">{{ file.nameTh }}</p>
                                <p class="file-subtitle">{{ file.nameEn || 'English title unavailable' }}</p>
                                <p
                                    v-if="openingDocumentKey === getFileKey(category.categoryId, file.filename)"
                                    class="file-status"
                                >
                                    กำลังเปิดเอกสาร...
                                </p>
                            </div>
                            <div class="file-arrow" :class="{ 'is-hidden': openingDocumentKey === getFileKey(category.categoryId, file.filename) }">
                                <svg
                                    xmlns="http://www.w3.org/2000/svg"
                                    fill="none"
                                    viewBox="0 0 24 24"
                                    stroke="currentColor"
                                    class="arrow-icon"
                                >
                                    <path
                                        stroke-linecap="round"
                                        stroke-linejoin="round"
                                        stroke-width="2"
                                        d="M9 5l7 7-7 7"
                                    />
                                </svg>
                            </div>
                            <div
                                v-if="openingDocumentKey === getFileKey(category.categoryId, file.filename)"
                                class="file-loading-indicator"
                                aria-hidden="true"
                            ></div>
                        </button>
                    </div>
                </section>
            </div>
        </div>
        
        <PdfViewer
            :is-open="isPdfViewerOpen"
            :pdf-url="selectedPdfUrl"
            :title="selectedPdfTitle"
            :subtitle="selectedPdfSubtitle"
            :filename="selectedPdfFilename"
            @close="closePdfViewer"
        />
    </main>
</template>

<script setup lang="ts">
import { computed, ref, onMounted } from "vue";
import documentsConfig from "../config/documents.json";
import { getCachedDocument } from "../services/documentService";
import type { DocumentFile, DocumentCategory } from "../types/document";
import PdfViewer from "../components/PdfViewer.vue";

const documentCategories = ref<DocumentCategory[]>([]);
const baseUrl = documentsConfig.baseUrl;
const openError = ref("");
const openingDocumentKey = ref("");

const isPdfViewerOpen = ref(false);
const selectedPdfUrl = ref("");
const selectedPdfTitle = ref("");
const selectedPdfSubtitle = ref("");
const selectedPdfFilename = ref("");

const totalDocumentCount = computed(() => {
    return documentCategories.value.reduce((total: number, category: DocumentCategory) => {
        return total + category.files.length;
    }, 0);
});

const getFileKey = (categoryId: string, filename: string): string => {
    return `${categoryId}-${filename}`;
};

/**
 * Get the full URL for a document
 * First checks cache for Base64 data, falls back to remote URL
 */
const getDocumentUrl = async (categoryId: string, categoryPath: string, filename: string): Promise<string> => {
    // Try to get cached document first
    const cachedDocument = await getCachedDocument(categoryId, filename);
    if (cachedDocument) {
        return cachedDocument;
    }
    
    // Fall back to remote URL (encode filename for Thai characters and special characters)
    const encodedFilename = encodeURIComponent(filename);
    return `${baseUrl}/${categoryPath}/${encodedFilename}`;
};

/**
 * Open document in PDF viewer
 */
const openDocument = async (category: DocumentCategory, file: DocumentFile) => {
    const fileKey = getFileKey(category.categoryId, file.filename);

    try {
        openError.value = "";
        openingDocumentKey.value = fileKey;

        const documentUrl = await getDocumentUrl(category.categoryId, category.categoryPath, file.filename);

        selectedPdfUrl.value = documentUrl;
        selectedPdfTitle.value = file.nameTh;
        selectedPdfSubtitle.value = file.nameEn;
        selectedPdfFilename.value = file.filename;
        isPdfViewerOpen.value = true;
    } catch {
        openError.value = "ไม่สามารถเปิดเอกสารได้ในขณะนี้ กรุณาลองใหม่อีกครั้ง / Cannot open this document right now. Please try again.";
    } finally {
        openingDocumentKey.value = "";
    }
};

/**
 * Close PDF viewer
 */
const closePdfViewer = () => {
    isPdfViewerOpen.value = false;
    selectedPdfUrl.value = "";
    selectedPdfTitle.value = "";
    selectedPdfSubtitle.value = "";
    selectedPdfFilename.value = "";
};

/**
 * Load document categories
 */
const loadDocuments = () => {
    // Filter categories that have files
    documentCategories.value = documentsConfig.documents.filter(
        (category) => category.files.length > 0
    );
};

onMounted(() => {
    loadDocuments();
});
</script>

<style scoped>
.main-content {
    --document-ink: #1f2937;
    --document-body: #4b5563;
    --document-muted: #6b7280;
    --document-border: #e5e7eb;
    --document-surface: #ffffff;
    --document-surface-soft: #f9fafb;
    --document-accent: #3b82f6;
    --document-accent-deep: #2563eb;
    --document-focus: rgba(37, 99, 235, 0.22);

    flex: 1;
    padding: 1.25rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.5rem max(0.75rem, env(safe-area-inset-left, 0px));
    max-width: 900px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.document-container {
    display: flex;
    flex-direction: column;
    gap: 2rem;
}

.header-section {
    text-align: center;
    padding: 1.25rem 1rem 1.75rem;
    border-radius: 1rem;
    border: 1px solid rgba(229, 231, 235, 0.82);
    background: linear-gradient(180deg, rgba(255, 255, 255, 0.9) 0%, rgba(243, 244, 246, 0.9) 100%);
}

.page-title {
    font-size: clamp(2rem, 3.8vw, 2.5rem);
    font-weight: 700;
    color: var(--document-ink);
    margin: 0 0 0.5rem 0;
    line-height: 1.2;
}

.page-subtitle {
    font-size: 1.125rem;
    color: var(--document-body);
    margin: 0;
    line-height: 1.6;
    max-width: 56ch;
    margin-inline: auto;
}

.summary-row {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 0.75rem;
    margin-top: 1rem;
}

.summary-pill,
.category-count {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-height: 2rem;
    padding: 0.35rem 0.75rem;
    border-radius: 999px;
    border: 1px solid rgba(59, 130, 246, 0.16);
    background: rgba(59, 130, 246, 0.08);
    color: var(--document-accent-deep);
    font-size: 0.95rem;
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
    background: var(--document-surface);
    border-radius: 1rem;
    border: 1px solid var(--document-border);
}

.empty-message {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--document-ink);
    margin: 0 0 0.5rem 0;
}

.empty-submessage {
    font-size: 1rem;
    color: var(--document-body);
    margin: 0;
}

.categories-list {
    display: flex;
    flex-direction: column;
    gap: 2rem;
}

.category-section {
    background: var(--document-surface);
    border-radius: 1rem;
    padding: 1.25rem;
    border: 1px solid var(--document-border);
}

.category-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 1rem;
    margin-bottom: 1.5rem;
    padding-bottom: 1rem;
    border-bottom: 1px solid var(--document-border);
}

.category-title {
    font-size: clamp(1.25rem, 3vw, 1.75rem);
    font-weight: 700;
    color: var(--document-ink);
    margin: 0 0 0.25rem 0;
    overflow-wrap: anywhere;
}

.category-subtitle {
    font-size: 1rem;
    color: var(--document-body);
    margin: 0;
}

.files-grid {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.file-card {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
    background: var(--document-surface-soft);
    border: 1px solid var(--document-border);
    border-radius: 0.75rem;
    cursor: pointer;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        border-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
    width: 100%;
    text-align: left;
    min-height: 72px;
    -webkit-tap-highlight-color: transparent;
}

.file-card:disabled {
    cursor: wait;
}

.file-card.is-opening {
    background: var(--document-surface);
    border-color: rgba(59, 130, 246, 0.32);
    box-shadow: 0 4px 12px rgba(59, 130, 246, 0.1);
}

@media (hover: hover) {
    .file-card:hover {
        background: var(--document-surface);
        border-color: var(--document-accent);
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.15);
    }
}

.file-card:active {
    transform: translateY(0);
}

.file-card:focus-visible {
    outline: 3px solid var(--document-focus);
    outline-offset: 2px;
    border-color: var(--document-accent-deep);
}

.file-icon {
    flex-shrink: 0;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 3rem;
    height: 3rem;
    border-radius: 0.875rem;
    background: rgba(59, 130, 246, 0.08);
    color: var(--document-accent-deep);
}

.file-icon-svg {
    width: 1.5rem;
    height: 1.5rem;
}

.file-info {
    flex: 1;
    min-width: 0;
}

.file-title {
    font-size: 1rem;
    font-weight: 600;
    color: var(--document-ink);
    margin: 0 0 0.25rem 0;
    line-height: 1.4;
}

.file-subtitle {
    font-size: 0.875rem;
    color: var(--document-body);
    margin: 0;
    line-height: 1.45;
}

.file-status {
    margin: 0.5rem 0 0;
    font-size: 0.8125rem;
    font-weight: 600;
    color: var(--document-accent-deep);
}

.file-arrow {
    flex-shrink: 0;
}

.file-arrow.is-hidden {
    opacity: 0;
}

.arrow-icon {
    width: 1.5rem;
    height: 1.5rem;
    color: #94a3b8;
    transition:
        color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        opacity 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

@media (hover: hover) {
    .file-card:hover .arrow-icon {
        color: var(--document-accent);
        transform: translateX(4px);
    }
}

.file-loading-indicator {
    width: 1.125rem;
    height: 1.125rem;
    border-radius: 999px;
    border: 2px solid rgba(59, 130, 246, 0.2);
    border-top-color: var(--document-accent-deep);
    flex-shrink: 0;
    animation: document-spin 0.8s linear infinite;
}

@keyframes document-spin {
    to {
        transform: rotate(360deg);
    }
}

@media (max-width: 768px) {
    .main-content {
        padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .header-section {
        padding: 1rem 0.875rem 1.25rem;
    }

    .page-title {
        font-size: 2rem;
    }

    .page-subtitle {
        font-size: 1rem;
    }

    .category-section {
        padding: 1rem;
    }

    .category-header {
        flex-direction: column;
        align-items: flex-start;
    }

    .file-card {
        padding: 1rem;
    }

    .file-icon {
        width: 2.75rem;
        height: 2.75rem;
    }

    .file-title {
        font-size: 1rem;
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
}

@media (prefers-reduced-motion: reduce) {
    .file-card,
    .arrow-icon {
        transition: none;
    }

    .file-loading-indicator {
        animation: none;
    }
}
</style>
