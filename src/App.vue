<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// --- Reload / Grand Entrance Door State ---
const isDoorOpen = ref(false)
const showIntroScreen = ref(true)

// --- Active Page State (Artboard 1 vs Page 2) ---
// 1 = ទំព័រទី ១: សំបុត្រគោរពអញ្ជើញ (Artboard 1 Invitation)
// 2 = ទំព័រទី ២: ភាពកក្រើក និងសប្បាយរីករាយជាមួយ SL CLUB
const currentPage = ref(1)

function setPage(page) {
  currentPage.value = page
}

function nextPage() {
  currentPage.value = currentPage.value === 1 ? 2 : 1
}

function prevPage() {
  currentPage.value = currentPage.value === 2 ? 1 : 2
}

// --- Hamburger Menu State (ទ្រេ ៣) ---
const isMenuOpen = ref(false)

function toggleMenu() {
  isMenuOpen.value = !isMenuOpen.value
}

function closeMenu() {
  isMenuOpen.value = false
}

function handleMenuAction(action) {
  closeMenu()
  if (action === 'page1') setPage(1)
  if (action === 'page2') setPage(2)
  if (action === 'replayDoor') replayDoor()
  if (action === 'openModal') openReservationModal()
  if (action === 'directions') openDirections()
}

// --- Background Video State & Controls ---
const videoRef = ref(null)
const isVideoPlaying = ref(true)

// --- Background Music State & Controls ---
const audioRef = ref(null)
const isMusicPlaying = ref(false)

function toggleVideo() {
  if (!videoRef.value) return
  if (videoRef.value.paused) {
    videoRef.value.play()
    isVideoPlaying.value = true
  } else {
    videoRef.value.pause()
    isVideoPlaying.value = false
  }
}

function playMusic() {
  if (audioRef.value) {
    audioRef.value.play().then(() => {
      isMusicPlaying.value = true
    }).catch((err) => {
      console.warn("Autoplay gesture required:", err)
    })
  }
}

function pauseMusic() {
  if (audioRef.value) {
    audioRef.value.pause()
    isMusicPlaying.value = false
  }
}

function toggleMusic() {
  if (isMusicPlaying.value) {
    pauseMusic()
  } else {
    playMusic()
  }
}

function openDoor() {
  if (isDoorOpen.value) return
  isDoorOpen.value = true
  playMusic() // Synchronously starts the looping background song on door opening
  setTimeout(() => {
    showIntroScreen.value = false
  }, 1200)
}

function replayDoor() {
  showIntroScreen.value = true
  isDoorOpen.value = false
}

// --- VIP Tiers ---
const tiers = [
  {
    id: 'general',
    title: 'កាត VIP Club Pass',
    fullName: 'កាតចូលរួមទូទៅ VIP Club Pass',
    cost: '$150',
    description: 'ចូលទស្សនាការប្រគំតន្ត្រី + តំបន់រាំ Dance Floor & Welcome Drink',
    features: [
      'ចូលទស្សនាការប្រគំតន្ត្រី VIP Live Concert',
      'តំបន់រាំកម្សាន្ត Dance Floor ក្បែរឆាក',
      'ភេសជ្ជៈទទួលស្វាគមន៍ Welcome Drink ១ កែវ',
      'សេវាចតរថយន្តទូទៅ'
    ]
  },
  {
    id: 'founder',
    title: 'តុពិសេស Founder Skybox VIP',
    fullName: 'តុពិសេស Founder Skybox VIP',
    cost: '$450',
    description: 'សេវាស្រាសំប៉ាញផ្ទាល់, ម្ហូប Fine Dining, ទិដ្ឋភាពល្អបំផុត & បារឯកជន',
    features: [
      'តុពិសេស VIP Skybox ទិដ្ឋភាពល្អផ្តាច់គេ',
      'ស្រាសំប៉ាញ Dom Pérignon & Reserve Mixology',
      'ម្ហូបបែប Fine Dining ពិសេសចម្អិនផ្ទាល់',
      'អ្នកបម្រើផ្ទាល់ (Dedicated VIP Butler)',
      'សេវាចតរថយន្ត VIP Porte-cochère'
    ]
  }
]

const selectedTier = ref(tiers[1])
const isModalOpen = ref(false)
const reservationSuccess = ref(false)
const guestName = ref('')
const guestPhone = ref('')

// Countdown Logic with Khmer Numerals
const countdownText = ref('០៤ ថ្ងៃ : ០៨ ម៉ោង : ៣២ នាទី : ៤៥ វិនាទី')
let countdownInterval = null

const khmerNumerals = ['០', '១', '២', '៣', '៤', '៥', '៦', '៧', '៨', '៩']
function toKhmerNum(num) {
  return String(num)
    .padStart(2, '0')
    .split('')
    .map(d => khmerNumerals[parseInt(d)] || d)
    .join('')
}

onMounted(() => {
  let seconds = 45
  let minutes = 32
  let hours = 8
  let days = 4

  countdownInterval = setInterval(() => {
    if (seconds > 0) {
      seconds--
    } else {
      seconds = 59
      if (minutes > 0) {
        minutes--
      } else {
        minutes = 59
        if (hours > 0) {
          hours--
        } else {
          hours = 23
          if (days > 0) days--
        }
      }
    }
    countdownText.value = `${toKhmerNum(days)} ថ្ងៃ : ${toKhmerNum(hours)} ម៉ោង : ${toKhmerNum(minutes)} នាទី : ${toKhmerNum(seconds)} វិនាទី`
  }, 1000)
})

onUnmounted(() => {
  if (countdownInterval) clearInterval(countdownInterval)
})

function openReservationModal(tier) {
  if (tier) selectedTier.value = tier
  reservationSuccess.value = false
  isModalOpen.value = true
}

function closeReservationModal() {
  isModalOpen.value = false
}

function confirmReservation() {
  if (!guestName.value.trim() && !guestPhone.value.trim()) {
    alert('សូមបំពេញឈ្មោះ និងលេខទូរស័ព្ទរបស់អ្នកដើម្បីកក់កន្លែង។')
    return
  }
  reservationSuccess.value = true
  setTimeout(() => {
    closeReservationModal()
    alert('សូមអរគុណ! ក្រុមការងារ Concierge នៃ SL Club នឹងទាក់ទងមកលោកអ្នកក្នុងរយៈពេល ២ ម៉ោង ដើម្បីរៀបចំការទទួលស្វាគមន៍ VIP។')
    guestName.value = ''
    guestPhone.value = ''
  }, 400)
}

