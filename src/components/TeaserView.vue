<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  isMusicPlaying: {
    type: Boolean,
    default: false
  },
  targetDate: {
    type: Date,
    default: () => new Date('2026-11-09T19:00:00')
  }
})

const emit = defineEmits(['toggle-music', 'view-official'])

// Video ref & state
const videoRef = ref(null)
const isVideoLoaded = ref(false)

// Countdown state
const days = ref(0)
const hours = ref(0)
const minutes = ref(0)
const seconds = ref(0)
const isExpired = ref(false)
let timerId = null

function updateCountdown() {
  const now = new Date().getTime()
  const target = props.targetDate.getTime()
  const diff = target - now

  if (diff <= 0) {
    days.value = 0
    hours.value = 0
    minutes.value = 0
    seconds.value = 0
    isExpired.value = true
    return
  }

  days.value = Math.floor(diff / (1000 * 60 * 60 * 24))
  hours.value = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
  minutes.value = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60))
  seconds.value = Math.floor((diff % (1000 * 60)) / 1000)
}

// Convert numbers to 2-digit strings
const formattedDays = computed(() => String(days.value).padStart(2, '0'))
const formattedHours = computed(() => String(hours.value).padStart(2, '0'))
const formattedMinutes = computed(() => String(minutes.value).padStart(2, '0'))
const formattedSeconds = computed(() => String(seconds.value).padStart(2, '0'))

// Khmer numeral conversion
function toKhmerNum(num) {
  const khmerDigits = ['០', '១', '២', '៣', '៤', '៥', '៦', '៧', '៨', '៩']
  return String(num).split('').map(d => khmerDigits[parseInt(d)] ?? d).join('')
}

const khmerDays = computed(() => toKhmerNum(formattedDays.value))
const khmerHours = computed(() => toKhmerNum(formattedHours.value))
const khmerMinutes = computed(() => toKhmerNum(formattedMinutes.value))
const khmerSeconds = computed(() => toKhmerNum(formattedSeconds.value))

function scrollToCountdown() {
  const el = document.getElementById('teaser-countdown')
  if (el) {
    el.scrollIntoView({ behavior: 'smooth' })
  }
}

function openMaps() {
  window.open('https://maps.app.goo.gl/hSJKgn4A28KUYYnm8', '_blank')
}

function openTelegram() {
  window.open('https://t.me/SLCLUB33', '_blank')
}

onMounted(() => {
  updateCountdown()
  timerId = setInterval(updateCountdown, 1000)
})

onUnmounted(() => {
  if (timerId) clearInterval(timerId)
})
</script>

