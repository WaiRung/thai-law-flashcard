<template>
    <header class="app-header">
        <div class="header-content">
            <button
                v-show="showBackButton"
                @click="handleBack"
                type="button"
                class="back-btn"
                aria-label="Back to categories"
            >
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                    class="back-icon"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M15 19l-7-7 7-7"
                    />
                </svg>
            </button>
            <img src="/icon.svg" alt="Thai Law" class="app-logo" />
            <div class="header-text">
                <h1 class="app-title">{{ appTitle }}</h1>
                <p class="app-subtitle">{{ appSubtitle }}</p>
            </div>
        </div>
    </header>
</template>

<script setup lang="ts">
import { useRouter, useRoute } from "vue-router";
import { computed } from "vue";
import { useHeader } from "../composables/useHeader";

const router = useRouter();
const route = useRoute();
const { appTitle, appSubtitle } = useHeader();

// Show back button when not on the home route
const showBackButton = computed(() => route.path !== '/');

const handleBack = () => {
    router.back();
};
</script>

<style scoped>
.app-header {
    background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
    color: white;
    padding: calc(1rem + env(safe-area-inset-top, 0px)) max(1rem, env(safe-area-inset-right, 0px)) 1rem max(1rem, env(safe-area-inset-left, 0px));
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.header-content {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.875rem;
    max-width: 800px;
    margin: 0 auto;
    position: relative;
    padding: 0 3.25rem;
}

.back-btn {
    position: absolute;
    left: 0;
    background: rgba(255, 255, 255, 0.2);
    border: none;
    color: white;
    width: 44px;
    height: 44px;
    border-radius: 0.5rem;
    cursor: pointer;
    transition:
        background-color 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        transform 0.2s cubic-bezier(0.2, 0.8, 0.2, 1),
        box-shadow 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
    display: flex;
    align-items: center;
    justify-content: center;
    -webkit-tap-highlight-color: transparent;
}

@media (hover: hover) {
    .back-btn:hover {
        background: rgba(255, 255, 255, 0.3);
        transform: translateX(-2px);
    }
}

.back-btn:active {
    background: rgba(255, 255, 255, 0.3);
    transform: translateX(0);
}

.back-btn:focus-visible {
    outline: 3px solid rgba(219, 234, 254, 0.9);
    outline-offset: 2px;
    box-shadow: 0 0 0 4px rgba(37, 99, 235, 0.24);
}

.back-icon {
    width: 1.5rem;
    height: 1.5rem;
}

.app-logo {
    width: 3rem;
    height: 3rem;
    filter: brightness(0) invert(1);
}

.header-text {
    text-align: center;
    min-width: 0;
}

.app-title {
    font-size: clamp(1.25rem, 4.5vw, 1.875rem);
    font-weight: 700;
    margin: 0;
    margin-bottom: 0.375rem;
    line-height: 1.2;
    overflow-wrap: anywhere;
}

.app-subtitle {
    font-size: clamp(0.875rem, 2.8vw, 1rem);
    margin: 0;
    opacity: 0.9;
    line-height: 1.4;
    overflow-wrap: anywhere;
}

@media (max-width: 640px) {
    .app-header {
        padding: calc(0.75rem + env(safe-area-inset-top, 0px)) max(0.75rem, env(safe-area-inset-right, 0px)) 0.75rem max(0.75rem, env(safe-area-inset-left, 0px));
    }

    .header-content {
        gap: 0.5rem;
        padding: 0 2.9rem;
    }

    .back-icon {
        width: 1.25rem;
        height: 1.25rem;
    }

    .app-logo {
        width: 2.5rem;
        height: 2.5rem;
    }

    .app-subtitle {
        max-width: 34ch;
        margin-inline: auto;
    }
}

@media (prefers-reduced-motion: reduce) {
    .back-btn {
        transition: none;
    }
}
</style>
