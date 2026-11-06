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

// Admin Login State
const isAdminLoginOpen = ref(false)
const adminPinInput = ref('')
const adminPinError = ref(false)

function openAdminLogin() {
  adminPinInput.value = ''
  adminPinError.value = false
  isAdminLoginOpen.value = true
}

function verifyAdminPin() {
  if (adminPinInput.value === '2026' || adminPinInput.value === '8888' || adminPinInput.value === '1234') {
    isAdmin.value = true
    localStorage.setItem('sl_club_admin_session', 'true')
    isAdminLoginOpen.value = false
    showIntroOverlay.value = false // Skip intro for admin
  } else {
    adminPinError.value = true
  }
}

function autoStartIntro() {
  isPlayingIntro.value = true
  showIntroOverlay.value = true

  // Auto-play the opening entrance video automatically on link load
  if (introVideoRef.value) {
    try {
      introVideoRef.value.muted = true
      introVideoRef.value.currentTime = 0
      introVideoRef.value.playbackRate = 1.25 // Smooth & brisk 5s opening
      const playPromise = introVideoRef.value.play()
      if (playPromise !== undefined) {
        playPromise.catch(err => {
          console.warn("Autoplay was prevented by browser policy, opening directly:", err)
          finishIntroVideo()
        })
      }
    } catch (e) {
      console.warn("Autoplay exception, opening directly:", e)
      finishIntroVideo()
    }
  }

  // Safety timer fallback: auto-finish within 5 seconds so guest is never waiting
  setTimeout(() => {
    if (showIntroOverlay.value) {
      finishIntroVideo()
    }
  }, 5000)
}

function startIntroVideo() {
  autoStartIntro()
}

function finishIntroVideo() {
  showIntroOverlay.value = false
  isPlayingIntro.value = false
}

function replayIntro() {
  showIntroOverlay.value = true
  autoStartIntro()
}

function checkDateGate() {
  const now = new Date()
  isNov9OrLater.value = now.getTime() >= TARGET_DATE.getTime()

  if (typeof window !== 'undefined') {
    const params = new URLSearchParams(window.location.search)
    const isAdminParam = params.get('role') === 'admin' ||
                         params.get('admin') === 'true' ||
                         params.get('admin') === '1' ||
                         localStorage.getItem('sl_club_admin_session') === 'true'
    isAdmin.value = isAdminParam
    if (params.get('role') === 'admin' || params.get('admin') === 'true') {
      showIntroOverlay.value = false // Skip intro directly for admin
    }

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

  // Open auto right away when guest opens link - no button click needed
  if (!isAdmin.value) {
    autoStartIntro()
  } else {
    showIntroOverlay.value = false
  }
})

onUnmounted(() => {
  if (dateCheckInterval) clearInterval(dateCheckInterval)
})
</script>

