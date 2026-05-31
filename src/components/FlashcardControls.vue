<template>
    <div>
        <!-- Controls -->
        <div class="controls">
            <button
                @click="handlePrevious"
                type="button"
                :disabled="currentIndex === 0"
                class="control-btn control-prev"
                aria-label="Previous card"
            >
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                    class="btn-icon"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M15 19l-7-7 7-7"
                    />
                </svg>
                <span class="btn-text">ก่อนหน้า</span>
            </button>

            <button
                @click="handleShuffle"
                type="button"
                class="control-btn control-shuffle"
                aria-label="Shuffle cards"
            >
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                    class="btn-icon"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
                    />
                </svg>
                <span class="btn-text">สุ่ม</span>
            </button>

            <button
                @click="handleNext"
                type="button"
                :disabled="currentIndex === totalCards - 1"
                class="control-btn control-next"
                aria-label="Next card"
            >
                <span class="btn-text">ถัดไป</span>
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                    class="btn-icon"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M9 5l7 7-7 7"
                    />
                </svg>
            </button>
        </div>

        <!-- Reset Button -->
        <div class="reset-section">
            <button type="button" @click="handleReset" class="reset-btn">
                รีเซ็ตความคืบหน้า
            </button>
        </div>
    </div>
</template>

<script setup lang="ts">
interface Props {
    currentIndex: number;
    totalCards: number;
}

defineProps<Props>();

const emit = defineEmits<{
    previous: [];
    next: [];
    shuffle: [];
    reset: [];
}>();

const handlePrevious = () => {
    emit("previous");
};

const handleNext = () => {
    emit("next");
};

const handleShuffle = () => {
    emit("shuffle");
};

const handleReset = () => {
    emit("reset");
};
</script>

<style scoped>
.controls {
    display: flex;
    gap: 1rem;
    justify-content: center;
    margin-bottom: 1rem;
    position: relative;
    z-index: 1;
}

.control-btn {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 0.5rem;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition:
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        opacity 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
    color: white;
    background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
    box-shadow: 0 2px 8px rgba(59, 130, 246, 0.3);
    min-height: 44px;
    -webkit-tap-highlight-color: transparent;
}

@media (hover: hover) {
    .control-btn:hover:not(:disabled) {
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.4);
    }
}

.control-btn:active:not(:disabled) {
    transform: translateY(0);
}

.control-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    background: linear-gradient(135deg, #9ca3af 0%, #6b7280 100%);
}

.control-shuffle {
    background: linear-gradient(135deg, #06b6d4 0%, #0891b2 100%);
    box-shadow: 0 2px 8px rgba(6, 182, 212, 0.3);
}

@media (hover: hover) {
    .control-shuffle:hover {
        box-shadow: 0 4px 12px rgba(6, 182, 212, 0.4);
    }
}

.control-btn:focus-visible,
.reset-btn:focus-visible {
    outline: 3px solid rgba(37, 99, 235, 0.28);
    outline-offset: 2px;
}

.btn-icon {
    width: 1.25rem;
    height: 1.25rem;
}

.btn-text {
    font-size: 0.875rem;
}

.reset-section {
    display: flex;
    justify-content: center;
    margin-top: 1rem;
}

.reset-btn {
    padding: 0.625rem 1.25rem;
    background-color: #ef4444;
    color: white;
    border: none;
    border-radius: 0.5rem;
    font-size: 0.875rem;
    font-weight: 600;
    cursor: pointer;
    transition:
        background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
    box-shadow: 0 2px 8px rgba(239, 68, 68, 0.3);
    min-height: 44px;
    -webkit-tap-highlight-color: transparent;
}

@media (hover: hover) {
    .reset-btn:hover {
        background-color: #dc2626;
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(239, 68, 68, 0.4);
    }
}

.reset-btn:active {
    transform: translateY(0);
}

@media (max-width: 640px) {
    .controls {
        gap: 0.5rem;
        justify-content: stretch;
    }

    .control-btn {
        padding: 0.625rem 0.875rem;
        font-size: 0.875rem;
        flex: 1;
        justify-content: center;
    }

    .btn-text {
        display: none;
    }

    .btn-icon {
        width: 1.5rem;
        height: 1.5rem;
    }

    .control-shuffle .btn-icon {
        width: 1.25rem;
        height: 1.25rem;
    }

    .reset-btn {
        width: 100%;
        max-width: 18rem;
    }
}

@media (min-width: 641px) {
    .control-prev .btn-icon,
    .control-next .btn-icon {
        width: 1.5rem;
        height: 1.5rem;
    }
}

@media (prefers-reduced-motion: reduce) {
    .control-btn,
    .reset-btn {
        transition: none;
    }

    .control-btn:active:not(:disabled),
    .reset-btn:active {
        transform: none;
    }
}
</style>
