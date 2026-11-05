<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import TeaserView from './components/TeaserView.vue'
import MainEventView from './components/MainEventView.vue'

// --- Target Date: 09 November 2026 19:00:00 ---
const TARGET_DATE = new Date('2026-11-09T19:00:00')

// Gate state: before Nov 9, default to TeaserView; on/after Nov 9, auto-unlock MainEventView
const isNov9OrLater = ref(false)
const previewOfficial = ref(false)
const isAdmin = ref(false)
const guestName = ref('')

// Grand Video Entrance State
const showIntroOverlay = ref(true)
const isPlayingIntro = ref(false)
const introVideoRef = ref(null)

function startIntroVideo() {
  isPlayingIntro = ref(true)
  isPlayingIntro.value = true
  if (introVideoRef.value) {
    introVideoRef.value.currentTime = 0
    introVideoRef.value.play().catch(err => {
      console.warn("Autoplay blocked:", err)
    })
  }
}

function finishIntroVideo() {
  showIntroOverlay.value = false
  isPlayingIntro.value = false
}

function replayIntro() {
  showIntroOverlay.value = true
  isPlayingIntro.value = false
}

function checkDateGate() {
  const now = new Date()
  isNov9OrLater.value = now.getTime() >= TARGET_DATE.getTime()

  if (typeof window !== 'undefined') {
    const params = new URLSearchParams(window.location.search)
    isAdmin.value = params.get('role') === 'admin' ||
                    params.get('admin') === 'true' ||
                    localStorage.getItem('sl_club_admin_session') === 'true'

    const name = params.get('to') || params.get('name') || params.get('guest')
    if (name && name.trim()) {
      guestName.value = name.trim()
    }
  }
}

