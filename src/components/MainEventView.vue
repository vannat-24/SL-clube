<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['replay-door'])

// --- Grand Video Entrance State ---
const showIntroScreen = ref(false)
const isPlayingIntroVideo = ref(false)
const introVideoRef = ref(null)

function startIntroVideo() {
  isPlayingIntroVideo.value = true
  if (introVideoRef.value) {
    introVideoRef.value.currentTime = 0
    introVideoRef.value.play().catch(err => {
      console.warn("Autoplay blocked, user interaction required:", err)
    })
  }
}

function handleVideoEnded() {
  finishIntro()
}

function finishIntro() {
  showIntroScreen.value = false
  isPlayingIntroVideo.value = false
}

function replayDoor() {
  showIntroScreen.value = true
  startIntroVideo()
}

// --- Hamburger Menu State (ទ្រេ ៣) ---
const isMenuOpen = ref(false)

function toggleMenu() {
  isMenuOpen.value = !isMenuOpen.value
}

function closeMenu() {
  isMenuOpen.value = false
}

function handleMenuNav(sectionId) {
  closeMenu()
  scrollToSection(sectionId)
}

function handleMenuAction(action) {
  closeMenu()
  if (action === 'replayDoor') replayDoor()
  if (action === 'openModal') openReservationModal()
  if (action === 'directions') openDirections()
}

// --- Background Video State & Controls ---
const videoRef = ref(null)
const isVideoPlaying = ref(true)

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



// --- VIP Tiers ---
const tiers = [
  {
    id: 'general',
    title: 'កាតចូលរួមទូទៅ VIP Club Pass',
    fullName: 'កាតចូលរួមទូទៅ VIP Club Pass',
    cost: '$150',
    level: 'កម្រិត ១',
    description: 'ចូលទស្សនាការប្រគំតន្ត្រី + តំបន់រាំកម្សាន្ត Dance Floor & Welcome Drink',
    features: [
      'ចូលទស្សនាការប្រគំតន្ត្រី VIP Live Concert',
      'តំបន់រាំកម្សាន្ត Dance Floor ក្បែរឆាក',
      'ភេសជ្ជៈទទួលស្វាគមន៍ Welcome Drink ១ កែវ',
      'សេវាចតរថយន្តទូទៅ'
    ],
    featured: false
  },
  {
    id: 'founder',
    title: 'តុពិសេស Founder Skybox VIP',
    fullName: 'តុពិសេស Founder Skybox VIP',
    cost: '$450',
    level: 'កម្រិតពិសេសបំផុត',
    description: 'សេវាស្រាសំប៉ាញផ្ទាល់, ម្ហូប Fine Dining, ទិដ្ឋភាពល្អបំផុត & បារឯកជន',
    features: [
      'តុពិសេស VIP Skybox ទិដ្ឋភាពល្អផ្តាច់គេ',
      'ស្រាសំប៉ាញ Dom Pérignon & Reserve Mixology',
      'ម្ហូបបែប Fine Dining ពិសេសចម្អិនផ្ទាល់',
      'អ្នកបម្រើផ្ទាល់ (Dedicated VIP Butler)',
      'សេវាចតរថយន្ត VIP Porte-cochère / Helipad'
    ],
    featured: true
  }
]

const selectedTier = ref(tiers[1]) // Default to Founder Skybox VIP
const isModalOpen = ref(false)
const reservationSuccess = ref(false)

const guestName = ref('')
const guestPhone = ref('')

const activeTab = ref('grand-opening')

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

function selectTier(tier) {
  selectedTier.value = tier
}

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
    alert('សូមអរគុណ! ក្រុមការងារ Concierge នៃ SL Club នឹងទាក់ទងមកលោកអ្នកក្នុងរយៈពេល ២ ម៉ោង ដើម្បីរៀបចំការទទួលស្វាគមន៍។')
    guestName.value = ''
    guestPhone.value = ''
  }, 400)
}

function openDirections() {
  window.open('https://maps.app.goo.gl/hSJKgn4A28KUYYnm8', '_blank')
}

function scrollToSection(id) {
  const el = document.getElementById(id)
  if (el) {
    el.scrollIntoView({ behavior: 'smooth' })
  }
}
</script>

