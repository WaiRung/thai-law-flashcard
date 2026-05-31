<template>
    <main class="main-content">
        <div class="document-container">
            <div class="header-section">
                <h1 class="page-title">Documents</h1>
                <p class="page-subtitle">เอกสาร PDF สำหรับอ่านบนมือถือและแท็บเล็ต</p>
            </div>

            <div v-if="documentCategories.length === 0" class="empty-state">
                <p class="empty-message">ไม่มีเอกสารในขณะนี้</p>
                <p class="empty-submessage">No documents available at the moment</p>
            </div>

            <div v-else class="categories-list">
                <div
                    v-for="category in documentCategories"
                    :key="category.categoryId"
                    class="category-section"
                >
                    <div class="category-header">
                        <h3 class="category-title">{{ category.nameTh }}</h3>
                        <p class="category-subtitle">{{ category.nameEn }}</p>
                    </div>

                    <div class="files-grid">
                        <button
                            v-for="(file, index) in category.files"
                            :key="index"
                            type="button"
                            class="file-card"
                            @click="openDocument(category, file)"
                            :aria-label="`เปิดเอกสาร ${file.nameTh}`"
                        >
                            <div class="file-icon">📄</div>
                            <div class="file-info">
                                <p class="file-title">{{ file.nameTh }}</p>
                                <p class="file-subtitle">{{ file.nameEn }}</p>
                            </div>
                            <div class="file-arrow">
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
                        </button>
                    </div>
                </div>
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
import { ref, onMounted } from "vue";
import documentsConfig from "../config/documents.json";
import { getCachedDocument } from "../services/documentService";
import type { DocumentFile, DocumentCategory } from "../types/document";
import PdfViewer from "../components/PdfViewer.vue";

const documentCategories = ref<DocumentCategory[]>([]);
const baseUrl = documentsConfig.baseUrl;

// PDF viewer state
const isPdfViewerOpen = ref(false);
const selectedPdfUrl = ref("");
const selectedPdfTitle = ref("");
const selectedPdfSubtitle = ref("");
const selectedPdfFilename = ref("");

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
    try {
        const documentUrl = await getDocumentUrl(category.categoryId, category.categoryPath, file.filename);
        
        // Open PDF in viewer
        selectedPdfUrl.value = documentUrl;
        selectedPdfTitle.value = file.nameTh;
        selectedPdfSubtitle.value = file.nameEn;
        selectedPdfFilename.value = file.filename;
        isPdfViewerOpen.value = true;
    } catch (error) {
        console.error('Error opening document:', error);
        // Note: In a production app, this should use a toast/notification component
        // For now, using alert as a simple fallback
        alert('ไม่สามารถเปิดเอกสารได้ / Cannot open document');
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
    padding: 1rem 0 1.5rem;
}

.page-title {
    font-size: clamp(1.875rem, 4vw, 2.5rem);
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
    font-size: 2.5rem;
    flex-shrink: 0;
}

.file-info {
    flex: 1;
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

.file-arrow {
    flex-shrink: 0;
}

.arrow-icon {
    width: 1.5rem;
    height: 1.5rem;
    color: #94a3b8;
    transition: all 0.2s;
}

@media (hover: hover) {
    .file-card:hover .arrow-icon {
        color: var(--document-accent);
        transform: translateX(4px);
    }
}

@media (max-width: 768px) {
    .main-content {
        padding: 1rem max(0.75rem, env(safe-area-inset-right, 0px)) 1.25rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .header-section {
        padding: 0.5rem 0 1rem;
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

    .file-card {
        padding: 1rem;
    }

    .file-icon {
        font-size: 2rem;
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
}
</style>
