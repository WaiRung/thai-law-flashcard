<template>
    <main class="main-content">
        <div class="menu-container">
            <div class="welcome-section">
                <h2 class="welcome-title">ยินดีต้อนรับ</h2>
                <p class="welcome-subtitle">Welcome to Thai Law</p>
            </div>

            <!-- Cache Status Section - Main functionality -->
            <CacheStatus
                :is-cache-available="isCacheAvailable"
                :cache-metadata="cacheMetadata"
                :is-downloading="isDownloading"
                :download-success="downloadSuccess"
                @reload="handleDownloadData"
            />

            <div class="menu-items">
                <button type="button" @click="navigateToSections" class="menu-item">
                    <div class="menu-item-icon">📜</div>
                    <div class="menu-item-content">
                        <h3 class="menu-item-title">รายการมาตรา</h3>
                        <p class="menu-item-description">ดูรายการมาตราทั้งหมด</p>
                    </div>
                    <div class="menu-item-arrow">
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

                <button type="button" @click="navigateToQuizLaw" class="menu-item">
                    <div class="menu-item-icon">🎯</div>
                    <div class="menu-item-content">
                        <h3 class="menu-item-title">QuizLaw</h3>
                        <p class="menu-item-description">ทดสอบความรู้กฎหมายแบบเกม</p>
                    </div>
                    <div class="menu-item-arrow">
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

                <button type="button" @click="navigateToFlashcards" class="menu-item">
                    <div class="menu-item-icon">🎴</div>
                    <div class="menu-item-content">
                        <h3 class="menu-item-title">Flashcards</h3>
                        <p class="menu-item-description">เรียนรู้กฎหมายไทยด้วย Flashcards</p>
                    </div>
                    <div class="menu-item-arrow">
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

                <button type="button" @click="navigateToDiagrams" class="menu-item">
                    <div class="menu-item-icon">📊</div>
                    <div class="menu-item-content">
                        <h3 class="menu-item-title">Diagrams</h3>
                        <p class="menu-item-description">ดู Diagrams ประกอบกฎหมาย</p>
                    </div>
                    <div class="menu-item-arrow">
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

                <button type="button" @click="navigateToDocuments" class="menu-item">
                    <div class="menu-item-icon">📄</div>
                    <div class="menu-item-content">
                        <h3 class="menu-item-title">Documents</h3>
                        <p class="menu-item-description">ดูเอกสาร PDF ประกอบกฎหมาย</p>
                    </div>
                    <div class="menu-item-arrow">
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
    </main>
</template>

<script setup lang="ts">
import { onMounted } from "vue";
import { useRouter } from "vue-router";
import CacheStatus from "../components/CacheStatus.vue";
import { useDataManager } from "../composables/useDataManager";

const router = useRouter();

// Use data manager composable
const {
    isCacheAvailable,
    cacheMetadata,
    isDownloading,
    downloadSuccess,
    checkCache,
    downloadData,
} = useDataManager();

const navigateToFlashcards = () => {
    router.push({ name: "flashcard-categories" });
};

const navigateToSections = () => {
    router.push({ name: "sections-list" });
};

const navigateToQuizLaw = () => {
    router.push({ name: "quizlaw-categories" });
};

const navigateToDiagrams = () => {
    router.push({ name: "diagrams" });
};

const navigateToDocuments = () => {
    router.push({ name: "documents" });
};

const handleDownloadData = async () => {
    await downloadData();
};

// Initialize on mount
onMounted(async () => {
    // Check if cache is available on page load
    await checkCache();
});
</script>

<style scoped>
.main-content {
    --menu-ink: #1f2937;
    --menu-body: #4b5563;
    --menu-muted: #6b7280;
    --menu-border: #e5e7eb;
    --menu-surface: #ffffff;
    --menu-surface-soft: #f9fafb;
    --menu-accent: #3b82f6;
    --menu-accent-deep: #2563eb;
    --menu-focus: rgba(59, 130, 246, 0.26);

    flex: 1;
    padding: 1.5rem 1rem;
    max-width: 600px;
    width: 100%;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
}

