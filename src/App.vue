<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import TeaserView from './components/TeaserView.vue'
import MainEventView from './components/MainEventView.vue'

// --- Target Date: 09 November 2026 19:00:00 ---
const TARGET_DATE = new Date('2026-11-09T19:00:00')

// Gate state: before Nov 9, default to TeaserView; on/after Nov 9, auto-unlock MainEventView
const isNov9OrLater = ref(false)
const previewOfficial = ref(false)

function checkDateGate() {
  const now = new Date()
  isNov9OrLater.value = now.getTime() >= TARGET_DATE.getTime()
}

// Current active view
const showOfficial = computed(() => isNov9OrLater.value || previewOfficial.value)

let dateCheckInterval = null

onMounted(() => {
  checkDateGate()
  dateCheckInterval = setInterval(checkDateGate, 1000)
})

onUnmounted(() => {
  if (dateCheckInterval) clearInterval(dateCheckInterval)
})
</script>

<template>
  <div class="relative min-h-screen bg-[#08080a] text-slate-100 overflow-x-hidden">
    <!-- FLOATING DEV / TEST CONTROLLER (Only visible when user toggles preview) -->
    <aside
      v-if="!isNov9OrLater"
      class="fixed bottom-4 right-4 z-[9999] flex items-center gap-2"
    >
      <button
        v-if="previewOfficial"
        @click="previewOfficial = false"
        type="button"
        class="flex items-center gap-2 px-4 py-2 rounded-full bg-black/85 backdrop-blur-xl border border-amber-400 text-amber-300 font-bold text-xs sm:text-sm shadow-[0_0_20px_rgba(251,191,36,0.5)] hover:scale-105 active:scale-95 transition-all cursor-pointer"
      >
        <span class="material-symbols-outlined text-sm">arrow_back</span>
        <span>ត្រឡប់ទៅទំព័រ Teaser (មុនថ្ងៃទី 9)</span>
      </button>
    </aside>

    <!-- DYNAMIC VIEW RENDERER -->
    <Transition name="fade" mode="out-in">
      <!-- 1. TEASER VIEW: Shown Before November 9th (Video Background only, no music) -->
      <TeaserView
        v-if="!showOfficial"
        :target-date="TARGET_DATE"
        @view-official="previewOfficial = true"
      />

      <!-- 2. OFFICIAL VIP EVENT VIEW: Shown on & after November 9th -->
      <MainEventView
        v-else
      />
    </Transition>
  </div>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease-in-out, transform 0.4s ease-in-out;
}

.fade-enter-from {
  opacity: 0;
  transform: scale(0.98);
}

.fade-leave-to {
  opacity: 0;
  transform: scale(1.02);
}
</style>