// Current active view
const showOfficial = computed(() => isNov9OrLater.value || (isAdmin.value && previewOfficial.value))

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
    <!-- ================= GRAND LUXURY VIP VIDEO INTRO OVERLAY ================= -->
    <Transition name="fade">
      <div
        v-if="showIntroOverlay"
        class="fixed inset-0 z-[999] flex items-center justify-center bg-black overflow-hidden select-none"
      >
        <!-- Full-Screen Video Player (Plays on Guest Click) -->
        <div v-show="isPlayingIntro" class="absolute inset-0 bg-black">
          <video
            ref="introVideoRef"
            playsinline
            preload="auto"
            class="w-full h-full object-cover"
            @ended="finishIntroVideo"
          >
            <source src="/intro-video.mp4" type="video/mp4" />
          </video>

          <!-- Skip Button -->
          <button
            @click="finishIntroVideo"
            type="button"
            class="absolute top-6 right-6 z-30 flex items-center gap-1.5 px-4 py-2 rounded-full bg-black/60 hover:bg-black/80 backdrop-blur-md border border-white/20 text-white text-xs font-semibold shadow-lg hover:scale-105 active:scale-95 transition-all cursor-pointer"
          >
            <span>រំលង (Skip)</span>
            <span class="material-symbols-outlined text-sm">skip_next</span>
          </button>
        </div>

        <!-- Welcoming Invitation Card Before Click (Replaces Old CSS Loading) -->
        <div
          v-show="!isPlayingIntro"
          class="relative z-10 flex flex-col items-center text-center px-4 max-w-md mx-auto"
        >
          <!-- Background Atmospheric Glow -->
          <div class="absolute -inset-24 bg-radial-[circle_at_50%_50%] from-amber-400/25 via-amber-500/5 to-transparent blur-3xl rounded-full"></div>

          <!-- SL Club Emblem -->
          <div class="relative mb-6">
            <div class="w-28 h-28 sm:w-36 sm:h-36 rounded-full p-1 bg-gradient-to-b from-slate-200 via-amber-300 to-amber-600 shadow-[0_0_50px_rgba(251,191,36,0.6)]">
              <div class="w-full h-full rounded-full bg-black p-2 flex items-center justify-center overflow-hidden border border-amber-400/50">
                <img src="/logo.jpg" alt="SL Club" class="w-full h-full object-contain filter drop-shadow-[0_0_12px_rgba(251,191,36,0.8)]" />
              </div>
            </div>
          </div>

          <!-- VIP Honorable Guest Badge -->
          <div class="inline-flex items-center gap-1.5 px-3.5 py-1.5 rounded-full bg-black/80 border border-amber-400/40 text-amber-300 text-xs font-bold tracking-widest uppercase mb-4 shadow-md backdrop-blur-md">
            <span class="material-symbols-outlined text-sm">stars</span>
            <span>VIP EXCLUSIVE INVITATION</span>
          </div>

          <!-- Dynamic Guest Name or Default General Greeting -->
          <div v-if="guestName" class="mb-4">
            <span class="text-xs text-amber-300/90 font-medium">សូមគោរពអញ្ជើញ</span>
            <h2 class="text-2xl sm:text-3xl font-black text-transparent bg-clip-text bg-gradient-to-r from-amber-200 via-white to-amber-300 py-1 drop-shadow-[0_0_15px_rgba(251,191,36,0.7)] font-['Kantumruy_Pro','Battambang',sans-serif]">
              {{ guestName }}
            </h2>
            <span class="text-xs text-slate-300/80">ចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB</span>
          </div>
          <div v-else class="mb-4">
            <h2 class="text-xl sm:text-2xl font-bold text-white font-['Kantumruy_Pro',sans-serif]">
              សូមគោរពអញ្ជើញ ឯកឧត្តម លោកជំទាវ លោក លោកស្រី
            </h2>
            <p class="text-xs text-slate-300 mt-1">ចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB Battambang</p>
          </div>

          <p class="text-xs sm:text-sm text-slate-400 mb-8 max-w-xs leading-relaxed font-sans uppercase tracking-wider">
            GRAND OPENING VIP GALA NIGHT • 09 NOV 2026
          </p>

          <!-- Main Interactive Button to Play Video and Enter Site -->
          <button
            @click="startIntroVideo"
            type="button"
            class="group flex items-center justify-center gap-3 px-8 sm:px-10 py-3.5 sm:py-4 rounded-full bg-gradient-to-r from-amber-400 via-amber-300 to-amber-500 text-black font-extrabold text-sm sm:text-base shadow-[0_0_35px_rgba(251,191,36,0.7)] hover:shadow-[0_0_55px_rgba(251,191,36,0.95)] hover:scale-105 active:scale-95 transition-all duration-300 cursor-pointer"
          >
            <span class="material-symbols-outlined text-xl group-hover:scale-110 transition-transform">lock_open</span>
            <span>ចុចបើកសំបុត្រអញ្ជើញ VIP</span>
          </button>

          <span class="text-[11px] text-slate-400 mt-3 font-light tracking-wide">
            (ចុចដើម្បីទស្សនាវីដេអូបើកទ្វារចូលវេបសាយ)
          </span>
        </div>
      </div>
    </Transition>

    <!-- FLOATING DEV / TEST CONTROLLER (Only visible to Admin) -->
    <aside
      v-if="!isNov9OrLater && isAdmin"
      class="fixed bottom-4 right-4 z-[9999] flex items-center gap-2"
    >
      <button
        @click="replayIntro"
        type="button"
        class="flex items-center gap-1.5 px-3 py-1.5 rounded-full bg-black/85 backdrop-blur-xl border border-amber-400/50 text-amber-300 font-bold text-xs shadow-md hover:scale-105 active:scale-95 transition-all cursor-pointer"
        title="ចាក់វីដេអូបើកទ្វារឡើងវិញ"
      >
        <span class="material-symbols-outlined text-xs">replay</span>
        <span>Replay Video</span>
      </button>

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
      <!-- 1. TEASER VIEW: Shown Before November 9th -->
      <TeaserView
        v-if="!showOfficial"
        :target-date="TARGET_DATE"
        @view-official="previewOfficial = true"
      />

      <!-- 2. OFFICIAL VIP EVENT VIEW: Shown on & after November 9th -->
      <MainEventView
        v-else
        @replay-door="replayIntro"
      />
    </Transition>
  </div>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
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