<template>
  <div class="min-h-screen flex flex-col bg-[#0e0e0e] font-body-md text-on-surface antialiased selection:bg-primary-container selection:text-on-primary-container relative">

    <!-- CONTINUOUS LOOPING VIP BACKGROUND VIDEO -->
    <div class="fixed inset-0 z-0 overflow-hidden pointer-events-none">
      <video
        ref="videoRef"
        autoplay
        loop
        muted
        playsinline
        preload="auto"
        class="w-full h-full object-cover object-center filter brightness-[0.92] contrast-[1.08] transform scale-105"
        src="/bg-video.mp4"
      ></video>
      <div class="absolute inset-0 bg-gradient-to-b from-black/60 via-black/38 to-black/70"></div>
      <div class="absolute inset-0 bg-[radial-gradient(ellipse_at_top,_rgba(242,202,80,0.1)_0%,_transparent_75%)]"></div>
    </div>

    <!-- ================= GRAND LUXURY VIP VIDEO INTRO ENTRANCE ================= -->
    <div
      v-if="showIntroScreen"
      class="fixed inset-0 z-[100] flex items-center justify-center overflow-hidden bg-black select-none"
    >
      <!-- Mode A: Video Playing Full-Screen -->
      <div v-show="isPlayingIntroVideo" class="absolute inset-0 z-20 bg-black">
        <video
          ref="introVideoRef"
          playsinline
          preload="auto"
          class="w-full h-full object-cover"
          @ended="handleVideoEnded"
        >
          <source src="/intro-video.mp4" type="video/mp4" />
        </video>

        <!-- Skip Button -->
        <button
          @click="finishIntro"
          type="button"
          class="absolute top-6 right-6 z-30 flex items-center gap-1.5 px-4 py-2 rounded-full bg-black/60 hover:bg-black/80 backdrop-blur-md border border-white/20 text-white text-xs font-semibold shadow-lg hover:scale-105 active:scale-95 transition-all cursor-pointer"
        >
          <span>រំលង (Skip)</span>
          <span class="material-symbols-outlined text-sm">skip_next</span>
        </button>
      </div>

      <!-- Mode B: Initial Welcoming Screen Before Click -->
      <div
        v-show="!isPlayingIntroVideo"
        class="relative z-10 flex flex-col items-center text-center px-4 sm:px-6 max-w-lg mx-auto"
      >
        <!-- Atmospheric Background Glow -->
        <div class="absolute -inset-20 bg-radial-[circle_at_50%_50%] from-primary/20 via-primary/5 to-transparent blur-3xl rounded-full"></div>

        <!-- Crest SL Logo -->
        <div class="relative mb-6">
          <div class="w-28 h-28 sm:w-36 sm:h-36 md:w-40 md:h-40 rounded-full p-1 bg-gradient-to-b from-secondary-fixed via-primary to-primary-container shadow-[0_0_50px_rgba(242,202,80,0.6)]">
            <div class="w-full h-full rounded-full bg-black/95 p-3 flex items-center justify-center overflow-hidden border border-primary/40">
              <img
                src="/logo.jpg"
                alt="SL Club Golden Monogram"
                class="w-full h-full object-contain filter drop-shadow-[0_0_15px_rgba(242,202,80,0.8)]"
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
        <p class="font-body-sm text-xs sm:text-sm md:text-base text-slate-200 max-w-md mb-8 leading-relaxed drop-shadow-[0_1px_4px_rgba(0,0,0,0.8)]">
          រាត្រីបើកសម្ពោធក្លឹបកម្សាន្តរាត្រី ភោជនីយដ្ឋានប្រណិត និងការប្រគំតន្ត្រីរាំរែក VIP
        </p>

        <!-- Click to Play Video & Enter Website -->
        <button
          @click="startIntroVideo"
          type="button"
          class="group relative inline-flex items-center justify-center gap-3 px-8 sm:px-10 py-3.5 sm:py-4 rounded-full bg-gradient-to-r from-secondary-fixed via-primary to-primary-container text-on-primary font-label-md text-sm sm:text-base font-bold shadow-[0_0_35px_rgba(242,202,80,0.7)] hover:shadow-[0_0_50px_rgba(242,202,80,0.95)] hover:scale-105 active:scale-95 transition-all duration-300 cursor-pointer"
        >
          <span class="material-symbols-outlined text-[20px] sm:text-[24px] group-hover:scale-110 transition-transform">play_circle</span>
          <span>ចុចបើកទស្សនា • ENTER SL CLUB</span>
        </button>

        <span class="text-[11px] sm:text-xs text-slate-400 mt-3 font-light tracking-wide">
          (ចុចដើម្បីទស្សនាវីដេអូបើកទ្វារចូលវេបសាយ)
        </span>
      </div>
    </div>

    <!-- FIXED HEADER -->
    <header class="fixed top-0 inset-x-0 z-50 bg-[#0e0e0e]/80 backdrop-blur-xl pt-safe shadow-[0_4px_25px_rgba(0,0,0,0.6)] border-b border-primary/20">
      <div class="max-w-7xl mx-auto h-16 sm:h-20 px-4 sm:px-6 lg:px-8 flex items-center justify-between gap-4">

        <!-- Left: Brand Logo & Title -->
        <div class="flex items-center gap-2.5 sm:gap-3 min-w-0">
          <img
            alt="SL Club Logo"
            class="h-9 w-9 sm:h-11 sm:w-11 rounded-lg sm:rounded-xl object-contain bg-black/90 p-0.5 border border-primary/40 shadow-[0_0_14px_rgba(242,202,80,0.4)] shrink-0"
            src="/logo.jpg"
          />
          <div class="flex flex-col min-w-0">
            <div class="flex items-center gap-1.5 sm:gap-2">
              <span class="font-headline-sm text-sm sm:text-xl text-primary font-bold tracking-wide leading-none truncate drop-shadow">SL CLUB</span>
              <span class="inline-flex items-center px-2 py-0.5 rounded-full bg-secondary-container/90 text-secondary text-[9px] sm:text-[10px] font-label-sm tracking-wider uppercase font-semibold border border-primary/20 shrink-0 whitespace-nowrap">
                សំបុត្រ VIP
              </span>
            </div>
            <span class="font-label-sm text-[10px] sm:text-xs text-slate-300 tracking-normal truncate drop-shadow-sm">
              សំបុត្រគោរពអញ្ជើញផ្លូវការ
            </span>
          </div>
        </div>

        <!-- Center: Desktop Navigation Bar -->
        <nav class="hidden md:flex items-center gap-8 text-sm font-semibold">
          <button @click="scrollToSection('events')" type="button" class="text-slate-300 hover:text-primary transition flex items-center gap-1.5 cursor-pointer drop-shadow-sm">
            <span class="material-symbols-outlined text-[18px]">celebration</span>
            <span>កម្មវិធី</span>
          </button>
          <button @click="scrollToSection('services')" type="button" class="text-slate-300 hover:text-primary transition flex items-center gap-1.5 cursor-pointer drop-shadow-sm">
            <span class="material-symbols-outlined text-[18px]">speaker</span>
            <span>សេវាកម្ម</span>
          </button>
          <button @click="scrollToSection('schedule')" type="button" class="text-slate-300 hover:text-primary transition flex items-center gap-1.5 cursor-pointer drop-shadow-sm">
            <span class="material-symbols-outlined text-[18px]">calendar_month</span>
            <span>កាលវិភាគ</span>
          </button>
          <button @click="scrollToSection('venue')" type="button" class="text-slate-300 hover:text-primary transition flex items-center gap-1.5 cursor-pointer drop-shadow-sm">
            <span class="material-symbols-outlined text-[18px]">location_on</span>
            <span>ទីតាំង</span>
          </button>
          <button @click="scrollToSection('tickets')" type="button" class="text-slate-300 hover:text-primary transition flex items-center gap-1.5 cursor-pointer drop-shadow-sm">
            <span class="material-symbols-outlined text-[18px]">table_bar</span>
            <span>កញ្ចប់ VIP</span>
          </button>
        </nav>

        <!-- Right: Action Buttons -->
        <div class="flex items-center gap-2 sm:gap-3 shrink-0">
          <button
            @click="openReservationModal()"
            type="button"
            class="hidden sm:inline-flex h-9 sm:h-10 px-4 sm:px-5 rounded-full bg-gradient-to-r from-secondary-fixed to-primary-container text-on-primary font-label-md text-xs sm:text-sm uppercase tracking-wider items-center justify-center font-bold shadow-[0_0_15px_rgba(212,175,55,0.35)] hover:shadow-[0_0_22px_rgba(212,175,55,0.6)] transition-all cursor-pointer shrink-0"
          >
            <span>កក់កន្លែង</span>
          </button>

          <button
            @click="toggleMenu"
            :aria-expanded="isMenuOpen"
            title="មឺនុយ (ត្រេ ៣)"
            type="button"
            class="w-9 h-9 sm:w-10 sm:h-10 rounded-full bg-black/75 hover:bg-black/90 border border-primary/40 text-primary flex flex-col items-center justify-center gap-1 transition cursor-pointer shadow-md shrink-0 active:scale-95"
          >
            <span :class="['w-4.5 h-0.5 bg-primary rounded-full transition-all duration-300', isMenuOpen ? 'rotate-45 translate-y-1.5' : '']"></span>
            <span :class="['w-4.5 h-0.5 bg-primary rounded-full transition-all duration-300', isMenuOpen ? 'opacity-0 scale-0' : '']"></span>
            <span :class="['w-4.5 h-0.5 bg-primary rounded-full transition-all duration-300', isMenuOpen ? '-rotate-45 -translate-y-1.5' : '']"></span>
          </button>
        </div>

      </div>
    </header>

    <!-- VIP HAMBURGER MENU DRAWER -->
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
        <div class="w-full max-w-sm rounded-2xl bg-[#121212]/95 backdrop-blur-2xl border border-primary/40 shadow-[0_20px_60px_rgba(0,0,0,0.95)] p-5 space-y-4">
          <div class="flex items-center justify-between pb-3 border-b border-outline-variant/30">
            <div class="flex items-center gap-2">
              <img src="/logo.jpg" alt="SL Club" class="w-7 h-7 rounded-lg object-contain border border-primary/30" />
              <div>
                <h4 class="font-headline-sm text-sm sm:text-base font-bold text-primary leading-tight">SL CLUB VIP MENU</h4>
                <p class="text-[10px] text-slate-400">មឺនុយសេវាកម្ម និងព័ត៌មាន</p>
              </div>
            </div>
            <button
              @click="closeMenu"
              class="w-7 h-7 rounded-full bg-surface-container-high flex items-center justify-center text-slate-300 hover:text-white cursor-pointer"
            >
              <span class="material-symbols-outlined text-[16px]">close</span>
            </button>
          </div>

          <button
            @click="handleMenuAction('openModal')"
            class="w-full py-3 rounded-full bg-gradient-to-r from-secondary-fixed via-primary to-primary-container text-on-primary font-bold text-xs sm:text-sm uppercase tracking-wider shadow-[0_0_15px_rgba(212,175,55,0.4)] flex items-center justify-center gap-2 cursor-pointer active:scale-95 transition"
          >
            <span class="material-symbols-outlined text-[18px]">airplane_ticket</span>
            <span>កក់កន្លែង (Reserve VIP Pass)</span>
          </button>

          <div class="space-y-2 pt-1">
            <button
              @click="handleMenuAction('replayDoor')"
              class="w-full px-3.5 py-2.5 rounded-xl bg-surface-container/60 hover:bg-surface-container text-slate-200 flex items-center justify-between text-xs sm:text-sm font-medium transition cursor-pointer"
            >
              <div class="flex items-center gap-2.5">
                <span class="material-symbols-outlined text-primary text-[20px]">meeting_room</span>
                <span>បើកទ្វារកិត្តិយសឡើងវិញ (Replay Door)</span>
              </div>
              <span class="material-symbols-outlined text-slate-400 text-[16px]">chevron_right</span>
            </button>

            <button
              @click="toggleVideo"
              class="w-full px-3.5 py-2.5 rounded-xl bg-surface-container/60 hover:bg-surface-container text-slate-200 flex items-center justify-between text-xs sm:text-sm font-medium transition cursor-pointer"
            >
              <div class="flex items-center gap-2.5">
                <span class="material-symbols-outlined text-primary text-[20px]">
                  {{ isVideoPlaying ? 'motion_photos_on' : 'motion_photos_paused' }}
                </span>
                <span>{{ isVideoPlaying ? 'ផ្អាកវីដេអូ Background' : 'ចាក់វីដេអូ Background' }}</span>
              </div>
              <span class="text-[10px] text-primary font-bold px-2 py-0.5 rounded bg-primary/10">
                {{ isVideoPlaying ? 'ON' : 'PAUSED' }}
              </span>
            </button>
          </div>

          <div class="space-y-1 pt-2 border-t border-outline-variant/30">
            <button @click="handleMenuNav('events')" class="w-full px-3.5 py-2 rounded-xl hover:bg-white/5 text-slate-200 hover:text-primary flex items-center gap-3 text-xs sm:text-sm font-medium transition cursor-pointer text-left">
              <span class="material-symbols-outlined text-primary text-[19px]">celebration</span>
              <span>កម្មវិធីរាត្រីសម្ពោធ (Events)</span>
            </button>
            <button @click="handleMenuNav('services')" class="w-full px-3.5 py-2 rounded-xl hover:bg-white/5 text-slate-200 hover:text-primary flex items-center gap-3 text-xs sm:text-sm font-medium transition cursor-pointer text-left">
              <span class="material-symbols-outlined text-primary text-[19px]">speaker</span>
              <span>សេវាកម្ម &amp; ការរាំរែក (Services)</span>
            </button>
            <button @click="handleMenuNav('schedule')" class="w-full px-3.5 py-2 rounded-xl hover:bg-white/5 text-slate-200 hover:text-primary flex items-center gap-3 text-xs sm:text-sm font-medium transition cursor-pointer text-left">
              <span class="material-symbols-outlined text-primary text-[19px]">calendar_month</span>
              <span>កាលវិភាគ (Schedule)</span>
            </button>
            <button @click="handleMenuNav('venue')" class="w-full px-3.5 py-2 rounded-xl hover:bg-white/5 text-slate-200 hover:text-primary flex items-center gap-3 text-xs sm:text-sm font-medium transition cursor-pointer text-left">
              <span class="material-symbols-outlined text-primary text-[19px]">location_on</span>
              <span>ទីតាំង &amp; Google Maps</span>
            </button>
            <button @click="handleMenuNav('tickets')" class="w-full px-3.5 py-2 rounded-xl hover:bg-white/5 text-slate-200 hover:text-primary flex items-center gap-3 text-xs sm:text-sm font-medium transition cursor-pointer text-left">
              <span class="material-symbols-outlined text-primary text-[19px]">table_bar</span>
              <span>កញ្ចប់សំបុត្រ VIP (Tiers)</span>
            </button>
          </div>

          <div class="pt-3 border-t border-outline-variant/30 grid grid-cols-2 gap-2">
            <a href="https://t.me/SLCLUB33" target="_blank" rel="noopener noreferrer" class="py-2.5 px-3 rounded-xl bg-[#229ED9]/15 hover:bg-[#229ED9]/25 text-[#229ED9] border border-[#229ED9]/30 text-xs font-semibold flex items-center justify-center gap-2 transition">
              <svg class="w-4 h-4 fill-current shrink-0" viewBox="0 0 24 24"><path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.446 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.242-.213-.054-.333-.373-.121l-6.871 4.326-2.962-.924c-.643-.204-.657-.643.136-.953l11.57-4.458c.538-.196 1.006.128.832.943z"/></svg>
              <span>Telegram</span>
            </a>
            <a href="tel:+18005557528" class="py-2.5 px-3 rounded-xl bg-primary/15 hover:bg-primary/25 text-primary border border-primary/30 text-xs font-semibold flex items-center justify-center gap-2 transition">
              <span class="material-symbols-outlined text-[18px]">call</span>
              <span>ទូរស័ព្ទ VIP</span>
            </a>
          </div>

        </div>
      </div>
    </Transition>

    <!-- MAIN CONTENT -->
    <main class="relative z-10 flex flex-col w-full pt-20 sm:pt-24 pb-32 md:pb-24 min-h-screen">
      <div class="max-w-7xl mx-auto w-full px-4 sm:px-6 lg:px-8 space-y-12 sm:space-y-16 lg:space-y-24">

        <!-- 1. HERO SECTION -->
        <section id="events" class="relative w-full pt-4 sm:pt-8 pb-4">
          <div class="grid grid-cols-1 lg:grid-cols-12 gap-8 lg:gap-12 items-center">

            <!-- Hero Left Column -->
            <div class="lg:col-span-7 flex flex-col items-center lg:items-start text-center lg:text-left space-y-4 sm:space-y-6">
              <div class="inline-flex items-center gap-2 px-4 py-1.5 rounded-full bg-black/75 shadow-lg backdrop-blur-md border border-primary/30">
                <span class="material-symbols-outlined text-primary text-[16px]">auto_awesome</span>
                <span class="font-label-sm text-xs sm:text-sm tracking-wider text-primary font-bold drop-shadow">
                  សំបុត្រគោរពអញ្ជើញផ្លូវការ • VIP GALA
                </span>
              </div>

              <div class="space-y-2 max-w-xl">
                <div class="inline-flex items-center gap-2 mb-1">
                  <span class="w-8 h-[1px] bg-gradient-to-r from-transparent to-primary"></span>
                  <span class="font-label-sm text-secondary tracking-widest uppercase text-xs sm:text-sm font-semibold drop-shadow">
                    លិខិតអញ្ជើញពិសេស
                  </span>
                  <span class="w-8 h-[1px] bg-gradient-to-l from-transparent to-primary"></span>
                </div>

                <h2 class="font-khmer-calligraphy text-3xl sm:text-5xl lg:text-5xl font-bold bg-gradient-to-r from-secondary-fixed via-primary to-primary-container bg-clip-text text-transparent drop-shadow-[0_2px_12px_rgba(0,0,0,0.95)] leading-tight">
                  សូមគោរពអញ្ជើញ
                </h2>
                <p class="font-body-md text-sm sm:text-base text-slate-200 drop-shadow-[0_1px_5px_rgba(0,0,0,0.9)]">
                  ចូលរួមជាភ្ញៀវកិត្តិយសក្នុងរាត្រីបើកសម្ពោធមហោឡារិក
                </p>
              </div>

              <div class="space-y-3 max-w-xl">
                <span class="inline-block font-label-sm text-secondary-fixed tracking-widest uppercase text-xs sm:text-sm font-semibold drop-shadow">
                  ការប្រគំតន្ត្រីផ្សាយផ្ទាល់ ភោជនីយដ្ឋានប្រណិត &amp; រាត្រីរាំរែក
                </span>
                <h1 class="font-headline-xl text-3xl sm:text-5xl lg:text-6xl font-extrabold bg-gradient-to-b from-secondary-fixed via-primary to-on-primary-container bg-clip-text text-transparent tracking-normal leading-tight drop-shadow-[0_4px_16px_rgba(0,0,0,0.95)]">
                  ពិធីសម្ពោធជាផ្លូវការ
                </h1>
                <p class="font-body-md text-sm sm:text-base text-slate-200 leading-relaxed drop-shadow-[0_1px_5px_rgba(0,0,0,0.95)]">
                  <strong class="text-primary font-bold">SL CLUB</strong> • ក្លឹបកម្សាន្តរាត្រីលំដាប់អភិជន ភោជនីយដ្ឋានប្រណិត តន្ត្រីផ្សាយផ្ទាល់ និងការរាំរែកសប្បាយរីករាយ ជាមួយសេវាកម្មកម្រិតកំពូល។
                </p>
              </div>

              <div class="flex flex-wrap items-center justify-center lg:justify-start gap-4 pt-2">
                <button
                  @click="openReservationModal()"
                  type="button"
                  class="h-12 px-6 sm:px-8 rounded-full bg-gradient-to-r from-primary via-primary-container to-secondary-fixed text-on-primary font-label-md text-sm sm:text-base font-bold shadow-[0_0_25px_rgba(212,175,55,0.5)] hover:shadow-[0_0_35px_rgba(212,175,55,0.8)] hover:scale-105 active:scale-95 transition-all cursor-pointer flex items-center gap-2"
                >
                  <span>កក់សំបុត្រ / ទទួលការអញ្ជើញ</span>
                  <span class="material-symbols-outlined text-[20px]">arrow_forward</span>
                </button>
                <button
                  @click="scrollToSection('schedule')"
                  type="button"
                  class="h-12 px-6 rounded-full bg-black/60 hover:bg-black/90 backdrop-blur-md border border-primary/30 text-slate-100 font-label-md text-sm sm:text-base transition cursor-pointer flex items-center gap-2 shadow-md"
                >
                  <span class="material-symbols-outlined text-[18px]">calendar_today</span>
                  <span>មើលកាលវិភាគ</span>
                </button>
              </div>
            </div>

            <!-- Hero Right Column: Grand VIP Stage & Countdown Showcase Frame -->
            <div class="lg:col-span-5 flex justify-center w-full">
              <div class="relative w-full max-w-md lg:max-w-none aspect-[4/3] sm:aspect-[16/11] min-h-[380px] sm:min-h-[440px] rounded-2xl overflow-hidden shadow-[0_25px_60px_rgba(0,0,0,0.95)] bg-gradient-to-b from-neutral-900 via-black to-neutral-950 border-2 border-primary/40 group flex flex-col justify-between p-4 sm:p-5">

                <!-- Video Ambience Background -->
                <div class="absolute inset-0 z-0 overflow-hidden">
                  <video
                    autoplay
                    loop
                    muted
                    playsinline
                    class="w-full h-full object-cover object-center filter brightness-[0.85] contrast-[1.1] transform group-hover:scale-105 transition-transform duration-700 ease-out"
                    src="/bg-video.mp4"
                  ></video>
                  <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-black/40 to-black/70"></div>
                  <div class="absolute inset-0 bg-[radial-gradient(circle_at_center,_rgba(242,202,80,0.15)_0%,_transparent_70%)]"></div>
                </div>

                <!-- Top Row: Official Gala Badges -->
                <div class="relative z-10 flex items-center justify-between gap-2">
                  <div class="px-3 py-1.5 rounded-xl bg-black/85 backdrop-blur-md flex items-center gap-2 border border-primary/40 shadow-lg">
                    <img src="/logo.jpg" alt="SL Club Logo" class="w-6 h-6 rounded object-contain" />
                    <span class="font-label-sm text-xs sm:text-sm text-primary-fixed uppercase tracking-wider font-semibold drop-shadow">
                      រាត្រីសម្ពោធផ្លូវការ
                    </span>
                  </div>

                  <div class="px-2.5 py-1 rounded-full bg-primary/20 backdrop-blur-md border border-primary/30 flex items-center gap-1.5 shadow-sm">
                    <span class="w-2 h-2 rounded-full bg-primary animate-ping"></span>
                    <span class="text-[10px] sm:text-xs text-primary font-bold uppercase tracking-wider">VIP GALA</span>
                  </div>
                </div>

                <!-- Center: SL Club Crest & Glow -->
                <div class="relative z-10 my-auto flex flex-col items-center text-center py-4">
                  <div class="w-20 h-20 sm:w-24 sm:h-24 rounded-2xl p-1 bg-gradient-to-b from-secondary-fixed via-primary to-primary-container shadow-[0_0_35px_rgba(242,202,80,0.6)] group-hover:scale-110 transition-transform duration-500">
                    <div class="w-full h-full rounded-xl bg-black/95 p-2 flex items-center justify-center overflow-hidden border border-primary/40">
                      <img src="/logo.jpg" alt="SL Club Emblem" class="w-full h-full object-contain filter drop-shadow-[0_0_10px_rgba(242,202,80,0.7)]" />
                    </div>
                  </div>
                  <h3 class="font-headline-sm text-lg sm:text-xl font-extrabold text-primary tracking-wider mt-3 drop-shadow-[0_2px_10px_rgba(0,0,0,0.95)]">
                    SL CLUB GRAND OPENING
                  </h3>
                  <p class="font-label-sm text-[11px] sm:text-xs text-slate-300 drop-shadow">
                    Luxury Lounge • Live Concert • Fine Dining
                  </p>
                </div>

                <!-- Bottom Row: Live Countdown Bar -->
                <div class="relative z-10 p-3 sm:p-4 rounded-xl bg-black/90 backdrop-blur-xl flex items-center justify-between shadow-2xl border border-primary/40">
                  <div class="flex items-center gap-2">
                    <span class="material-symbols-outlined text-tertiary text-[18px] sm:text-[22px]">timer</span>
                    <span class="font-label-sm text-xs sm:text-sm text-slate-200 font-medium">រាប់ថយក្រោយ</span>
                  </div>
                  <div class="flex items-center gap-1 font-label-md text-xs sm:text-sm md:text-base text-primary font-bold tracking-normal drop-shadow">
                    {{ countdownText }}
                  </div>
                </div>

              </div>
            </div>

          </div>
        </section>

        <!-- 2. KEY EVENT HIGHLIGHT PILLS -->
        <section id="services" class="w-full">
          <div class="flex flex-col sm:flex-row sm:items-center justify-between mb-4 sm:mb-6 gap-2">
            <div class="flex flex-col">
              <span class="font-label-sm text-xs sm:text-sm uppercase tracking-wider text-primary font-semibold drop-shadow">
                សេវាកម្ម និងការកម្សាន្តលំដាប់ខ្ពស់
              </span>
              <h3 class="font-headline-sm text-xl sm:text-3xl text-slate-100 font-bold drop-shadow">
                បទពិសោធន៍ដ៏អស្ចារ្យនៅ SL Club
              </h3>
            </div>
            <span class="font-label-sm text-xs sm:text-sm text-primary flex items-center gap-1 bg-black/75 backdrop-blur-md px-3.5 py-1.5 rounded-full self-start sm:self-auto border border-primary/30 shadow-md">
              <span class="material-symbols-outlined text-[16px]">verified</span> សម្រាប់ភ្ញៀវកិត្តិយស &amp; VIP
            </span>
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 sm:gap-6">
            <div class="p-5 sm:p-6 rounded-2xl bg-[#141414]/80 backdrop-blur-xl flex flex-col gap-3 shadow-xl border border-outline-variant/30 hover:border-primary/50 transition-all hover:bg-[#1a1a1a]/85 group">
              <div class="w-12 h-12 rounded-xl bg-secondary-container/80 flex items-center justify-center text-primary group-hover:scale-110 transition-transform shadow-md">
                <span class="material-symbols-outlined text-[24px]">restaurant</span>
              </div>
              <h4 class="font-label-lg text-base sm:text-lg text-slate-100 font-semibold leading-snug drop-shadow-sm">
                ភោជនីយដ្ឋានលំដាប់ផ្កាយ ៥
              </h4>
              <p class="font-body-sm text-xs sm:text-sm text-slate-300 leading-relaxed">
                ម្ហូបបែប Fine Dining ចម្អិនដោយចុងភៅជំនាញ និងភេសជ្ជៈលំដាប់ខ្ពស់ប្រកបដោយអនាម័យ និងរសជាតិឆ្ងាញ់ពិសា។
              </p>
            </div>

            <div class="p-5 sm:p-6 rounded-2xl bg-[#141414]/80 backdrop-blur-xl flex flex-col gap-3 shadow-xl border border-outline-variant/30 hover:border-primary/50 transition-all hover:bg-[#1a1a1a]/85 group">
              <div class="w-12 h-12 rounded-xl bg-secondary-container/80 flex items-center justify-center text-primary group-hover:scale-110 transition-transform shadow-md">
                <span class="material-symbols-outlined text-[24px]">speaker</span>
              </div>
              <h4 class="font-label-lg text-base sm:text-lg text-slate-100 font-semibold leading-snug drop-shadow-sm">
                ឆាកតន្ត្រី &amp; កម្រាលរាំ Dance
              </h4>
              <p class="font-body-sm text-xs sm:text-sm text-slate-300 leading-relaxed">
                ប្រព័ន្ធសំឡេង Sound System កម្រិតពិភពលោក ភ្លើង Laser 3D និងការចាក់ភ្លេងរាំរែកដោយកំពូលឌីជេល្បីៗ។
              </p>
            </div>

            <div class="p-5 sm:p-6 rounded-2xl bg-[#141414]/80 backdrop-blur-xl flex flex-col gap-3 shadow-xl border border-outline-variant/30 hover:border-primary/50 transition-all hover:bg-[#1a1a1a]/85 group">
              <div class="w-12 h-12 rounded-xl bg-secondary-container/80 flex items-center justify-center text-primary group-hover:scale-110 transition-transform shadow-md">
                <span class="material-symbols-outlined text-[24px]">local_bar</span>
              </div>
              <h4 class="font-label-lg text-base sm:text-lg text-slate-100 font-semibold leading-snug drop-shadow-sm">
                Reserve Mixology
              </h4>
              <p class="font-body-sm text-xs sm:text-sm text-slate-300 leading-relaxed">
                ស្រាសំប៉ាញល្បីៗប្រចាំប្រទេស និងស្រាក្រឡុកច្នៃប្រណិតពិសេសរៀបចំដោយ Mixologist ជំនាញ។
              </p>
            </div>

            <div class="p-5 sm:p-6 rounded-2xl bg-[#141414]/80 backdrop-blur-xl flex flex-col gap-3 shadow-xl border border-outline-variant/30 hover:border-primary/50 transition-all hover:bg-[#1a1a1a]/85 group">
              <div class="w-12 h-12 rounded-xl bg-secondary-container/80 flex items-center justify-center text-primary group-hover:scale-110 transition-transform shadow-md">
                <span class="material-symbols-outlined text-[24px]">table_restaurant</span>
              </div>
              <h4 class="font-label-lg text-base sm:text-lg text-slate-100 font-semibold leading-snug drop-shadow-sm">
                បន្ទប់ VIP Skybox Lounge
              </h4>
              <p class="font-body-sm text-xs sm:text-sm text-slate-300 leading-relaxed">
                តុអង្គុយឯកជនភាពខ្ពស់ មានទិដ្ឋភាពច្បាស់ជាងគេ ជាមួយសេវាកម្មអ្នកបម្រើផ្ទាល់ (Butler Service)។
              </p>
            </div>
          </div>
        </section>

        <!-- 3. EVENT ITINERARY & SCHEDULE -->
        <section id="schedule" class="w-full">
          <div class="rounded-2xl p-6 sm:p-8 lg:p-10 bg-[#121212]/85 backdrop-blur-2xl shadow-2xl flex flex-col gap-8 border border-primary/25">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-outline-variant/30 pb-6">
              <div class="flex flex-col">
                <span class="font-label-sm text-xs sm:text-sm uppercase tracking-wider text-primary font-semibold drop-shadow">
                  កាលវិភាគរាត្រីសម្ពោធ
                </span>
                <h2 class="font-headline-md text-2xl sm:text-3xl text-slate-100 font-bold mt-1 drop-shadow">
                  ថ្ងៃសុក្រ ទី ០៩ ខែតុលា ឆ្នាំ ២០២៦
                </h2>
                <span class="font-body-sm text-xs sm:text-sm text-slate-300 mt-0.5">
                  ចាប់បើកទ្វារទទួលភ្ញៀវវេលាម៉ោង ៦:០០ ល្ងាច តទៅ
                </span>
              </div>
              <div class="px-4 py-2 rounded-xl bg-secondary-container/90 text-secondary font-label-sm text-xs sm:text-sm text-center shadow-inner leading-normal font-medium self-start sm:self-auto border border-primary/20">
                សម្លៀកបំពាក់៖ <span class="font-bold text-secondary-fixed">Black Tie / Cocktail</span>
              </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 relative">
              <div class="p-4 sm:p-5 rounded-xl bg-black/50 backdrop-blur-md flex items-start gap-4 border border-outline-variant/30 hover:border-primary/40 transition">
                <div class="w-12 h-12 rounded-xl bg-surface-container-highest/90 flex items-center justify-center shrink-0 shadow-md border border-primary/30 text-primary">
                  <span class="material-symbols-outlined text-[22px]">celebration</span>
                </div>
                <div class="flex flex-col min-w-0 flex-1">
                  <div class="flex items-center justify-between gap-2 mb-1">
                    <span class="font-label-md text-sm sm:text-base text-primary font-bold drop-shadow">១៨:០០ - ១៩:១៥</span>
                    <span class="font-label-sm text-xs text-slate-300 bg-surface-container-high/80 px-2 py-0.5 rounded">Grand Portico</span>
                  </div>
                  <h5 class="font-body-md text-sm sm:text-base text-slate-100 font-semibold leading-tight">ទទួលស្វាគមន៍កម្រាលព្រំក្រហម &amp; ស្រាសំប៉ាញ</h5>
                  <p class="font-body-sm text-xs sm:text-sm text-slate-300 mt-1 leading-relaxed">
                    ការទទួលបដិសណ្ឋារកិច្ចដោយ Dom Pérignon និងអាហារសម្រន់ Canapés ពិសេស។
                  </p>
                </div>
              </div>

              <div class="p-4 sm:p-5 rounded-xl bg-black/50 backdrop-blur-md flex items-start gap-4 border border-outline-variant/30 hover:border-primary/40 transition">
                <div class="w-12 h-12 rounded-xl bg-secondary-container/90 flex items-center justify-center shrink-0 shadow-md border border-primary/40 text-primary">
                  <span class="material-symbols-outlined text-[22px]">restaurant</span>
                </div>
                <div class="flex flex-col min-w-0 flex-1">
                  <div class="flex items-center justify-between gap-2 mb-1">
                    <span class="font-label-md text-sm sm:text-base text-primary font-bold drop-shadow">១៩:៣០ - ២០:៤៥</span>
                    <span class="font-label-sm text-xs text-tertiary bg-surface-container-high/80 px-2 py-0.5 rounded">Grand Dining Hall</span>
                  </div>
                  <h5 class="font-body-md text-sm sm:text-base text-slate-100 font-semibold leading-tight">ពិធីពិសាភោជនាហារ និងស្រាសំប៉ាញ (Gourmet Dining)</h5>
                  <p class="font-body-sm text-xs sm:text-sm text-slate-300 mt-1 leading-relaxed">
                    មុខម្ហូបពិសេសច្នៃដោយចុងភៅលំដាប់ផ្កាយ ៥ អមដោយការប្រគំភ្លេង Jazz &amp; Saxophone ផ្ទាល់។
                  </p>
                </div>
              </div>

              <div class="p-4 sm:p-5 rounded-xl bg-black/50 backdrop-blur-md flex items-start gap-4 border border-outline-variant/30 hover:border-primary/40 transition">
                <div class="w-12 h-12 rounded-xl bg-surface-container-highest/90 flex items-center justify-center shrink-0 shadow-md border border-primary/30 text-primary">
                  <span class="material-symbols-outlined text-[22px]">music_note</span>
                </div>
                <div class="flex flex-col min-w-0 flex-1">
                  <div class="flex items-center justify-between gap-2 mb-1">
                    <span class="font-label-md text-sm sm:text-base text-primary font-bold drop-shadow">២១:០០ - ២២:៤៥</span>
                    <span class="font-label-sm text-xs text-slate-300 bg-surface-container-high/80 px-2 py-0.5 rounded">Main Concert Stage</span>
                  </div>
                  <h5 class="font-body-md text-sm sm:text-base text-slate-100 font-semibold leading-tight">ការប្រគំតន្ត្រីផ្សាយផ្ទាល់ &amp; ការរាំរែក VIP Live Concert</h5>
                  <p class="font-body-sm text-xs sm:text-sm text-slate-300 mt-1 leading-relaxed">
                    ការសម្តែងតន្ត្រីផ្ទាល់យ៉ាងកក្រើក និងក្បាច់រាំអស្ចារ្យពីកំពូលតារាចម្រៀងល្បីៗប្រចាំប្រទេស។
                  </p>
                </div>
              </div>

              <div class="p-4 sm:p-5 rounded-xl bg-black/50 backdrop-blur-md flex items-start gap-4 border border-outline-variant/30 hover:border-primary/40 transition">
                <div class="w-12 h-12 rounded-xl bg-surface-container-highest/90 flex items-center justify-center shrink-0 shadow-md border border-primary/30 text-primary">
                  <span class="material-symbols-outlined text-[22px]">nightlife</span>
                </div>
                <div class="flex flex-col min-w-0 flex-1">
                  <div class="flex items-center justify-between gap-2 mb-1">
                    <span class="font-label-md text-sm sm:text-base text-primary font-bold drop-shadow">២៣:០០ - យប់ជ្រៅ</span>
                    <span class="font-label-sm text-xs text-slate-300 bg-surface-container-high/80 px-2 py-0.5 rounded">Dance Floor &amp; Lounge</span>
                  </div>
                  <h5 class="font-body-md text-sm sm:text-base text-slate-100 font-semibold leading-tight">រាត្រីកម្សាន្តរាំរែក Afterparty &amp; EDM DJ Blast</h5>
                  <p class="font-body-sm text-xs sm:text-sm text-slate-300 mt-1 leading-relaxed">
                    តន្ត្រី EDM / Deep House ដោយឌីជេលំដាប់ជួរមុខ ការរាំរែកសប្បាយពេញទំហឹង និងសេវាកម្មតុ VIP។
                  </p>
                </div>
              </div>
            </div>
          </div>
        </section>

        <!-- 4. VENUE & LUXURY LOCATION -->
        <section id="venue" class="w-full">
          <div class="rounded-2xl overflow-hidden bg-[#121212]/85 backdrop-blur-2xl shadow-2xl grid grid-cols-1 lg:grid-cols-12 border border-primary/25">
            <div
              @click="openDirections"
              title="ចុចដើម្បីបើក Google Maps"
              class="relative lg:col-span-7 h-64 sm:h-80 lg:h-full min-h-[300px] bg-cover bg-center overflow-hidden cursor-pointer group bg-[#161616]"
              style="background-image: url('https://lh3.googleusercontent.com/aida-public/AB6AXuDWOs7-RBh4vvU7kXYnb5DJrmGIUOW0vKz9vicZCbVcfH47u1dLB402cTN5rYINsL3tslojQViGM515dqBxE7deRF4lgL1oHloaMzPMdiVk-_Dbc7Ur3MsHIXYJkP5C0U_SflXg8mjhWmy6-dPZro59v5zM_5Tqb6FXcGFbsdftaiNCUKnfKOSvH0AzhUPTWR6AYdl4s6csvCE1RexzSUCY40hFMRXSx6TwKTfCCd2G')"
            >
              <div class="absolute inset-0 bg-gradient-to-t from-[#121212] via-black/40 to-transparent group-hover:via-black/20 transition-all"></div>
              <div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 flex flex-col items-center group-hover:scale-110 transition-transform">
                <div class="relative flex items-center justify-center">
                  <div class="w-12 h-12 rounded-full bg-primary/25 animate-ping absolute"></div>
                  <div class="w-10 h-10 rounded-full bg-primary flex items-center justify-center shadow-[0_0_20px_rgba(242,202,80,0.8)] text-on-primary">
                    <span class="material-symbols-outlined text-[22px]">location_on</span>
                  </div>
                </div>
                <div class="mt-2 px-3.5 py-1 rounded-full bg-black/90 backdrop-blur-md shadow-lg text-center border border-primary/40 flex items-center gap-1.5">
                  <span class="font-label-sm text-xs sm:text-sm text-primary font-bold">SL CLUB &amp; LOUNGE</span>
                  <span class="material-symbols-outlined text-xs text-primary">open_in_new</span>
                </div>
              </div>

              <div class="absolute top-4 left-4 px-3 py-1.5 rounded-lg bg-black/85 backdrop-blur-sm flex items-center gap-2 border border-primary/30">
                <span class="material-symbols-outlined text-primary text-[16px]">map</span>
                <span class="font-label-sm text-xs sm:text-sm text-slate-200 font-medium">ចុចមើលលើ Google Maps</span>
              </div>
            </div>

            <div class="lg:col-span-5 p-6 sm:p-8 lg:p-10 flex flex-col justify-between gap-6">
              <div class="space-y-3">
                <span class="font-label-sm text-xs sm:text-sm text-primary font-semibold uppercase tracking-wider drop-shadow">
                  ទីតាំង និងការធ្វើដំណើរ
                </span>
                <h3 class="font-headline-sm text-2xl sm:text-3xl text-slate-100 font-bold leading-tight drop-shadow">
                  The Club &amp; Luxury Lounge
                </h3>
                <p class="font-body-sm text-sm sm:text-base text-slate-300 leading-relaxed">
                  មហាវិថី 888 Grand Boulevard, Skyline District, Suite 100
                </p>

                <div class="pt-2 flex flex-wrap gap-3">
                  <a
                    href="https://maps.app.goo.gl/hSJKgn4A28KUYYnm8"
                    target="_blank"
                    rel="noopener noreferrer"
                    class="h-10 px-5 rounded-full bg-secondary-container/90 text-secondary hover:bg-secondary-container transition-all flex items-center gap-2 font-label-sm text-xs sm:text-sm font-bold cursor-pointer border border-primary/20 shadow-md"
                  >
                    <span class="material-symbols-outlined text-[18px]">directions</span>
                    <span>ទិសដៅធ្វើដំណើរ (Google Maps)</span>
                  </a>

                  <a
                    href="https://t.me/SLCLUB33"
                    target="_blank"
                    rel="noopener noreferrer"
                    class="h-10 px-5 rounded-full bg-[#229ED9]/20 hover:bg-[#229ED9]/35 text-[#229ED9] border border-[#229ED9]/40 transition-all flex items-center gap-2 font-label-sm text-xs sm:text-sm font-bold shadow-md cursor-pointer"
                  >
                    <svg class="w-4 h-4 fill-current" viewBox="0 0 24 24"><path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.446 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.242-.213-.054-.333-.373-.121l-6.871 4.326-2.962-.924c-.643-.204-.657-.643.136-.953l11.57-4.458c.538-.196 1.006.128.832.943z"/></svg>
                    <span>Telegram Channel: @SLCLUB33</span>
                  </a>
                </div>
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 pt-4 border-t border-outline-variant/30">
                <div class="p-3.5 rounded-xl bg-black/50 backdrop-blur-md flex items-center gap-3 border border-outline-variant/30">
                  <span class="material-symbols-outlined text-primary text-[24px]">directions_car</span>
                  <div class="flex flex-col">
                    <span class="font-label-sm text-xs sm:text-sm text-slate-200 font-semibold">សេវាចតរថយន្ត VIP</span>
                    <span class="font-body-sm text-[11px] sm:text-xs text-slate-400">ច្រកចូល Porte-cochère</span>
                  </div>
                </div>

                <div class="p-3.5 rounded-xl bg-black/50 backdrop-blur-md flex items-center gap-3 border border-outline-variant/30">
                  <span class="material-symbols-outlined text-primary text-[24px]">flight</span>
                  <div class="flex flex-col">
                    <span class="font-label-sm text-xs sm:text-sm text-slate-200 font-semibold">ចំណតឧទ្ធម្ភាគចក្រ</span>
                    <span class="font-body-sm text-[11px] sm:text-xs text-slate-400">Deck 4 ជាន់លើបង្អស់</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </section>

        <!-- 5. VIP RESERVATION TIER QUICK PICK -->
        <section id="tickets" class="w-full">
          <div class="flex flex-col sm:flex-row sm:items-center justify-between mb-6 gap-2">
            <div class="flex flex-col">
              <span class="font-label-sm text-xs sm:text-sm uppercase tracking-wider text-primary font-semibold drop-shadow">
                កញ្ចប់អញ្ជើញ និងកន្លែងអង្គុយ VIP
              </span>
              <h3 class="font-headline-sm text-2xl sm:text-3xl text-slate-100 font-bold drop-shadow">
                ជ្រើសរើសប្រភេទសំបុត្រកិត្តិយស
              </h3>
            </div>
            <span class="font-label-sm text-xs sm:text-sm text-tertiary bg-black/75 backdrop-blur-md px-3.5 py-1.5 rounded-full border border-tertiary/30 self-start sm:self-auto font-medium shadow-md">
              នៅសល់តែ ៣៨ កន្លែងទៀតប៉ុណ្ណោះ
            </span>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div
              v-for="tier in tiers"
              :key="tier.id"
              @click="selectTier(tier)"
              :class="[
                'p-6 sm:p-8 rounded-2xl transition-all cursor-pointer flex flex-col justify-between relative backdrop-blur-2xl',
                tier.featured ? 'bg-[#151515]/90 shadow-2xl' : 'bg-[#101010]/85 shadow-lg hover:bg-[#181818]/90',
                selectedTier.id === tier.id ? 'border-2 border-primary ring-2 ring-primary/40' : 'border border-outline-variant/30'
              ]"
            >
              <div
                v-if="tier.featured"
                class="absolute -top-3 right-6 px-3.5 py-1 rounded-full bg-primary text-on-primary font-label-sm text-xs font-bold tracking-normal shadow-lg flex items-center gap-1"
              >
                <span class="material-symbols-outlined text-[14px]">stars</span>
                <span>{{ tier.level }}</span>
              </div>

              <div class="space-y-4">
                <div class="flex items-start justify-between gap-4">
                  <div>
                    <h4 :class="['font-label-lg text-lg sm:text-xl font-bold', tier.featured ? 'text-secondary-fixed' : 'text-slate-100']">
                      {{ tier.title }}
                    </h4>
                    <span v-if="!tier.featured" class="inline-block mt-1 font-label-sm text-xs px-2 py-0.5 rounded bg-surface-container-highest/80 text-slate-300 font-medium">
                      {{ tier.level }}
                    </span>
                  </div>
                  <div class="text-right shrink-0">
                    <span class="font-headline-sm text-2xl sm:text-3xl text-primary font-extrabold drop-shadow">{{ tier.cost }}</span>
                    <span class="block font-label-sm text-xs text-slate-400">ក្នុងមួយរូប</span>
                  </div>
                </div>

                <p class="font-body-sm text-xs sm:text-sm text-slate-300 leading-relaxed">
                  {{ tier.description }}
                </p>

                <ul class="space-y-2 pt-2 border-t border-outline-variant/30">
                  <li v-for="(f, idx) in tier.features" :key="idx" class="flex items-center gap-2 text-xs sm:text-sm text-slate-200">
                    <span class="material-symbols-outlined text-primary text-[16px] shrink-0">check_circle</span>
                    <span>{{ f }}</span>
                  </li>
                </ul>
              </div>

              <div class="pt-6 mt-4">
                <button
                  type="button"
                  :class="[
                    'w-full py-3 rounded-full font-label-md text-xs sm:text-sm font-bold transition flex items-center justify-center gap-2 cursor-pointer shadow-md',
                    selectedTier.id === tier.id
                      ? 'bg-primary text-on-primary shadow-primary/30'
                      : 'bg-black/60 text-slate-200 hover:bg-black/90 border border-primary/20'
                  ]"
                >
                  <span class="material-symbols-outlined text-[18px]">
                    {{ selectedTier.id === tier.id ? 'radio_button_checked' : 'radio_button_unchecked' }}
                  </span>
                  <span>{{ selectedTier.id === tier.id ? 'បានជ្រើសរើស' : 'ចុចជ្រើសរើសកញ្ចប់នេះ' }}</span>
                </button>
              </div>

            </div>
          </div>
        </section>

      </div>
    </main>

    <!-- PERSISTENT STICKY VIP ACTION BANNER -->
    <div class="fixed bottom-20 md:bottom-6 inset-x-0 z-40 px-4 pointer-events-none">
      <div class="max-w-2xl mx-auto bg-black/95 backdrop-blur-2xl p-3 sm:p-4 rounded-2xl shadow-[0_10px_50px_rgba(0,0,0,0.95)] border border-primary/40 flex items-center justify-between gap-3 pointer-events-auto">
        <div class="flex flex-col min-w-0 pl-1">
          <div class="flex items-center gap-1.5">
            <span class="font-label-sm text-xs text-primary font-bold tracking-normal">កញ្ចប់ដែលបានជ្រើស៖</span>
            <span class="w-1.5 h-1.5 rounded-full bg-tertiary-container animate-pulse"></span>
          </div>
          <div class="flex items-baseline gap-1.5">
            <span class="font-headline-sm text-lg sm:text-2xl text-slate-100 font-extrabold">{{ selectedTier.cost }}</span>
            <span class="font-label-sm text-xs text-slate-300 truncate">({{ selectedTier.title }})</span>
          </div>
        </div>

        <div class="flex items-center gap-2 shrink-0">
          <a
            aria-label="Telegram Channel @SLCLUB33"
            title="Telegram Channel @SLCLUB33"
            target="_blank"
            rel="noopener noreferrer"
            class="w-10 h-10 sm:w-11 sm:h-11 rounded-full bg-[#229ED9]/20 hover:bg-[#229ED9]/35 text-[#229ED9] flex items-center justify-center transition-colors shadow-md border border-[#229ED9]/40"
            href="https://t.me/SLCLUB33"
          >
            <svg class="w-4 h-4 sm:w-5 sm:h-5 fill-current" viewBox="0 0 24 24"><path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.446 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.242-.213-.054-.333-.373-.121l-6.871 4.326-2.962-.924c-.643-.204-.657-.643.136-.953l11.57-4.458c.538-.196 1.006.128.832.943z"/></svg>
          </a>

          <a
            aria-label="ទូរស័ព្ទទៅកាន់សេវាបម្រើអតិថិជន VIP"
            class="w-10 h-10 sm:w-11 sm:h-11 rounded-full bg-[#1c1c1c] hover:bg-[#252525] text-primary flex items-center justify-center transition-colors shadow-md border border-primary/30"
            href="tel:+18005557528"
          >
            <span class="material-symbols-outlined text-[20px]">phone_in_talk</span>
          </a>

          <button
            @click="openReservationModal()"
            type="button"
            class="h-10 sm:h-11 px-4 sm:px-6 rounded-full bg-gradient-to-r from-primary via-primary-container to-secondary-fixed text-on-primary font-label-md text-xs sm:text-sm font-bold shadow-[0_0_20px_rgba(212,175,55,0.5)] hover:shadow-[0_0_28px_rgba(212,175,55,0.75)] flex items-center gap-1.5 transition-all transform active:scale-95 cursor-pointer"
          >
            <span>កក់សំបុត្រឥឡូវនេះ</span>
            <span class="material-symbols-outlined text-[18px]">arrow_forward</span>
          </button>
        </div>
      </div>
    </div>

    <!-- RESERVATION CONFIRMATION DIALOG -->
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
        <div class="bg-[#141414]/95 backdrop-blur-2xl p-6 sm:p-8 rounded-2xl shadow-[0_25px_70px_rgba(0,0,0,0.95)] flex flex-col gap-4 text-on-surface border border-primary/40 w-full max-w-md">
          <div class="flex items-center justify-between">
            <div class="flex items-center gap-3">
              <img src="/logo.jpg" alt="SL Club Logo" class="w-8 h-8 rounded-lg object-contain border border-primary/40 bg-black/80 shadow-md" />
              <span class="font-headline-sm text-lg sm:text-xl font-bold text-slate-100">ការចុះឈ្មោះ VIP</span>
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
                <svg class="w-4 h-4 fill-current shrink-0" viewBox="0 0 24 24"><path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.446 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.242-.213-.054-.333-.373-.121l-6.871 4.326-2.962-.924c-.643-.204-.657-.643.136-.953l11.57-4.458c.538-.196 1.006.128.832.943z"/></svg>
                <span>ទាក់ទង / តាមដាន Telegram: @SLCLUB33</span>
              </a>
            </div>
          </div>

          <div class="flex items-center justify-end gap-3 pt-4 border-t border-outline-variant/30 mt-2">
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
              class="h-10 px-6 rounded-full bg-primary text-on-primary font-label-md text-xs sm:text-sm font-bold shadow-lg hover:bg-primary/90 transition cursor-pointer"
            >
              បញ្ជាក់ការកក់
            </button>
          </div>
        </div>
      </div>
    </Transition>

    <!-- MOBILE BOTTOM NAVIGATION BAR -->
    <nav class="md:hidden fixed bottom-0 inset-x-0 z-50 pb-safe bg-black/90 backdrop-blur-xl shadow-[0_-8px_30px_rgba(0,0,0,0.85)] border-t border-primary/20">
      <div class="flex justify-around items-center h-20 px-2 max-w-md mx-auto">
        <button @click="scrollToSection('events')" type="button" class="flex flex-col items-center justify-center gap-1 w-16 h-14 transition-colors cursor-pointer text-primary">
          <span class="material-symbols-outlined text-[22px]">celebration</span>
          <span class="font-label-sm text-[11px]">កម្មវិធី</span>
        </button>
        <button @click="scrollToSection('services')" type="button" class="flex flex-col items-center justify-center gap-1 w-16 h-14 transition-colors cursor-pointer text-slate-300 hover:text-primary">
          <span class="material-symbols-outlined text-[22px]">speaker</span>
          <span class="font-label-sm text-[11px]">សេវាកម្ម</span>
        </button>
        <button @click="scrollToSection('schedule')" type="button" class="flex flex-col items-center justify-center gap-1 w-16 h-14 transition-colors cursor-pointer text-slate-300 hover:text-primary">
          <span class="material-symbols-outlined text-[22px]">calendar_month</span>
          <span class="font-label-sm text-[11px]">កាលវិភាគ</span>
        </button>
        <button @click="scrollToSection('tickets')" type="button" class="flex flex-col items-center justify-center gap-1 w-16 h-14 transition-colors cursor-pointer text-slate-300 hover:text-primary">
          <span class="material-symbols-outlined text-[22px]">table_bar</span>
          <span class="font-label-sm text-[11px]">កញ្ចប់ VIP</span>
        </button>
      </div>
    </nav>

  </div>
</template>