<template>
  <div class="relative min-h-screen bg-[#08080a] text-slate-100 overflow-x-hidden">
    <!-- ================= GRAND LUXURY VIP VIDEO INTRO OVERLAY (AUTO-OPENS ON LINK LOAD) ================= -->
    <Transition name="fade">
      <div
        v-if="showIntroOverlay"
        class="fixed inset-0 z-[999] flex items-center justify-center bg-black overflow-hidden select-none cursor-pointer"
        @click="finishIntroVideo"
        title="ចុចលើអេក្រង់ដើម្បីរំលងចូលទំព័រភ្លាមៗ"
      >
        <!-- Full-Screen Video Player Layer -->
        <div class="absolute inset-0 bg-black pointer-events-auto">
          <video
            ref="introVideoRef"
            autoplay
            playsinline
            webkit-playsinline
            muted
            preload="auto"
            class="w-full h-full object-cover"
            @ended="finishIntroVideo"
            @error="finishIntroVideo"
          >
            <source src="/intro-video.mp4" type="video/mp4" />
          </video>

          <!-- Top Navigation Bar -->
          <div class="absolute top-4 sm:top-6 inset-x-4 sm:inset-x-6 z-40 flex items-center justify-between pointer-events-none">
            <!-- Discreet Admin Portal Button -->
            <button
              @click.stop="openAdminLogin"
              type="button"
              class="flex items-center gap-1.5 px-3 py-1.5 rounded-full bg-black/60 hover:bg-black/80 backdrop-blur-md border border-white/20 text-slate-300 hover:text-amber-400 text-xs font-medium cursor-pointer pointer-events-auto transition-colors"
            >
              <span class="material-symbols-outlined text-xs text-amber-400">lock</span>
              <span>Admin</span>
            </button>

            <!-- Skip Button -->
            <button
              @click.stop="finishIntroVideo"
              type="button"
              class="flex items-center gap-1.5 px-4 py-2 rounded-full bg-black/60 hover:bg-black/80 backdrop-blur-md border border-white/20 text-white text-xs font-semibold shadow-lg hover:scale-105 active:scale-95 transition-all cursor-pointer pointer-events-auto"
            >
              <span>រំលង (Skip)</span>
              <span class="material-symbols-outlined text-sm">skip_next</span>
            </button>
          </div>

          <!-- Bottom Cue -->
          <div class="absolute bottom-6 sm:bottom-8 inset-x-0 z-30 flex justify-center pointer-events-none">
            <span class="px-4 py-1.5 rounded-full bg-black/60 backdrop-blur-md border border-white/10 text-slate-300 text-[11px] font-medium tracking-wider">
              (ចុចលើអេក្រង់ដើម្បីចូលទំព័រភ្លាមៗ • Tap to skip)
            </span>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ADMIN LOGIN PIN MODAL -->
    <Transition name="fade">
      <div
        v-if="isAdminLoginOpen"
        class="fixed inset-0 z-[1000] flex items-center justify-center p-4 bg-black/90 backdrop-blur-md"
        @click.self="isAdminLoginOpen = false"
      >
        <div class="relative w-full max-w-sm rounded-2xl bg-[#12131a] border border-amber-400/40 p-6 shadow-2xl flex flex-col gap-4 text-center">
          <div class="w-12 h-12 rounded-full bg-amber-400/20 border border-amber-400/40 flex items-center justify-center text-amber-400 mx-auto">
            <span class="material-symbols-outlined text-2xl">lock</span>
          </div>

          <div>
            <h3 class="text-lg font-bold text-white font-['Kantumruy_Pro',sans-serif]">ចូលកាន់ Admin Portal</h3>
            <p class="text-xs text-slate-400 mt-1">សូមបញ្ចូលលេខកូដសម្ងាត់ Admin PIN (2026)</p>
          </div>

          <div class="flex flex-col gap-2">
            <input
              v-model="adminPinInput"
              type="password"
              placeholder="វាយលេខកូដ PIN..."
              class="w-full text-center tracking-widest text-lg font-mono px-4 py-3 rounded-xl bg-white/5 border text-white placeholder-slate-500 focus:outline-none"
              :class="adminPinError ? 'border-red-500' : 'border-white/20 focus:border-amber-400'"
              @keyup.enter="verifyAdminPin"
              autofocus
            />
            <span v-if="adminPinError" class="text-xs text-red-400">លេខកូដមិនត្រឹមត្រូវឡើយ (PIN: 2026)</span>
          </div>

          <div class="flex items-center gap-2 pt-1">
            <button
              @click="isAdminLoginOpen = false"
              type="button"
              class="flex-1 py-2.5 rounded-xl bg-white/10 hover:bg-white/20 text-slate-300 text-xs font-semibold cursor-pointer"
            >
              បោះបង់
            </button>
            <button
              @click="verifyAdminPin"
              type="button"
              class="flex-1 py-2.5 rounded-xl bg-gradient-to-r from-amber-400 to-amber-500 text-black text-xs font-bold shadow-md cursor-pointer"
            >
              ចូល Admin
            </button>
          </div>
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
