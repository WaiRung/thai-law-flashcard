<template>
    <div class="app-shell">
        <div class="app-shell__backdrop" aria-hidden="true"></div>
        <AppHeader />
        <div class="app-shell__content">
            <router-view />
        </div>
    </div>
</template>

<script setup lang="ts">
import AppHeader from "./components/AppHeader.vue";
</script>

<style scoped>
.app-shell {
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    position: relative;
    isolation: isolate;
    overflow: hidden;
    background:
        radial-gradient(circle at top left, rgba(59, 130, 246, 0.08), transparent 34%),
        radial-gradient(circle at top right, rgba(20, 184, 166, 0.06), transparent 28%),
        linear-gradient(180deg, #f8fafc 0%, #f3f4f6 46%, #eef2f7 100%);
    color: #1f2937;
    padding-bottom: env(safe-area-inset-bottom, 0px);
}

.app-shell__backdrop {
    position: absolute;
    inset: 0;
    pointer-events: none;
    background:
        radial-gradient(circle at 20% 18%, rgba(59, 130, 246, 0.06), transparent 28%),
        radial-gradient(circle at 82% 0%, rgba(79, 70, 229, 0.05), transparent 24%),
        radial-gradient(circle at 50% 100%, rgba(14, 165, 233, 0.04), transparent 30%);
}

.app-shell__content {
    position: relative;
    z-index: 1;
    flex: 1;
    display: flex;
    flex-direction: column;
    min-width: 0;
}
</style>