function openDirections() {
  window.open('https://maps.app.goo.gl/hSJKgn4A28KUYYnm8', '_blank')
}
</script>

<template>
  <div class="min-h-screen flex flex-col bg-[#0b0b0d] font-body-md text-on-surface antialiased selection:bg-primary selection:text-on-primary relative overflow-x-hidden">

    <!-- ============================================================== -->
    <!-- CONTINUOUS LOOPING VIP BACKGROUND VIDEO & AUDIO -->
    <!-- ============================================================== -->
    <div class="fixed inset-0 z-0 overflow-hidden pointer-events-none">
      <video
        ref="videoRef"
        autoplay
        loop
        muted
        playsinline
        preload="auto"
        class="w-full h-full object-cover object-center filter brightness-[0.88] contrast-[1.05]"
        src="/bg-video.mp4"
      ></video>
      <!-- Translucent dark & golden ambient vignette: Keeps video vivid and visible while ensuring crisp text readability -->
      <div class="absolute inset-0 bg-gradient-to-b from-black/55 via-black/35 to-black/75"></div>
      <div class="absolute inset-0 bg-[radial-gradient(circle_at_center,_rgba(242,202,80,0.08)_0%,_transparent_70%)]"></div>
    </div>

    <!-- BGM Audio Element with Continuous Loop -->
    <audio
      ref="audioRef"
      loop
      preload="auto"
      src="/bg-music.mp3"
    ></audio>

    <!-- ============================================================== -->
    <!-- GRAND LUXURY VIP DOUBLE-DOOR INTRO / RELOAD SCREEN -->
    <!-- ============================================================== -->
    <div
      v-if="showIntroScreen"
      class="fixed inset-0 z-[100] flex items-center justify-center overflow-hidden bg-black select-none"
    >
      <!-- LEFT DOOR -->
      <div
        :class="[
          'absolute top-0 left-0 w-1/2 h-full bg-gradient-to-r from-[#0a0a0a] via-[#121212] to-[#1a1919] border-r-2 border-primary/50 flex items-center justify-end pr-2 md:pr-6 transition-transform duration-1000 ease-[cubic-bezier(0.77,0,0.175,1)] shadow-[inset_-25px_0_60px_rgba(0,0,0,0.95)]',
          isDoorOpen ? '-translate-x-full' : 'translate-x-0'
        ]"
      >
        <div class="h-5/6 w-4/5 border border-primary/20 rounded-l-2xl relative mr-4 p-6 hidden md:flex flex-col justify-between">
          <div class="w-12 h-12 border-t-2 border-l-2 border-primary/60"></div>
          <div class="space-y-6 opacity-30">
            <div class="h-0.5 w-full bg-gradient-to-r from-transparent via-primary/50 to-primary"></div>
            <div class="h-0.5 w-3/4 bg-gradient-to-r from-transparent via-primary/50 to-primary ml-auto"></div>
            <div class="h-0.5 w-1/2 bg-gradient-to-r from-transparent via-primary/50 to-primary ml-auto"></div>
          </div>
          <div class="w-12 h-12 border-b-2 border-l-2 border-primary/60"></div>
        </div>
        <div class="w-3 md:w-4 h-36 md:h-52 rounded-full bg-gradient-to-b from-primary via-primary-container to-secondary-fixed shadow-[0_0_20px_rgba(242,202,80,0.6)] mr-1"></div>
      </div>

      <!-- RIGHT DOOR -->
      <div
        :class="[
          'absolute top-0 right-0 w-1/2 h-full bg-gradient-to-l from-[#0a0a0a] via-[#121212] to-[#1a1919] border-l-2 border-primary/50 flex items-center justify-start pl-2 md:pl-6 transition-transform duration-1000 ease-[cubic-bezier(0.77,0,0.175,1)] shadow-[inset_25px_0_60px_rgba(0,0,0,0.95)]',
          isDoorOpen ? 'translate-x-full' : 'translate-x-0'
        ]"
      >
        <div class="w-3 md:w-4 h-36 md:h-52 rounded-full bg-gradient-to-b from-primary via-primary-container to-secondary-fixed shadow-[0_0_20px_rgba(242,202,80,0.6)] ml-1"></div>
        <div class="h-5/6 w-4/5 border border-primary/20 rounded-r-2xl relative ml-4 p-6 hidden md:flex flex-col justify-between">
          <div class="w-12 h-12 border-t-2 border-r-2 border-primary/60 ml-auto"></div>
          <div class="space-y-6 opacity-30">
            <div class="h-0.5 w-full bg-gradient-to-l from-transparent via-primary/50 to-primary"></div>
            <div class="h-0.5 w-3/4 bg-gradient-to-l from-transparent via-primary/50 to-primary mr-auto"></div>
            <div class="h-0.5 w-1/2 bg-gradient-to-l from-transparent via-primary/50 to-primary mr-auto"></div>
          </div>
          <div class="w-12 h-12 border-b-2 border-r-2 border-primary/60 ml-auto"></div>
        </div>
      </div>

      <!-- GOLDEN LIGHT BLOOM WHEN OPENING -->
      <div
        :class="[
          'absolute inset-0 pointer-events-none transition-opacity duration-700 bg-gradient-radial from-primary/30 via-transparent to-transparent z-10 flex items-center justify-center',
          isDoorOpen ? 'opacity-100 scale-150' : 'opacity-0 scale-90'
        ]"
      >
        <div class="w-96 md:w-[600px] h-96 md:h-[600px] rounded-full bg-primary/25 blur-[120px]"></div>
      </div>

      <!-- CENTER VIP EMBLEM & ENTER BUTTON -->
      <div
        :class="[
          'relative z-20 flex flex-col items-center text-center px-4 sm:px-6 transition-all duration-700 max-w-lg mx-auto',
          isDoorOpen ? 'opacity-0 scale-110 pointer-events-none' : 'opacity-100 scale-100'
        ]"
      >
        <div class="relative mb-5 md:mb-6">
          <div class="absolute inset-0 rounded-full bg-primary/30 blur-2xl animate-pulse"></div>
          <div class="relative w-28 h-28 sm:w-36 sm:h-36 md:w-40 md:h-40 rounded-full p-1 bg-gradient-to-b from-secondary-fixed via-primary to-primary-container shadow-[0_0_40px_rgba(242,202,80,0.5)]">
            <div class="w-full h-full rounded-full bg-black/95 p-3 flex items-center justify-center overflow-hidden border border-primary/40">
              <img
                src="/logo.jpg"
                alt="SL Club Logo"
                class="w-full h-full object-contain filter drop-shadow-[0_0_12px_rgba(242,202,80,0.7)]"
              />
            </div>
          </div>
        </div>

        <div class="inline-flex items-center gap-2 mb-3 px-4 py-1.5 rounded-full bg-black/80 border border-primary/30 shadow-md backdrop-blur-md">
          <span class="material-symbols-outlined text-primary text-[15px]">stars</span>
          <span class="font-label-sm text-xs sm:text-sm text-primary font-bold tracking-widest uppercase">
            VIP EXCLUSIVE INVITATION
          </span>
        </div>

        <h1 class="font-khmer-calligraphy text-2xl sm:text-4xl md:text-5xl font-bold bg-gradient-to-r from-secondary-fixed via-primary to-primary-container bg-clip-text text-transparent drop-shadow-[0_2px_12px_rgba(0,0,0,0.9)] mb-2">
          សូមគោរពអញ្ជើញចូលរួម
        </h1>
        <p class="font-body-sm text-xs sm:text-sm text-slate-200 max-w-md mb-7 leading-relaxed drop-shadow-[0_1px_4px_rgba(0,0,0,0.8)]">
          រាត្រីបើកសម្ពោធក្លឹបកម្សាន្តរាត្រី ភោជនីយដ្ឋានប្រណិត និងការប្រគំតន្ត្រីរាំរែក VIP
        </p>

        <button
          @click="openDoor"
          type="button"
          class="group relative inline-flex items-center justify-center gap-3 px-8 sm:px-10 py-3.5 sm:py-4 rounded-full bg-gradient-to-r from-secondary-fixed via-primary to-primary-container text-on-primary font-label-md text-sm sm:text-base font-bold shadow-[0_0_30px_rgba(242,202,80,0.6)] hover:shadow-[0_0_45px_rgba(242,202,80,0.9)] hover:scale-105 active:scale-95 transition-all duration-300 cursor-pointer"
        >
          <span class="material-symbols-outlined text-[20px] sm:text-[24px] group-hover:rotate-12 transition-transform">lock_open</span>
          <span>បើកទ្វារកិត្តិយស • ENTER SL CLUB</span>
        </button>

        <span class="text-[11px] sm:text-xs text-slate-300 mt-3 font-light tracking-wide drop-shadow">
          (ចុចដើម្បីបើកទ្វារចូលទស្សនាវេបសាយ)
        </span>
      </div>
    </div>

    <!-- ============================================================== -->
    <!-- FIXED HEADER (CLEAN, NO OVERLAPPING BUTTONS) -->
    <!-- ============================================================== -->
    <header class="fixed top-0 inset-x-0 z-50 bg-[#0c0c0e]/85 backdrop-blur-xl pt-safe shadow-[0_4px_25px_rgba(0,0,0,0.7)] border-b border-primary/20">
      <div class="max-w-6xl mx-auto h-16 sm:h-20 px-3 sm:px-6 flex items-center justify-between gap-3">

        <!-- Left: Brand Logo & Title -->
        <div class="flex items-center gap-2.5 sm:gap-3 min-w-0">
          <img
            alt="SL Club Logo"
            class="h-9 w-9 sm:h-11 sm:w-11 rounded-lg sm:rounded-xl object-contain bg-black/90 p-0.5 border border-primary/40 shadow-[0_0_14px_rgba(242,202,80,0.4)] shrink-0"
            src="/logo.jpg"
          />
          <div class="flex flex-col min-w-0">
            <div class="flex items-center gap-1.5 sm:gap-2">
              <span class="font-headline-sm text-base sm:text-xl text-primary font-bold tracking-wide leading-none truncate drop-shadow">SL CLUB</span>
              <span class="inline-flex items-center px-2 py-0.5 rounded-full bg-secondary-container/90 text-secondary text-[9px] sm:text-[10px] font-label-sm tracking-wider uppercase font-semibold border border-primary/20 shrink-0">
                VIP GALA
              </span>
            </div>
            <span class="font-label-sm text-[10px] sm:text-xs text-slate-300 truncate">
              សំបុត្រគោរពអញ្ជើញផ្លូវការ
            </span>
          </div>
        </div>

        <!-- Center: Page Switcher Tabs (Artboard 1 vs Page 2) -->
        <div class="flex items-center bg-black/75 p-1 rounded-full border border-primary/30 shadow-inner">
          <button
            @click="setPage(1)"
            type="button"
            :class="[
              'px-3 sm:px-4 py-1.5 rounded-full font-label-sm text-xs sm:text-sm font-semibold transition-all cursor-pointer flex items-center gap-1.5',
              currentPage === 1
                ? 'bg-gradient-to-r from-primary to-primary-container text-black font-bold shadow-[0_0_12px_rgba(242,202,80,0.4)]'
                : 'text-slate-300 hover:text-white'
            ]"
          >
            <span class="material-symbols-outlined text-[16px] sm:text-[18px]">mail</span>
            <span>ទំព័រ ១: លិខិតអញ្ជើញ</span>
          </button>

          <button
            @click="setPage(2)"
            type="button"
            :class="[
              'px-3 sm:px-4 py-1.5 rounded-full font-label-sm text-xs sm:text-sm font-semibold transition-all cursor-pointer flex items-center gap-1.5',
              currentPage === 2
                ? 'bg-gradient-to-r from-primary to-primary-container text-black font-bold shadow-[0_0_12px_rgba(242,202,80,0.4)]'
                : 'text-slate-300 hover:text-white'
            ]"
          >
            <span class="material-symbols-outlined text-[16px] sm:text-[18px]">local_fire_department</span>
            <span>ទំព័រ ២: ភាពកក្រើក</span>
          </button>
        </div>

        <!-- Right: Action Buttons (Music Button ALWAYS VISIBLE + Telegram + Menu ត្រេ ៣) -->
        <div class="flex items-center gap-1.5 sm:gap-2.5 shrink-0">
          <!-- Background Music Toggle Button (ALWAYS VISIBLE) -->
          <button
            @click="toggleMusic"
            :title="isMusicPlaying ? 'បិទភ្លេង (Pause Music)' : 'ចាក់ភ្លេង (Play Music)'"
            type="button"
            :class="[
              'h-9 sm:h-10 px-2.5 sm:px-3 rounded-full flex items-center gap-1.5 transition cursor-pointer border shadow-sm shrink-0',
              isMusicPlaying
                ? 'bg-primary/20 border-primary text-primary shadow-[0_0_12px_rgba(242,202,80,0.4)]'
                : 'bg-black/60 border-primary/30 text-slate-400 hover:text-primary hover:border-primary'
            ]"
          >
            <div v-if="isMusicPlaying" class="flex items-end gap-0.5 h-3.5 w-3.5">
              <span class="w-0.5 bg-primary rounded-full animate-pulse h-3.5"></span>
              <span class="w-0.5 bg-primary rounded-full animate-bounce h-2"></span>
              <span class="w-0.5 bg-primary rounded-full animate-pulse h-3"></span>
            </div>
            <span v-else class="material-symbols-outlined text-[19px] sm:text-[21px]">music_off</span>
            <span class="font-label-sm text-[11px] sm:text-xs font-bold hidden md:inline">
              {{ isMusicPlaying ? 'MUSIC ON' : 'MUSIC OFF' }}
            </span>
          </button>

          <!-- Telegram Channel Button -->
          <a
            href="https://t.me/SLCLUB33"
            target="_blank"
            rel="noopener noreferrer"
            title="Telegram Channel @SLCLUB33"
            class="w-9 h-9 sm:w-10 sm:h-10 rounded-full bg-[#229ED9]/20 hover:bg-[#229ED9]/35 text-[#229ED9] border border-[#229ED9]/40 flex items-center justify-center transition shadow-sm shrink-0"
          >
            <svg class="w-4 h-4 fill-current" viewBox="0 0 24 24">
              <path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.446 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.242-.213-.054-.333-.373-.121l-6.871 4.326-2.962-.924c-.643-.204-.657-.643.136-.953l11.57-4.458c.538-.196 1.006.128.832.943z"/>
            </svg>
          </a>

          <!-- HAMBURGER MENU BUTTON (ទ្រេ ៣ / ☰) -->
          <button
            @click="toggleMenu"
            :aria-expanded="isMenuOpen"
            title="មឺនុយ (ត្រេ ៣)"
            type="button"
            class="w-9 h-9 sm:w-10 sm:h-10 rounded-full bg-black/75 hover:bg-black/90 border border-primary/40 text-primary flex flex-col items-center justify-center gap-1 transition cursor-pointer shadow-md shrink-0 active:scale-95"
          >
            <span :class="['w-4 h-0.5 bg-primary rounded-full transition-all duration-300', isMenuOpen ? 'rotate-45 translate-y-1.5' : '']"></span>
            <span :class="['w-4 h-0.5 bg-primary rounded-full transition-all duration-300', isMenuOpen ? 'opacity-0 scale-0' : '']"></span>
            <span :class="['w-4 h-0.5 bg-primary rounded-full transition-all duration-300', isMenuOpen ? '-rotate-45 -translate-y-1.5' : '']"></span>
          </button>
        </div>

      </div>
    </header>

    <!-- ============================================================== -->
    <!-- VIP HAMBURGER MENU DRAWER (ទ្រេ ៣ DROPDOWN) -->
    <!-- ============================================================== -->
    <Transition
      enter-active-class="transition duration-200 ease-out"
      enter-from-class="opacity-0 -translate-y-4 scale-95"
      enter-to-class="opacity-100 translate-y-0 scale-100"
      leave-active-class="transition duration-150 ease-in"
      leave-from-class="opacity-100 translate-y-0 scale-100"
      leave-to-class="opacity-0 -translate-y-4 scale-95"
    >
      <div
        v-if="isMenuOpen"
        class="fixed inset-x-0 top-16 sm:top-20 z-50 p-3 sm:p-4 bg-black/60 backdrop-blur-md flex justify-end"
        @click.self="closeMenu"
      >
        <div class="w-full max-w-sm rounded-2xl bg-[#121214]/95 backdrop-blur-2xl border border-primary/40 shadow-[0_20px_60px_rgba(0,0,0,0.95)] p-5 space-y-4">
          <div class="flex items-center justify-between pb-3 border-b border-primary/20">
            <div class="flex items-center gap-2">
              <img src="/logo.jpg" alt="SL Club" class="w-7 h-7 rounded-lg object-contain border border-primary/30" />
              <div>
                <h4 class="font-headline-sm text-sm sm:text-base font-bold text-primary leading-tight">SL CLUB VIP MENU</h4>
                <p class="text-[10px] text-slate-400">មឺនុយសេវាកម្ម និងព័ត៌មាន</p>
              </div>
            </div>
            <button
              @click="closeMenu"
              class="w-7 h-7 rounded-full bg-white/10 flex items-center justify-center text-slate-300 hover:text-white cursor-pointer"
            >
              <span class="material-symbols-outlined text-[16px]">close</span>
            </button>
          </div>

          <!-- Quick Actions inside drawer -->
          <button
            @click="handleMenuAction('openModal')"
            class="w-full py-3 rounded-full bg-gradient-to-r from-secondary-fixed via-primary to-primary-container text-black font-bold text-xs sm:text-sm uppercase tracking-wider shadow-[0_0_15px_rgba(212,175,55,0.4)] flex items-center justify-center gap-2 cursor-pointer active:scale-95 transition"
          >
            <span class="material-symbols-outlined text-[18px]">confirmation_number</span>
            <span>កក់សំបុត្រ VIP (Reserve Pass)</span>
          </button>

          <div class="space-y-1.5 pt-1">
            <button
              @click="handleMenuAction('page1')"
              :class="['w-full px-3.5 py-2.5 rounded-xl flex items-center justify-between text-xs sm:text-sm font-medium transition cursor-pointer', currentPage === 1 ? 'bg-primary/20 text-primary border border-primary/30' : 'bg-black/50 text-slate-200 hover:bg-black/80']"
            >
              <div class="flex items-center gap-2.5">
                <span class="material-symbols-outlined text-[20px]">mail</span>
                <span>ទំព័រទី ១: លិខិតគោរពអញ្ជើញ (Artboard 1)</span>
              </div>
              <span class="material-symbols-outlined text-slate-400 text-[16px]">chevron_right</span>
            </button>

            <button
              @click="handleMenuAction('page2')"
              :class="['w-full px-3.5 py-2.5 rounded-xl flex items-center justify-between text-xs sm:text-sm font-medium transition cursor-pointer', currentPage === 2 ? 'bg-primary/20 text-primary border border-primary/30' : 'bg-black/50 text-slate-200 hover:bg-black/80']"
            >
              <div class="flex items-center gap-2.5">
                <span class="material-symbols-outlined text-[20px]">local_fire_department</span>
                <span>ទំព័រទី ២: ភាពកក្រើកនៃ SL CLUB</span>
              </div>
              <span class="material-symbols-outlined text-slate-400 text-[16px]">chevron_right</span>
            </button>
          </div>

          <div class="space-y-2 pt-2 border-t border-primary/20">
            <button
              @click="handleMenuAction('replayDoor')"
              class="w-full px-3.5 py-2 rounded-xl hover:bg-white/5 text-slate-200 flex items-center gap-2.5 text-xs sm:text-sm font-medium transition cursor-pointer text-left"
            >
              <span class="material-symbols-outlined text-primary text-[19px]">meeting_room</span>
              <span>បើកទ្វារកិត្តិយសឡើងវិញ (Replay Door)</span>
            </button>

            <button
              @click="toggleVideo"
              class="w-full px-3.5 py-2 rounded-xl hover:bg-white/5 text-slate-200 flex items-center justify-between text-xs sm:text-sm font-medium transition cursor-pointer"
            >
              <div class="flex items-center gap-2.5">
                <span class="material-symbols-outlined text-primary text-[19px]">
                  {{ isVideoPlaying ? 'motion_photos_on' : 'motion_photos_paused' }}
                </span>
                <span>{{ isVideoPlaying ? 'ផ្អាកវីដេអូ Background' : 'ចាក់វីដេអូ Background' }}</span>
              </div>
              <span class="text-[10px] text-primary font-bold px-2 py-0.5 rounded bg-primary/10">
                {{ isVideoPlaying ? 'ON' : 'PAUSED' }}
              </span>
            </button>

            <button
              @click="handleMenuAction('directions')"
              class="w-full px-3.5 py-2 rounded-xl hover:bg-white/5 text-slate-200 flex items-center gap-2.5 text-xs sm:text-sm font-medium transition cursor-pointer text-left"
            >
              <span class="material-symbols-outlined text-primary text-[19px]">location_on</span>
              <span>មើលទីតាំងលើ Google Maps</span>
            </button>
          </div>

          <div class="pt-3 border-t border-primary/20 grid grid-cols-2 gap-2">
            <a
              href="https://t.me/SLCLUB33"
              target="_blank"
              rel="noopener noreferrer"
              class="py-2.5 px-3 rounded-xl bg-[#229ED9]/15 hover:bg-[#229ED9]/25 text-[#229ED9] border border-[#229ED9]/30 text-xs font-semibold flex items-center justify-center gap-2 transition"
            >
              <svg class="w-4 h-4 fill-current shrink-0" viewBox="0 0 24 24">
                <path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.446 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.242-.213-.054-.333-.373-.121l-6.871 4.326-2.962-.924c-.643-.204-.657-.643.136-.953l11.57-4.458c.538-.196 1.006.128.832.943z"/>
              </svg>
              <span>Telegram</span>
            </a>

            <a
              href="tel:+18005557528"
              class="py-2.5 px-3 rounded-xl bg-primary/15 hover:bg-primary/25 text-primary border border-primary/30 text-xs font-semibold flex items-center justify-center gap-2 transition"
            >
              <span class="material-symbols-outlined text-[18px]">call</span>
              <span>ទូរស័ព្ទ VIP</span>
            </a>
          </div>

        </div>
      </div>
    </Transition>

    <!-- ============================================================== -->
    <!-- MAIN CONTENT: ARTBOARD 1 VIP INVITATION SHOWCASE -->
    <!-- ============================================================== -->
    <main class="relative z-10 flex-1 flex flex-col justify-center items-center px-3 sm:px-6 pt-20 sm:pt-24 pb-8 min-h-screen">
      <div class="w-full max-w-xl mx-auto flex flex-col items-center">

        <!-- ========================================================== -->
        <!-- CARD CONTAINER: CHROME METALLIC FRAME WITH CORNER RIVETS -->
        <!-- ========================================================== -->
        <div class="chrome-card-frame rounded-2xl sm:rounded-3xl w-full p-4 sm:p-7 md:p-8 relative backdrop-blur-xl flex flex-col justify-between my-auto">

          <!-- 4 Chrome Screws in the 4 corners (Artboard 1 details) -->
          <div class="absolute top-3 left-3 chrome-screw"></div>
          <div class="absolute top-3 right-3 chrome-screw"></div>
          <div class="absolute bottom-3 left-3 chrome-screw"></div>
          <div class="absolute bottom-3 right-3 chrome-screw"></div>

          <!-- Page 1 & Page 2 Transition Wrapper -->
          <Transition
            mode="out-in"
            enter-active-class="transition duration-300 ease-out"
            enter-from-class="opacity-0 scale-98 translate-y-2"
            enter-to-class="opacity-100 scale-100 translate-y-0"
            leave-active-class="transition duration-200 ease-in"
            leave-from-class="opacity-100 scale-100 translate-y-0"
            leave-to-class="opacity-0 scale-98 -translate-y-2"
          >

            <!-- ====================================================== -->
            <!-- PAGE 1: ARTBOARD 1 OFFICIAL INVITATION -->
            <!-- ====================================================== -->
            <div v-if="currentPage === 1" key="page1" class="flex flex-col items-center text-center space-y-4 sm:space-y-5 w-full">

              <!-- Top SL CLUB Monogram Logo -->
              <div class="flex flex-col items-center">
                <div class="relative group">
                  <div class="absolute -inset-2 bg-gradient-to-r from-primary/30 to-white/30 rounded-full blur-xl opacity-75 group-hover:opacity-100 transition duration-500"></div>
                  <div class="relative w-20 h-20 sm:w-24 sm:h-24 rounded-2xl bg-black/80 border border-white/40 p-2.5 shadow-[0_0_25px_rgba(255,255,255,0.2)] flex items-center justify-center">
                    <img
                      src="/logo.jpg"
                      alt="SL Club Monogram"
                      class="w-full h-full object-contain filter drop-shadow-[0_0_8px_rgba(255,255,255,0.7)]"
                    />
                  </div>
                </div>
                <span class="font-headline-sm text-lg sm:text-xl font-black text-white tracking-[0.25em] mt-2 drop-shadow-[0_2px_10px_rgba(0,0,0,0.9)]">
                  CLUB
                </span>
              </div>

              <!-- Main Heading: សំបុត្រគោរពអញ្ជើញ & INVITATION -->
              <div class="space-y-1">
                <h2 class="font-khmer-calligraphy text-2xl sm:text-4xl md:text-[42px] font-bold text-white drop-shadow-[0_2px_14px_rgba(0,0,0,0.95)] leading-tight">
                  សំបុត្រគោរពអញ្ជើញ
                </h2>
                <div class="flex items-center justify-center gap-3">
                  <span class="w-8 sm:w-12 h-[1px] bg-gradient-to-r from-transparent to-white/70"></span>
                  <span class="font-headline-sm text-xs sm:text-sm font-semibold tracking-[0.35em] text-slate-200 uppercase drop-shadow">
                    INVITATION
                  </span>
                  <span class="w-8 sm:w-12 h-[1px] bg-gradient-to-l from-transparent to-white/70"></span>
                </div>
              </div>

              <!-- USER REQUESTED HIGHLIGHTED VIP BOX: សំបុត្រគោរពអញ្ជើញភ្ញៀវកិត្តិយស -->
              <div class="w-full max-w-md mx-auto my-1 p-3.5 sm:p-5 rounded-xl bg-gradient-to-r from-black/85 via-[#18181b]/90 to-black/85 border-2 border-primary/60 shadow-[0_0_25px_rgba(242,202,80,0.35)] backdrop-blur-md relative overflow-hidden group">
                <div class="absolute inset-0 bg-gradient-to-r from-transparent via-primary/10 to-transparent -translate-x-full group-hover:translate-x-full transition-transform duration-1000 ease-in-out"></div>
                <div class="relative z-10 flex flex-col items-center">
                  <span class="inline-flex items-center gap-1.5 text-[11px] sm:text-xs text-primary font-bold uppercase tracking-wider mb-1">
                    <span class="material-symbols-outlined text-[15px]">verified</span>
                    <span>VIP GUEST PASS</span>
                  </span>
                  <h3 class="font-khmer-calligraphy text-xl sm:text-2xl md:text-3xl font-extrabold text-primary drop-shadow-[0_2px_10px_rgba(0,0,0,0.95)]">
                    សំបុត្រគោរពអញ្ជើញភ្ញៀវកិត្តិយស
                  </h3>
                  <p class="font-body-sm text-[11px] sm:text-xs text-slate-200 mt-1">
                    រាត្រីបើកសម្ពោធមហោឡារិក • ថ្ងៃសុក្រ ទី ០៩ ខែតុលា ឆ្នាំ ២០២៦
                  </p>
                </div>
              </div>

              <!-- Live Countdown Display -->
              <div class="w-full max-w-md mx-auto px-3.5 py-2 rounded-xl bg-black/75 border border-primary/30 flex items-center justify-between text-xs sm:text-sm">
                <div class="flex items-center gap-1.5 text-slate-300">
                  <span class="material-symbols-outlined text-primary text-[18px]">timer</span>
                  <span class="font-medium">រាប់ថយក្រោយ៖</span>
                </div>
                <div class="text-primary font-bold tracking-wide">
                  {{ countdownText }}
                </div>
              </div>

              <!-- Venue Location Info matching Artboard 1 Footer -->
              <a
                href="https://maps.app.goo.gl/hSJKgn4A28KUYYnm8"
                target="_blank"
                rel="noopener noreferrer"
                class="w-full max-w-md mx-auto p-2.5 rounded-xl bg-black/60 hover:bg-black/85 border border-white/20 hover:border-primary/50 transition flex items-center justify-between gap-3 text-left group"
              >
                <div class="flex items-center gap-2.5 min-w-0">
                  <img src="/logo.jpg" alt="SL Club" class="w-6 h-6 rounded object-contain shrink-0 border border-primary/30" />
                  <div class="min-w-0">
                    <div class="flex items-center gap-1 text-[11px] text-primary font-bold">
                      <span class="material-symbols-outlined text-[14px]">location_on</span>
                      <span>LOCATION:</span>
                    </div>
                    <p class="font-label-sm text-[11px] sm:text-xs text-slate-200 font-semibold truncate">
                      BATTAMBANG, STREET 203, 100 METER FROM ABA BANK
                    </p>
                  </div>
                </div>
                <span class="material-symbols-outlined text-primary text-[18px] group-hover:translate-x-1 transition-transform shrink-0">open_in_new</span>
              </a>

              <!-- Card Action Cluster -->
              <div class="w-full max-w-md flex flex-col sm:flex-row items-center gap-2.5 pt-1">
                <button
                  @click="openReservationModal()"
                  type="button"
                  class="w-full sm:flex-1 h-11 px-5 rounded-full bg-gradient-to-r from-secondary-fixed via-primary to-primary-container text-black font-label-md text-xs sm:text-sm font-bold shadow-[0_0_20px_rgba(242,202,80,0.4)] hover:shadow-[0_0_30px_rgba(242,202,80,0.7)] hover:scale-102 active:scale-98 transition cursor-pointer flex items-center justify-center gap-2"
                >
                  <span class="material-symbols-outlined text-[18px]">how_to_reg</span>
                  <span>កក់កន្លែង VIP ទទួលការអញ្ជើញ</span>
                </button>

                <button
                  @click="setPage(2)"
                  type="button"
                  class="w-full sm:w-auto h-11 px-4 rounded-full bg-white/10 hover:bg-white/20 border border-white/30 text-white font-label-md text-xs sm:text-sm font-semibold transition cursor-pointer flex items-center justify-center gap-1.5"
                >
                  <span>ទំព័រ ២</span>
                  <span class="material-symbols-outlined text-[18px]">arrow_forward</span>
                </button>
              </div>

            </div>

            <!-- ====================================================== -->
            <!-- PAGE 2: ភាពកក្រើក និងសប្បាយរីករាយជាមួយ SL CLUB -->
            <!-- ====================================================== -->
            <div v-else key="page2" class="flex flex-col items-center text-center space-y-4 sm:space-y-5 w-full">

              <!-- Top Badge -->
              <div class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full bg-primary/20 border border-primary/40 text-primary text-xs font-bold tracking-wide">
                <span class="material-symbols-outlined text-[16px]">celebration</span>
                <span>GRAND OPENING EXPERIENCE</span>
              </div>

              <!-- USER REQUESTED TEXT: ពួកយើងនឹងនាំមកនូវភាពកក្រើក និងសប្បាយរីករាយជាមួយ SL CLUB -->
              <div class="space-y-2">
                <h2 class="font-khmer-calligraphy text-2xl sm:text-3xl md:text-4xl font-bold bg-gradient-to-r from-secondary-fixed via-primary to-white bg-clip-text text-transparent drop-shadow-[0_2px_12px_rgba(0,0,0,0.95)] leading-snug max-w-md mx-auto">
                  ពួកយើងនឹងនាំមកនូវភាពកក្រើក និងសប្បាយរីករាយជាមួយ SL CLUB
                </h2>
                <p class="font-body-sm text-xs sm:text-sm text-slate-200 max-w-sm mx-auto">
                  រាត្រីតន្ត្រីផ្សាយផ្ទាល់ ភោជនីយដ្ឋានលំដាប់ផ្កាយ ៥ និងការរាំរែកដែលពោរពេញដោយថាមពល
                </p>
              </div>

              <!-- Feature Highlights Grid (2x2) -->
              <div class="w-full max-w-md grid grid-cols-2 gap-2.5 text-left">
                <!-- Highlight 1: Live Concert -->
                <div class="p-3 rounded-xl bg-black/65 border border-primary/30 backdrop-blur-md flex flex-col gap-1.5">
                  <div class="w-8 h-8 rounded-lg bg-primary/20 text-primary flex items-center justify-center">
                    <span class="material-symbols-outlined text-[20px]">mic</span>
                  </div>
                  <h4 class="font-label-md text-xs sm:text-sm font-bold text-white">តន្ត្រីផ្សាយផ្ទាល់ &amp; DJs</h4>
                  <p class="text-[11px] text-slate-300 leading-tight">កំពូលតារាចម្រៀងល្បីៗ និង DJ ចាក់ភ្លេងរាំកម្សាន្ត</p>
                </div>

                <!-- Highlight 2: 5-Star Fine Dining -->
                <div class="p-3 rounded-xl bg-black/65 border border-primary/30 backdrop-blur-md flex flex-col gap-1.5">
                  <div class="w-8 h-8 rounded-lg bg-primary/20 text-primary flex items-center justify-center">
                    <span class="material-symbols-outlined text-[20px]">restaurant</span>
                  </div>
                  <h4 class="font-label-md text-xs sm:text-sm font-bold text-white">ភោជនីយដ្ឋានផ្កាយ ៥</h4>
                  <p class="text-[11px] text-slate-300 leading-tight">ម្ហូប Fine Dining ឆ្ងាញ់ពិសាចម្អិនដោយចុងភៅជំនាញ</p>
                </div>

                <!-- Highlight 3: Dance Floor & Laser 3D -->
                <div class="p-3 rounded-xl bg-black/65 border border-primary/30 backdrop-blur-md flex flex-col gap-1.5">
                  <div class="w-8 h-8 rounded-lg bg-primary/20 text-primary flex items-center justify-center">
                    <span class="material-symbols-outlined text-[20px]">speaker</span>
                  </div>
                  <h4 class="font-label-md text-xs sm:text-sm font-bold text-white">Dance Floor &amp; Laser</h4>
                  <p class="text-[11px] text-slate-300 leading-tight">សំឡេងកម្រិតពិភពលោក ភ្លើង 3D Laser ទំនើប</p>
                </div>

                <!-- Highlight 4: VIP Skybox Butler -->
                <div class="p-3 rounded-xl bg-black/65 border border-primary/30 backdrop-blur-md flex flex-col gap-1.5">
                  <div class="w-8 h-8 rounded-lg bg-primary/20 text-primary flex items-center justify-center">
                    <span class="material-symbols-outlined text-[20px]">diamond</span>
                  </div>
                  <h4 class="font-label-md text-xs sm:text-sm font-bold text-white">VIP Skybox &amp; Butler</h4>
                  <p class="text-[11px] text-slate-300 leading-tight">បន្ទប់ឯកជន ស្រាសំប៉ាញ និងអ្នកបម្រើផ្ទាល់</p>
                </div>
              </div>

              <!-- VIP Pricing Cards Overview -->
              <div class="w-full max-w-md bg-black/70 p-3 rounded-xl border border-primary/30 flex items-center justify-between text-left">
                <div>
                  <span class="text-[10px] text-primary uppercase font-bold tracking-wider">កញ្ចប់សំបុត្រ VIP</span>
                  <div class="flex items-center gap-2 mt-0.5">
                    <span class="text-sm font-extrabold text-white">Club Pass ($150)</span>
                    <span class="text-xs text-slate-400">|</span>
                    <span class="text-sm font-extrabold text-primary">Skybox ($450)</span>
                  </div>
                </div>
                <button
                  @click="openReservationModal()"
                  class="px-3.5 py-1.5 rounded-full bg-primary text-black font-bold text-xs hover:bg-primary/90 transition"
                >
                  កក់កន្លែង
                </button>
              </div>

              <!-- Page 2 Navigation Action Buttons -->
              <div class="w-full max-w-md flex flex-col sm:flex-row items-center gap-2.5 pt-1">
                <button
                  @click="setPage(1)"
                  type="button"
                  class="w-full sm:w-auto h-11 px-4 rounded-full bg-white/10 hover:bg-white/20 border border-white/30 text-white font-label-md text-xs sm:text-sm font-semibold transition cursor-pointer flex items-center justify-center gap-1.5"
                >
                  <span class="material-symbols-outlined text-[18px]">arrow_back</span>
                  <span>ទំព័រ ១: សំបុត្រ</span>
                </button>

                <button
                  @click="openReservationModal()"
                  type="button"
                  class="w-full sm:flex-1 h-11 px-5 rounded-full bg-gradient-to-r from-secondary-fixed via-primary to-primary-container text-black font-label-md text-xs sm:text-sm font-bold shadow-[0_0_20px_rgba(242,202,80,0.4)] hover:shadow-[0_0_30px_rgba(242,202,80,0.7)] hover:scale-102 active:scale-98 transition cursor-pointer flex items-center justify-center gap-2"
                >
                  <span class="material-symbols-outlined text-[18px]">table_bar</span>
                  <span>កក់តុ &amp; សំបុត្រ VIP ឥឡូវនេះ</span>
                </button>
              </div>

            </div>

          </Transition>

          <!-- Card Pagination Controls (Dots & Arrows) -->
          <div class="flex items-center justify-between pt-4 mt-2 border-t border-white/10 w-full text-xs text-slate-300">
            <button
              @click="prevPage"
              type="button"
              class="flex items-center gap-1 text-slate-300 hover:text-primary transition cursor-pointer font-medium"
            >
              <span class="material-symbols-outlined text-[16px]">chevron_left</span>
              <span>{{ currentPage === 1 ? 'ទៅទំព័រ ២' : 'ទៅទំព័រ ១' }}</span>
            </button>

            <!-- Page Indicators -->
            <div class="flex items-center gap-2">
              <button
                @click="setPage(1)"
                :class="['w-2.5 h-2.5 rounded-full transition-all cursor-pointer', currentPage === 1 ? 'bg-primary w-6' : 'bg-white/30']"
                title="ទំព័រ ១"
              ></button>
              <button
                @click="setPage(2)"
                :class="['w-2.5 h-2.5 rounded-full transition-all cursor-pointer', currentPage === 2 ? 'bg-primary w-6' : 'bg-white/30']"
                title="ទំព័រ ២"
              ></button>
            </div>

            <button
              @click="nextPage"
              type="button"
              class="flex items-center gap-1 text-slate-300 hover:text-primary transition cursor-pointer font-medium"
            >
              <span>{{ currentPage === 1 ? 'ទៅទំព័រ ២' : 'ទៅទំព័រ ១' }}</span>
              <span class="material-symbols-outlined text-[16px]">chevron_right</span>
            </button>
          </div>

        </div>

      </div>
    </main>

    <!-- ============================================================== -->
    <!-- INTERACTIVE RESERVATION CONFIRMATION DIALOG -->
    <!-- ============================================================== -->
    <Transition
      enter-active-class="transition duration-200 ease-out"
      enter-from-class="opacity-0 scale-95"
      enter-to-class="opacity-100 scale-100"
      leave-active-class="transition duration-150 ease-in"
      leave-from-class="opacity-100 scale-100"
      leave-to-class="opacity-0 scale-95"
    >
      <div
        v-if="isModalOpen"
        class="fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6 bg-black/85 backdrop-blur-md"
        @click.self="closeReservationModal"
      >
        <div class="bg-[#141416]/95 backdrop-blur-2xl p-6 sm:p-8 rounded-2xl shadow-[0_25px_70px_rgba(0,0,0,0.95)] flex flex-col gap-4 text-on-surface border border-primary/40 w-full max-w-md">

          <div class="flex items-center justify-between">
            <div class="flex items-center gap-3">
              <img src="/logo.jpg" alt="SL Club Logo" class="w-8 h-8 rounded-lg object-contain border border-primary/40 bg-black/80 shadow-md" />
              <span class="font-headline-sm text-lg sm:text-xl font-bold text-white">ការចុះឈ្មោះ VIP</span>
            </div>
            <button
              @click="closeReservationModal"
              type="button"
              class="w-8 h-8 rounded-full bg-black/60 hover:bg-black/90 flex items-center justify-center text-slate-300 hover:text-white cursor-pointer border border-primary/20"
            >
              <span class="material-symbols-outlined text-[18px]">close</span>
            </button>
          </div>

          <p class="font-body-sm text-xs sm:text-sm text-slate-200 leading-relaxed">
            លោកអ្នកកំពុងកក់កន្លែងសម្រាប់ <strong class="text-primary">{{ selectedTier.fullName }} ({{ selectedTier.cost }})</strong> នាថ្ងៃសុក្រ ទី ០៩ ខែតុលា ឆ្នាំ ២០២៦។
          </p>

          <div class="flex flex-col gap-3 pt-2">
            <div>
              <label class="block font-label-sm text-xs text-slate-300 mb-1 font-semibold">គោត្តនាម និងនាម</label>
              <input
                v-model="guestName"
                class="w-full h-11 px-3.5 rounded-lg bg-black/60 text-slate-100 font-body-sm text-sm placeholder:text-slate-500 focus:outline-none focus:ring-1 focus:ring-primary border border-primary/30"
                placeholder="ឯកឧត្តម / លោកជំទាវ / លោក / លោកស្រី"
                type="text"
              />
            </div>

            <div>
              <label class="block font-label-sm text-xs text-slate-300 mb-1 font-semibold">លេខទូរស័ព្ទ / WhatsApp ឬ Telegram</label>
              <input
                v-model="guestPhone"
                class="w-full h-11 px-3.5 rounded-lg bg-black/60 text-slate-100 font-body-sm text-sm placeholder:text-slate-500 focus:outline-none focus:ring-1 focus:ring-primary border border-primary/30"
                placeholder="+855 00 000 000"
                type="tel"
              />
            </div>

            <div class="pt-1">
              <a
                href="https://t.me/SLCLUB33"
                target="_blank"
                rel="noopener noreferrer"
                class="w-full py-2.5 px-3 rounded-lg bg-[#229ED9]/15 hover:bg-[#229ED9]/25 text-[#229ED9] border border-[#229ED9]/30 text-xs font-semibold flex items-center justify-center gap-2 transition"
              >
                <svg class="w-4 h-4 fill-current shrink-0" viewBox="0 0 24 24">
                  <path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.446 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.242-.213-.054-.333-.373-.121l-6.871 4.326-2.962-.924c-.643-.204-.657-.643.136-.953l11.57-4.458c.538-.196 1.006.128.832.943z"/>
                </svg>
                <span>ទាក់ទង / តាមដាន Telegram: @SLCLUB33</span>
              </a>
            </div>
          </div>

          <div class="flex items-center justify-end gap-3 pt-4 border-t border-primary/20 mt-2">
            <button
              @click="closeReservationModal"
              type="button"
              class="h-10 px-5 rounded-full font-label-md text-xs sm:text-sm text-slate-300 hover:text-white cursor-pointer"
            >
              បោះបង់
            </button>
            <button
              @click="confirmReservation"
              type="button"
              class="h-10 px-6 rounded-full bg-primary text-black font-label-md text-xs sm:text-sm font-bold shadow-lg hover:bg-primary/90 transition cursor-pointer"
            >
              បញ្ជាក់ការកក់
            </button>
          </div>

        </div>
      </div>
    </Transition>

  </div>
</template>