<template>
  <div class="relative w-full min-h-screen bg-[#08080a] text-slate-100 overflow-x-hidden selection:bg-amber-400 selection:text-black">
    <!-- ================= FIXED BACKGROUND VIDEO LAYER ================= -->
    <div class="fixed inset-0 z-0 pointer-events-none overflow-hidden">
      <video
        ref="videoRef"
        autoplay
        loop
        muted
        playsinline
        preload="auto"
        class="w-full h-full object-cover opacity-45 scale-105 filter brightness-[0.8] contrast-[1.1] transition-opacity duration-1000"
      >
        <source src="/bg-video.mp4" type="video/mp4" />
      </video>

      <!-- Atmospheric Fog & Club Dark Vignette Overlays -->
      <div class="absolute inset-0 bg-radial-[circle_at_50%_20%] from-transparent via-[#090a0f]/75 to-[#050608]/95"></div>
      <div class="absolute inset-0 bg-gradient-to-b from-[#08090d]/90 via-transparent to-[#050608]/95"></div>
      <div class="absolute inset-0 backdrop-blur-[0.5px]"></div>
    </div>

    <!-- ================= FLOATING TOP CONTROLS ================= -->
    <header class="fixed top-0 inset-x-0 z-50 flex items-center justify-between px-4 sm:px-8 py-3.5 sm:py-4 bg-black/40 backdrop-blur-md border-b border-white/10">
      <!-- Brand Crest -->
      <div class="flex items-center gap-3">
        <div class="relative w-9 h-9 sm:w-10 sm:h-10 rounded-lg p-0.5 bg-gradient-to-b from-slate-200 via-slate-400 to-slate-700 shadow-[0_0_15px_rgba(255,255,255,0.3)]">
          <img src="/logo.jpg" alt="SL Club" class="w-full h-full object-contain rounded-md" />
        </div>
        <div class="flex flex-col">
          <span class="text-xs sm:text-sm font-black tracking-widest text-slate-100 uppercase font-sans">SL CLUB</span>
          <span class="text-[10px] text-amber-400 tracking-wider">COMING SOON</span>
        </div>
      </div>

      <!-- Right Action Items -->
      <div class="flex items-center gap-2 sm:gap-3">
        <!-- Official Reveal Date Badge -->
        <div class="hidden sm:flex items-center gap-1.5 px-3 py-1 rounded-full bg-white/5 border border-white/15 text-[11px] text-slate-300 font-medium">
          <span class="w-2 h-2 rounded-full bg-amber-400 animate-ping"></span>
          <span>បើកសម្ពោធ៖ ថ្ងៃទី ០៩ វិច្ឆិកា</span>
        </div>

        <!-- Music Toggle Button (Persistent & Looping) -->
        <button
          @click="emit('toggle-music')"
          type="button"
          class="flex items-center gap-1.5 px-3 py-1.5 rounded-full backdrop-blur-md border transition-all duration-300 cursor-pointer text-xs font-semibold"
          :class="isMusicPlaying
            ? 'bg-amber-400/20 border-amber-400 text-amber-300 shadow-[0_0_15px_rgba(251,191,36,0.5)]'
            : 'bg-white/10 border-white/20 text-slate-300 hover:bg-white/20'"
          :title="isMusicPlaying ? 'បិទសម្លេង' : 'ចាក់បទភ្លេង'"
        >
          <span class="material-symbols-outlined text-base" :class="{ 'animate-spin': isMusicPlaying }">
            {{ isMusicPlaying ? 'music_note' : 'music_off' }}
          </span>
          <span class="hidden xs:inline">{{ isMusicPlaying ? 'ភ្លេងបើក' : 'ភ្លេងបិទ' }}</span>
        </button>

        <!-- Preview Nov 9 Event (Quick Test Toggle) -->
        <button
          @click="emit('view-official')"
          type="button"
          class="flex items-center gap-1 px-3 py-1.5 rounded-full bg-gradient-to-r from-amber-400/30 to-amber-500/30 hover:from-amber-400/50 hover:to-amber-500/50 border border-amber-400/60 text-amber-300 text-xs font-bold transition-all shadow-[0_0_12px_rgba(251,191,36,0.3)] cursor-pointer"
          title="ចុចដើម្បីសាកល្បងមើលទម្រង់ថ្ងៃទី៩"
        >
          <span class="material-symbols-outlined text-sm">visibility</span>
          <span class="hidden sm:inline">មើលទម្រង់ថ្ងៃទី៩</span>
        </button>
      </div>
    </header>

    <!-- ================= MAIN CONTENT WRAPPER ================= -->
    <div class="relative z-10 w-full flex flex-col items-center">

      <!-- ================= SECTION 1: ARTBOARD 1 (INVITATION STEEL CARD) ================= -->
      <section class="min-h-screen w-full flex flex-col items-center justify-center px-4 py-20 sm:py-24 relative">
        <!-- Center Card Container (Matches Mockup Artboard 1) -->
        <div class="relative w-full max-w-[430px] sm:max-w-[480px] md:max-w-[540px] aspect-[9/14] sm:aspect-[9/13.5] rounded-2xl overflow-hidden shadow-[0_0_60px_rgba(0,0,0,0.9),0_0_30px_rgba(255,255,255,0.15)] flex flex-col justify-between p-6 sm:p-8 transition-transform duration-500 hover:scale-[1.01]">
          
          <!-- Inner Background Video / Atmospheric Backdrop inside card -->
          <div class="absolute inset-0 z-0">
            <video
              autoplay
              loop
              muted
              playsinline
              class="w-full h-full object-cover opacity-60 filter brightness-[0.7] contrast-[1.15]"
            >
              <source src="/bg-video.mp4" type="video/mp4" />
            </video>
            <!-- Vignette & Dark Clouds Inside Frame -->
            <div class="absolute inset-0 bg-gradient-to-b from-black/85 via-black/40 to-black/90"></div>
            <div class="absolute inset-0 bg-[radial-gradient(circle_at_50%_40%,_transparent_30%,_rgba(0,0,0,0.85)_100%)]"></div>
          </div>

          <!-- Chrome Metallic Outer Frame Bezel with Rivets -->
          <div class="absolute inset-0 pointer-events-none rounded-2xl border-4 sm:border-[6px] border-slate-300/80 shadow-[inset_0_0_20px_rgba(255,255,255,0.3),0_0_30px_rgba(255,255,255,0.2)]">
            <!-- Corner Rivet 1: Top-Left -->
            <div class="absolute top-2 left-2 w-3.5 h-3.5 sm:w-4 sm:h-4 rounded-full bg-gradient-to-br from-white via-slate-300 to-slate-600 shadow-[0_1px_3px_rgba(0,0,0,0.8)] border border-slate-400 flex items-center justify-center">
              <div class="w-1.5 h-1.5 rounded-full bg-slate-700"></div>
            </div>
            <div class="absolute top-2 left-7 sm:left-8 w-2.5 h-2.5 rounded-full bg-gradient-to-br from-white via-slate-300 to-slate-600 shadow-[0_1px_2px_rgba(0,0,0,0.8)] border border-slate-400"></div>

            <!-- Corner Rivet 2: Top-Right -->
            <div class="absolute top-2 right-2 w-3.5 h-3.5 sm:w-4 sm:h-4 rounded-full bg-gradient-to-br from-white via-slate-300 to-slate-600 shadow-[0_1px_3px_rgba(0,0,0,0.8)] border border-slate-400 flex items-center justify-center">
              <div class="w-1.5 h-1.5 rounded-full bg-slate-700"></div>
            </div>
            <div class="absolute top-2 right-7 sm:right-8 w-2.5 h-2.5 rounded-full bg-gradient-to-br from-white via-slate-300 to-slate-600 shadow-[0_1px_2px_rgba(0,0,0,0.8)] border border-slate-400"></div>

            <!-- Corner Rivet 3: Bottom-Left -->
            <div class="absolute bottom-2 left-2 w-3.5 h-3.5 sm:w-4 sm:h-4 rounded-full bg-gradient-to-br from-white via-slate-300 to-slate-600 shadow-[0_1px_3px_rgba(0,0,0,0.8)] border border-slate-400 flex items-center justify-center">
              <div class="w-1.5 h-1.5 rounded-full bg-slate-700"></div>
            </div>

            <!-- Corner Rivet 4: Bottom-Right -->
            <div class="absolute bottom-2 right-2 w-3.5 h-3.5 sm:w-4 sm:h-4 rounded-full bg-gradient-to-br from-white via-slate-300 to-slate-600 shadow-[0_1px_3px_rgba(0,0,0,0.8)] border border-slate-400 flex items-center justify-center">
              <div class="w-1.5 h-1.5 rounded-full bg-slate-700"></div>
            </div>
          </div>

          <!-- Top Section: Glowing Logo & Monogram -->
          <div class="relative z-10 flex flex-col items-center text-center pt-2 sm:pt-4">
            <!-- Large Chrome Glowing Monogram ($L) -->
            <div class="relative group cursor-pointer" @click="scrollToCountdown">
              <div class="absolute -inset-4 bg-gradient-to-r from-white/20 via-slate-200/40 to-white/20 blur-xl opacity-70 group-hover:opacity-100 transition-opacity"></div>
              
              <!-- SL Icon / Emblem -->
              <div class="relative flex flex-col items-center">
                <svg class="w-24 h-24 sm:w-28 sm:h-28 md:w-32 md:h-32 filter drop-shadow-[0_0_20px_rgba(255,255,255,0.85)] drop-shadow-[0_0_35px_rgba(255,255,255,0.4)]" viewBox="0 0 100 100" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <!-- Stylized Intertwined Chrome Monogram -->
                  <defs>
                    <linearGradient id="chromeGrad" x1="0%" y1="0%" x2="100%" y2="100%">
                      <stop offset="0%" stop-color="#ffffff"/>
                      <stop offset="25%" stop-color="#e2e8f0"/>
                      <stop offset="50%" stop-color="#94a3b8"/>
                      <stop offset="75%" stop-color="#f8fafc"/>
                      <stop offset="100%" stop-color="#cbd5e1"/>
                    </linearGradient>
                    <filter id="glow">
                      <feGaussianBlur stdDeviation="3" result="coloredBlur"/>
                      <feMerge>
                        <feMergeNode in="coloredBlur"/>
                        <feMergeNode in="SourceGraphic"/>
                      </feMerge>
                    </filter>
                  </defs>
                  <!-- S Monogram path -->
                  <path d="M50 8 V20 M50 80 V92" stroke="url(#chromeGrad)" stroke-width="4.5" stroke-linecap="round"/>
                  <path d="M66 32 C66 22 58 16 50 16 C40 16 34 22 34 30 C34 42 66 42 66 58 C66 68 58 74 50 74 C40 74 34 68 34 56" stroke="url(#chromeGrad)" stroke-width="6.5" stroke-linecap="round" stroke-linejoin="round"/>
                  <!-- L Monogram path intertwined -->
                  <path d="M42 24 V68 H62" stroke="url(#chromeGrad)" stroke-width="5" stroke-linecap="round" stroke-linejoin="round" opacity="0.9"/>
                </svg>

                <!-- CLUB Chrome Text -->
                <h1 class="mt-1 text-2xl sm:text-3xl md:text-4xl font-black tracking-[0.25em] text-white font-sans uppercase filter drop-shadow-[0_0_15px_rgba(255,255,255,0.8)]">
                  CLUB
                </h1>
              </div>
            </div>

            <!-- Invitation Title Khmer & English -->
            <div class="mt-4 sm:mt-6 flex flex-col items-center">
              <h2 class="text-xl sm:text-2xl md:text-3xl font-bold tracking-wide text-slate-100 font-['Kantumruy_Pro','Battambang',sans-serif] filter drop-shadow-[0_2px_10px_rgba(0,0,0,0.9)]">
                សំបុត្រគោរពអញ្ជើញ
              </h2>
              <p class="mt-1 text-xs sm:text-sm font-semibold tracking-[0.35em] text-slate-300 uppercase font-sans">
                INVITATION
              </p>
            </div>
          </div>

          <!-- Middle Section: Frosted Glass VIP Name Badge Card -->
          <div class="relative z-10 w-full my-auto py-3">
            <div class="relative w-full rounded-xl p-4 sm:p-5 bg-white/10 backdrop-blur-md border border-white/20 shadow-[0_8px_32px_rgba(0,0,0,0.5),inset_0_1px_10px_rgba(255,255,255,0.15)] flex flex-col items-center justify-center text-center overflow-hidden group">
              <!-- Shimmer light reflection effect -->
              <div class="absolute inset-0 -translate-x-full group-hover:translate-x-full bg-gradient-to-r from-transparent via-white/15 to-transparent transition-transform duration-1000"></div>

              <div class="flex items-center gap-2 mb-1.5 text-amber-300">
                <span class="material-symbols-outlined text-sm">stars</span>
                <span class="text-[11px] sm:text-xs font-bold uppercase tracking-widest">VIP HONORABLE GUEST</span>
                <span class="material-symbols-outlined text-sm">stars</span>
              </div>
              <div class="h-0.5 w-16 bg-gradient-to-r from-transparent via-white/40 to-transparent my-1"></div>
              <p class="text-sm sm:text-base font-semibold text-white/90">
                សូមគោរពអញ្ជើញ ឯកឧត្តម លោកជំទាវ លោក លោកស្រី
              </p>
              <p class="text-[11px] sm:text-xs text-slate-300/80 mt-1">
                ចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB
              </p>
            </div>
          </div>

          <!-- Bottom Section: Location Info (Bottom Left) & Scroll Cue -->
          <div class="relative z-10 flex items-end justify-between pt-2">
            <!-- Location Details (Matches Mockup exact text) -->
            <div class="flex items-start gap-2.5 max-w-[85%] cursor-pointer hover:opacity-90 transition-opacity" @click="openMaps">
              <!-- Mini Emblem -->
              <div class="w-7 h-7 sm:w-8 sm:h-8 rounded bg-black/80 p-0.5 border border-white/30 shrink-0 shadow-md">
                <img src="/logo.jpg" alt="SL" class="w-full h-full object-contain" />
              </div>

              <!-- Address Block -->
              <div class="flex flex-col text-left">
                <div class="flex items-center gap-1 text-[11px] sm:text-xs font-bold tracking-wider text-amber-300">
                  <span class="material-symbols-outlined text-xs">location_on</span>
                  <span>LOCATION</span>
                </div>
                <p class="text-[10px] sm:text-[11px] font-bold text-slate-200 tracking-wider uppercase font-sans leading-tight">
                  BATTAMBANG, STREET 203,
                </p>
                <p class="text-[9px] sm:text-[10px] font-medium text-slate-300 tracking-wide uppercase font-sans leading-tight">
                  100 METER FROM ABA BANK
                </p>
              </div>
            </div>

            <!-- Direction Arrow to Maps -->
            <button
              @click="openMaps"
              type="button"
              class="w-8 h-8 rounded-full bg-white/10 hover:bg-white/20 border border-white/30 flex items-center justify-center text-white transition-all cursor-pointer shadow-md"
              title="មើលលើ Google Maps"
            >
              <span class="material-symbols-outlined text-sm">near_me</span>
            </button>
          </div>
        </div>

        <!-- Scroll Down Cue Below Card -->
        <button
          @click="scrollToCountdown"
          type="button"
          class="mt-6 flex flex-col items-center gap-1.5 text-slate-300 hover:text-white transition-colors cursor-pointer group"
        >
          <span class="text-xs sm:text-sm font-medium tracking-widest uppercase">រំកិលចុះក្រោមមើលពេលវេលា</span>
          <div class="w-6 h-10 rounded-full border-2 border-white/30 flex items-start justify-center p-1 group-hover:border-amber-400 transition-colors">
            <div class="w-1.5 h-2.5 rounded-full bg-amber-400 animate-bounce"></div>
          </div>
        </button>
      </section>

      <!-- ================= SECTION 2: COUNTDOWN REVEAL (ARTBOARD 1 COPY) ================= -->
      <section id="teaser-countdown" class="min-h-screen w-full flex flex-col items-center justify-center px-4 py-20 relative">
        <!-- Main Card / Container -->
        <div class="w-full max-w-2xl flex flex-col items-center text-center">

          <!-- Section 2 Header Text (User requested exact Khmer phrase) -->
          <div class="inline-flex items-center gap-2 px-4 py-1.5 rounded-full bg-amber-400/10 border border-amber-400/30 text-amber-300 text-xs sm:text-sm font-semibold mb-6 shadow-[0_0_20px_rgba(251,191,36,0.2)]">
            <span class="material-symbols-outlined text-base">schedule</span>
            <span>រាប់ថយក្រោយឆ្ពោះទៅកាន់រាត្រីបើកសម្ពោធ</span>
          </div>

          <h2 class="text-2xl sm:text-4xl md:text-5xl font-black text-transparent bg-clip-text bg-gradient-to-r from-amber-200 via-white to-amber-200 tracking-wide font-['Kantumruy_Pro','Battambang',sans-serif] filter drop-shadow-[0_0_25px_rgba(255,255,255,0.4)] leading-tight px-2">
            ពួកយើងនឹងមកដល់ឆាប់ៗនេះ ជាមួយនឹង SL CLUB
          </h2>

          <p class="mt-3 text-sm sm:text-base text-slate-300 font-sans tracking-widest uppercase max-w-lg">
            WE ARE COMING SOON WITH SL CLUB BATTAMBANG
          </p>

          <!-- ================= GIANT DAYS DISPLAY (ចំនួនថ្ងៃ) ================= -->
          <div class="mt-8 sm:mt-12 relative flex flex-col items-center justify-center">
            <!-- Background Glow Halo -->
            <div class="absolute inset-0 bg-radial-[circle_at_50%_50%] from-amber-400/25 via-amber-500/10 to-transparent blur-3xl rounded-full scale-125"></div>

            <!-- Big Number Frame -->
            <div class="relative flex flex-col items-center p-6 sm:p-10 rounded-3xl bg-black/60 backdrop-blur-xl border border-amber-400/40 shadow-[0_0_50px_rgba(251,191,36,0.3),inset_0_0_30px_rgba(255,255,255,0.05)]">
              <!-- Giant Days Number -->
              <div class="flex items-baseline justify-center">
                <span class="text-7xl sm:text-9xl md:text-[11rem] font-black text-transparent bg-clip-text bg-gradient-to-b from-white via-amber-200 to-amber-500 font-mono tracking-tighter filter drop-shadow-[0_0_35px_rgba(251,191,36,0.6)] leading-none select-none">
                  {{ formattedDays }}
                </span>
              </div>

              <!-- Days Label in Khmer & English -->
              <div class="mt-3 sm:mt-4 flex items-center gap-3">
                <div class="h-px w-8 sm:w-16 bg-gradient-to-r from-transparent via-amber-400 to-transparent"></div>
                <div class="flex flex-col items-center">
                  <span class="text-xl sm:text-2xl md:text-3xl font-extrabold text-amber-300 tracking-wider font-['Kantumruy_Pro',sans-serif]">
                    {{ khmerDays }} ថ្ងៃ (ចំនួនថ្ងៃ)
                  </span>
                  <span class="text-xs sm:text-sm font-bold tracking-[0.3em] text-slate-400 uppercase font-sans">
                    DAYS REMAINING
                  </span>
                </div>
                <div class="h-px w-8 sm:w-16 bg-gradient-to-r from-transparent via-amber-400 to-transparent"></div>
              </div>
            </div>
          </div>

          <!-- ================= SUB-TIMER: EXACT HOURS, MINUTES, SECONDS ================= -->
          <div class="mt-8 sm:mt-10 w-full max-w-xl">
            <p class="text-xs sm:text-sm font-semibold text-slate-300 mb-3 flex items-center justify-center gap-1.5">
              <span class="material-symbols-outlined text-base text-amber-400">timer</span>
              <span>ពេលវេលាជាក់លាក់ដែលរាប់ថយក្រោយ</span>
            </p>

            <div class="grid grid-cols-3 gap-3 sm:gap-6">
              <!-- HOURS -->
              <div class="relative flex flex-col items-center p-3.5 sm:p-5 rounded-2xl bg-white/5 backdrop-blur-md border border-white/15 shadow-[0_4px_20px_rgba(0,0,0,0.6)] group hover:border-amber-400/50 transition-colors">
                <span class="text-3xl sm:text-5xl md:text-6xl font-black text-white font-mono tracking-tight group-hover:text-amber-300 transition-colors">
                  {{ formattedHours }}
                </span>
                <span class="mt-1 text-xs sm:text-sm font-bold text-amber-400">
                  {{ khmerHours }} ម៉ោង
                </span>
                <span class="text-[9px] sm:text-[10px] tracking-wider text-slate-400 uppercase">
                  HOURS
                </span>
              </div>

              <!-- MINUTES -->
              <div class="relative flex flex-col items-center p-3.5 sm:p-5 rounded-2xl bg-white/5 backdrop-blur-md border border-white/15 shadow-[0_4px_20px_rgba(0,0,0,0.6)] group hover:border-amber-400/50 transition-colors">
                <span class="text-3xl sm:text-5xl md:text-6xl font-black text-white font-mono tracking-tight group-hover:text-amber-300 transition-colors">
                  {{ formattedMinutes }}
                </span>
                <span class="mt-1 text-xs sm:text-sm font-bold text-amber-400">
                  {{ khmerMinutes }} នាទី
                </span>
                <span class="text-[9px] sm:text-[10px] tracking-wider text-slate-400 uppercase">
                  MINUTES
                </span>
              </div>

              <!-- SECONDS -->
              <div class="relative flex flex-col items-center p-3.5 sm:p-5 rounded-2xl bg-white/5 backdrop-blur-md border border-white/15 shadow-[0_4px_20px_rgba(0,0,0,0.6)] group hover:border-amber-400/50 transition-colors">
                <span class="text-3xl sm:text-5xl md:text-6xl font-black text-amber-400 font-mono tracking-tight animate-pulse">
                  {{ formattedSeconds }}
                </span>
                <span class="mt-1 text-xs sm:text-sm font-bold text-amber-400">
                  {{ khmerSeconds }} វិនាទី
                </span>
                <span class="text-[9px] sm:text-[10px] tracking-wider text-slate-400 uppercase">
                  SECONDS
                </span>
              </div>
            </div>
          </div>

          <!-- Target Date Indicator -->
          <div class="mt-6 sm:mt-8 px-5 py-2.5 rounded-full bg-black/60 border border-white/10 text-xs sm:text-sm text-slate-300 flex items-center gap-2">
            <span class="material-symbols-outlined text-amber-400 text-base">event</span>
            <span>រាត្រីបើកសម្ពោធ៖ ថ្ងៃចន្ទ ទី ០៩ ខែ វិច្ឆិកា ឆ្នាំ ២០២៦ វេលាម៉ោង ៧:០០ យប់</span>
          </div>

          <!-- ================= ACTION & SOCIAL BUTTONS ================= -->
          <div class="mt-8 flex flex-wrap items-center justify-center gap-3 sm:gap-4 w-full">
            <!-- Google Maps Button -->
            <button
              @click="openMaps"
              type="button"
              class="flex items-center gap-2 px-5 py-3 rounded-full bg-gradient-to-r from-slate-800 to-slate-900 hover:from-slate-700 hover:to-slate-800 border border-white/20 text-white font-semibold text-xs sm:text-sm shadow-lg hover:shadow-xl hover:scale-105 active:scale-95 transition-all cursor-pointer"
            >
              <span class="material-symbols-outlined text-red-400 text-lg">location_on</span>
              <span>ទីតាំងលើ Google Maps</span>
            </button>

            <!-- Telegram Channel Button -->
            <button
              @click="openTelegram"
              type="button"
              class="flex items-center gap-2 px-5 py-3 rounded-full bg-gradient-to-r from-sky-600 to-sky-700 hover:from-sky-500 hover:to-sky-600 text-white font-semibold text-xs sm:text-sm shadow-lg shadow-sky-600/30 hover:scale-105 active:scale-95 transition-all cursor-pointer"
            >
              <svg class="w-4 h-4 fill-current" viewBox="0 0 24 24">
                <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm4.64 6.8c-.15 1.58-.8 5.42-1.13 7.19-.14.75-.42 1-.68 1.03-.58.05-1.02-.38-1.58-.75-.88-.58-1.38-.94-2.23-1.5-.99-.65-.35-1.01.22-1.59.15-.15 2.71-2.48 2.76-2.69a.2.2 0 00-.05-.18c-.06-.05-.14-.03-.21-.02-.09.02-1.49.95-4.22 2.79-.4.27-.76.41-1.08.4-.36-.01-1.04-.2-1.55-.37-.63-.2-1.12-.31-1.08-.66.02-.18.27-.36.75-.55 2.92-1.27 4.86-2.11 5.83-2.51 2.78-1.16 3.35-1.36 3.73-1.36.08 0 .27.02.39.12.1.08.13.19.14.27-.01.06.01.24 0 .38z"/>
              </svg>
              <span>Telegram Channel</span>
            </button>

            <!-- Call Info -->
            <a
              href="tel:0963333333"
              class="flex items-center gap-2 px-5 py-3 rounded-full bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-black font-bold text-xs sm:text-sm shadow-lg shadow-amber-500/30 hover:scale-105 active:scale-95 transition-all"
            >
              <span class="material-symbols-outlined text-lg">call</span>
              <span>ទំនាក់ទំនងសាកសួរ</span>
            </a>
          </div>

          <!-- Bottom Footer Note -->
          <p class="mt-12 text-[11px] sm:text-xs text-slate-500 font-sans tracking-widest uppercase">
            © 2026 SL CLUB BATTAMBANG. ALL RIGHTS RESERVED.
          </p>
        </div>
      </section>

    </div>
  </div>
</template>