.menu-container {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
}

.welcome-section {
    border-radius: 1rem;
    background:
        linear-gradient(180deg, rgba(255, 255, 255, 0.65) 0%, rgba(243, 244, 246, 0.7) 100%);
    border: 1px solid rgba(229, 231, 235, 0.85);
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
    text-align: center;
    padding: 1.5rem 1rem;
}

.welcome-title {
    font-size: 2.5rem;
    font-weight: 700;
    color: var(--menu-ink);
    margin: 0 0 0.5rem 0;
}

.welcome-subtitle {
    font-size: 1.125rem;
    color: var(--menu-body);
    line-height: 1.6;
    margin: 0 auto;
    max-width: 44ch;
    overflow-wrap: anywhere;
}

.menu-items {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.menu-item {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1.5rem;
    background: var(--menu-surface);
    border: 2px solid var(--menu-border);
    border-radius: 1rem;
    cursor: pointer;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        border-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
    text-align: left;
    width: 100%;
    min-height: 88px;
    -webkit-tap-highlight-color: transparent;
    animation: menuItemEnter 0.32s cubic-bezier(0.2, 0.8, 0.2, 1) both;
}

.menu-item:hover {
    background: var(--menu-surface-soft);
    border-color: var(--menu-accent);
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(59, 130, 246, 0.15);
}

.menu-item:active {
    transform: translateY(0);
}

.menu-item:focus-visible {
    outline: 3px solid var(--menu-focus);
    outline-offset: 2px;
    border-color: var(--menu-accent-deep);
    box-shadow: 0 0 0 4px rgba(37, 99, 235, 0.16);
}

.menu-item:nth-child(1) {
    animation-delay: 0.04s;
}

.menu-item:nth-child(2) {
    animation-delay: 0.08s;
}

.menu-item:nth-child(3) {
    animation-delay: 0.12s;
}

.menu-item:nth-child(4) {
    animation-delay: 0.16s;
}

.menu-item:nth-child(5) {
    animation-delay: 0.2s;
}

.menu-item-icon {
    font-size: 3rem;
    flex-shrink: 0;
}

.menu-item-content {
    flex: 1;
}

.menu-item-title {
    font-size: 1.5rem;
    font-weight: 700;
    color: var(--menu-ink);
    margin: 0 0 0.25rem 0;
    line-height: 1.3;
    overflow-wrap: anywhere;
}

.menu-item-description {
    font-size: 1rem;
    color: var(--menu-body);
    line-height: 1.5;
    margin: 0;
    overflow-wrap: anywhere;
}

.menu-item-arrow {
    flex-shrink: 0;
}

.arrow-icon {
    width: 1.5rem;
    height: 1.5rem;
    color: #9ca3af;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.menu-item:hover .arrow-icon {
    color: var(--menu-accent);
    transform: translateX(4px);
}

.menu-item:hover .menu-item-icon {
    transform: translateY(-1px);
}

.menu-item-icon {
    transition: transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
}

@keyframes menuItemEnter {
    from {
        opacity: 0;
        transform: translateY(8px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@media (prefers-reduced-motion: reduce) {
    .menu-item,
    .menu-item-icon,
    .arrow-icon {
        animation: none;
        transition: none;
    }

    .menu-item:hover,
    .menu-item:active {
        transform: none;
    }

    .menu-item:hover .arrow-icon,
    .menu-item:hover .menu-item-icon {
        transform: none;
    }
}

@media (max-width: 640px) {
    .main-content {
        padding: 1rem 0.75rem;
    }

    .welcome-section {
        padding: 1.25rem 0.75rem;
    }

    .welcome-title {
        font-size: 2rem;
    }

    .welcome-subtitle {
        font-size: 1rem;
    }

    .menu-item {
        padding: 1.25rem;
    }

    .menu-item-icon {
        font-size: 2.5rem;
    }

    .menu-item-title {
        font-size: 1.25rem;
    }

    .menu-item-description {
        font-size: 0.875rem;
    }
}
</style>
