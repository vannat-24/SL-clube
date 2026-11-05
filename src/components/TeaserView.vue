<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  targetDate: {
    type: Date,
    default: () => new Date('2026-11-09T19:00:00')
  }
})

const emit = defineEmits(['view-official'])

// Video ref & state
const videoRef = ref(null)

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

// =========================================================================
// ROLE SYSTEM: ADMIN vs GUEST
// =========================================================================
const isAdmin = ref(false)
const isAdminLoginModalOpen = ref(false)
const adminPinInput = ref('')
const adminPinError = ref(false)

// Modals for Admin
const isNameModalOpen = ref(false)
const isGuestListModalOpen = ref(false)
const inputGuestName = ref('')
const copiedSuccess = ref(false)
const copiedMessageSuccess = ref(false)

// Guest Name (Read from URL ?to=... or ?name=...)
const guestName = ref('')
const invitedGuests = ref([])

function checkRoleAndParams() {
  if (typeof window === 'undefined') return
  try {
    const params = new URLSearchParams(window.location.search)

    // 1. Check if user is in Admin role
    const roleParam = params.get('role') || params.get('admin')
    if (roleParam === 'admin' || roleParam === 'true' || roleParam === '1') {
      isAdmin.value = true
      localStorage.setItem('sl_club_admin_session', 'true')
    } else if (localStorage.getItem('sl_club_admin_session') === 'true') {
      isAdmin.value = true
    } else {
      isAdmin.value = false
    }

    // 2. Check Guest Name for VIP invitation
    const nameParam = params.get('to') || params.get('name') || params.get('guest')
    if (nameParam && nameParam.trim()) {
      guestName.value = nameParam.trim()
      document.title = `SL CLUB - សំបុត្រគោរពអញ្ជើញ ${guestName.value}`
    }

    // 3. Load saved guest list for Admin
    const saved = localStorage.getItem('sl_club_invited_guests')
    if (saved) {
      invitedGuests.value = JSON.parse(saved)
    }
  } catch (err) {
    console.warn("Could not check role or read URL params:", err)
  }
}

// Admin PIN Authentication (Default PIN: 2026)
function openAdminLogin() {
  adminPinInput.value = ''
  adminPinError.value = false
  isAdminLoginModalOpen.value = true
}

function verifyAdminPin() {
  if (adminPinInput.value === '2026' || adminPinInput.value === '8888') {
    isAdmin.value = true
    localStorage.setItem('sl_club_admin_session', 'true')
    isAdminLoginModalOpen.value = false
    openNameModal()
  } else {
    adminPinError.value = true
  }
}

function exitAdmin() {
  isAdmin.value = false
  localStorage.removeItem('sl_club_admin_session')
}

// Admin Guest Link Creation
function openNameModal() {
  inputGuestName.value = ''
  copiedSuccess.value = false
  copiedMessageSuccess.value = false
  isNameModalOpen.value = true
}

function closeNameModal() {
  isNameModalOpen.value = false
}

// Computed clean link for guest without raw spaces (100% blue clickable on all apps)
const generatedGuestUrl = computed(() => {
  if (typeof window === 'undefined') return ''
  const base = window.location.origin + window.location.pathname
  const trimmed = inputGuestName.value.trim()
  if (!trimmed) return base
  return `${base}?to=${encodeURIComponent(trimmed)}`
})

// Full message formatted for Message/SMS/Telegram with banner link on own line
const fullInvitationMessage = computed(() => {
  const name = inputGuestName.value.trim() || 'ភ្ញៀវកិត្តិយស'
  const link = generatedGuestUrl.value
  return `🌟 សំបុត្រគោរពអញ្ជើញ VIP GALA NIGHT 🌟\n\nសូមគោរពអញ្ជើញ៖ ${name}\nចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB Battambang\nកាលបរិច្ឆេទ៖ ថ្ងៃចន្ទ ទី ០៩ ខែ វិច្ឆិកា ឆ្នាំ ២០២៦ វេលាម៉ោង ៧:០០ យប់\n\n👉 សូមចុចលើ Link ខាងក្រោមដើម្បីបើកមើលសំបុត្រអញ្ជើញ៖\n${link}`
})

function saveGuestToHistory(name) {
  if (!name) return
  const exists = invitedGuests.value.find(g => g.name.toLowerCase() === name.toLowerCase())
  if (!exists) {
    invitedGuests.value.unshift({
      id: Date.now(),
      name,
      url: generatedGuestUrl.value,
      createdAt: new Date().toLocaleDateString('km-KH')
    })
    localStorage.setItem('sl_club_invited_guests', JSON.stringify(invitedGuests.value))
  }
}

function deleteGuestFromHistory(id) {
  invitedGuests.value = invitedGuests.value.filter(g => g.id !== id)
  localStorage.setItem('sl_club_invited_guests', JSON.stringify(invitedGuests.value))
}

// -------------------------------------------------------------
// ជម្រើសទី ១: ផ្ញើតាម TELEGRAM (Telegram Share)
// -------------------------------------------------------------
function sendViaTelegram(customName, customUrl) {
  const targetUrl = customUrl || generatedGuestUrl.value
  const targetName = customName || inputGuestName.value.trim() || 'ភ្ញៀវកិត្តិយស'
  saveGuestToHistory(targetName)

  const url = encodeURIComponent(targetUrl)
  const text = encodeURIComponent(`🌟 សំបុត្រគោរពអញ្ជើញ VIP GALA NIGHT 🌟\n\nសូមគោរពអញ្ជើញ៖ ${targetName}\nចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB Battambang\n\n👉 សូមចុចលើ Link ខាងក្រោមដើម្បីបើកមើលសំបុត្រអញ្ជើញ៖`)
  window.open(`https://t.me/share/url?url=${url}&text=${text}`, '_blank')
}

// -------------------------------------------------------------
// ជម្រើសទី ២: ផ្ញើតាម MESSAGE (Messenger / SMS / Direct Message)
// -------------------------------------------------------------
function sendViaMessage(customName, customUrl) {
  const targetUrl = customUrl || generatedGuestUrl.value
  const targetName = customName || inputGuestName.value.trim() || 'ភ្ញៀវកិត្តិយស'
  saveGuestToHistory(targetName)

  // A. If Mobile has Web Share API (Directly shares to Messenger, SMS, WhatsApp)
  if (navigator.share) {
    navigator.share({
      title: `SL CLUB - សំបុត្រគោរពអញ្ជើញ ${targetName}`,
      text: `សូមគោរពអញ្ជើញ ${targetName} ចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB VIP GALA ថ្ងៃទី ០៩ វិច្ឆិកា ២០២៦\n\n👉 ចុចទីនេះដើម្បីបើកមើលសំបុត្រអញ្ជើញ៖`,
      url: targetUrl
    }).catch(() => {})
  } else {
    // B. Fallback to SMS Direct Protocol on mobile / desktop
    const smsBody = encodeURIComponent(`🌟 សំបុត្រគោរពអញ្ជើញ VIP GALA NIGHT 🌟\nសូមគោរពអញ្ជើញ៖ ${targetName}\nចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB\n\n👉 ចុចមើលសំបុត្រអញ្ជើញ៖\n${targetUrl}`)
    window.open(`sms:?&body=${smsBody}`, '_blank')
  }
}

function copyFullMessage(customMessage) {
  const text = customMessage || fullInvitationMessage.value
  saveGuestToHistory(inputGuestName.value.trim())

  navigator.clipboard.writeText(text).then(() => {
    copiedMessageSuccess.value = true
    setTimeout(() => { copiedMessageSuccess.value = false }, 2500)
  }).catch(() => {
    const input = document.createElement('textarea')
    input.value = text
    document.body.appendChild(input)
    input.select()
    document.execCommand('copy')
    document.body.removeChild(input)
    copiedMessageSuccess.value = true
    setTimeout(() => { copiedMessageSuccess.value = false }, 2500)
  })
}

function copyGuestLink(customUrl) {
  const url = customUrl || generatedGuestUrl.value
  if (!url) return
  saveGuestToHistory(inputGuestName.value.trim())

  navigator.clipboard.writeText(url).then(() => {
    copiedSuccess.value = true
    setTimeout(() => { copiedSuccess.value = false }, 2500)
  }).catch(() => {
    const input = document.createElement('input')
    input.value = url
    document.body.appendChild(input)
    input.select()
    document.execCommand('copy')
    document.body.removeChild(input)
    copiedSuccess.value = true
    setTimeout(() => { copiedSuccess.value = false }, 2500)
  })
}

function scrollToCountdown() {
  const el = document.getElementById('teaser-countdown')
  if (el) {
    el.scrollIntoView({ behavior: 'smooth' })
  }
}

function scrollToGallery() {
  const el = document.getElementById('teaser-gallery')
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

// =========================================================================
// CLUB INTERIOR LUXURY GALLERY (5 PHOTOS)
// =========================================================================
const galleryImages = [
  {
    id: 1,
    src: '/gallery/club-1.jpg',
    title: 'ឆាកតន្ត្រី និងប្រព័ន្ធភ្លើង 3D Hologram Stage',
    sub: 'Futuristic Main Arena & 3D Dynamic Lighting System',
    tag: '3D STAGE & SOUND'
  },
  {
    id: 2,
    src: '/gallery/club-2.jpg',
    title: 'ទិដ្ឋភាពពីជាន់លើ VIP Skybox Mezzanine',
    sub: 'Elevated VIP Skybox & Mezzanine Overview',
    tag: 'VIP SKYBOX'
  },
  {
    id: 3,
    src: '/gallery/club-3.jpg',
    title: 'ឆាក DJ & អេក្រង់ LED Hexagon យក្ស',
    sub: 'Center DJ Stage & Hexagonal LED Wall',
    tag: 'HEXAGON DJ BOOTH'
  },
  {
    id: 4,
    src: '/gallery/club-4.jpg',
    title: 'តំបន់កៅអីសាឡុង VIP Lounge ឯកជន',
    sub: 'Exclusive VIP Lounge & Luxury Sofa Seating',
    tag: 'VIP LUXURY LOUNGE'
  },
  {
    id: 5,
    src: '/gallery/club-5.jpg',
    title: 'បារទំនើប & តំបន់រាំកម្សាន្ត Dance Floor',
    sub: 'High-Table Cocktail & Immersive Dance Floor',
    tag: 'BAR & DANCE FLOOR'
  }
]

const selectedGalleryIndex = ref(null)
const isLightboxOpen = ref(false)

function openLightbox(index) {
  selectedGalleryIndex.value = index
  isLightboxOpen.value = true
  if (typeof document !== 'undefined') {
    document.body.style.overflow = 'hidden'
  }
}

function closeLightbox() {
  isLightboxOpen.value = false
  selectedGalleryIndex.value = null
  if (typeof document !== 'undefined') {
    document.body.style.overflow = ''
  }
}

function prevGalleryImage() {
  if (selectedGalleryIndex.value === null) return
  selectedGalleryIndex.value = (selectedGalleryIndex.value - 1 + galleryImages.length) % galleryImages.length
}

function nextGalleryImage() {
  if (selectedGalleryIndex.value === null) return
  selectedGalleryIndex.value = (selectedGalleryIndex.value + 1) % galleryImages.length
}

function handleKeyDown(e) {
  if (!isLightboxOpen.value) return
  if (e.key === 'Escape') closeLightbox()
  if (e.key === 'ArrowLeft') prevGalleryImage()
  if (e.key === 'ArrowRight') nextGalleryImage()
}

onMounted(() => {
  checkRoleAndParams()
  updateCountdown()
  timerId = setInterval(updateCountdown, 1000)
  if (typeof window !== 'undefined') {
    window.addEventListener('keydown', handleKeyDown)
  }
})

onUnmounted(() => {
  if (timerId) clearInterval(timerId)
  if (typeof window !== 'undefined') {
    window.removeEventListener('keydown', handleKeyDown)
    if (typeof document !== 'undefined') {
      document.body.style.overflow = ''
    }
  }
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
    <header class="fixed top-0 inset-x-0 z-50 flex items-center justify-between px-3 sm:px-8 py-3 bg-black/60 backdrop-blur-md border-b border-white/10">
      <!-- Brand Crest -->
      <div class="flex items-center gap-2.5 sm:gap-3">
        <div class="relative w-9 h-9 sm:w-10 sm:h-10 rounded-lg p-0.5 bg-gradient-to-b from-slate-200 via-slate-400 to-slate-700 shadow-[0_0_15px_rgba(255,255,255,0.3)]">
          <img src="/logo.jpg" alt="SL Club" class="w-full h-full object-contain rounded-md" />
        </div>
        <div class="flex flex-col">
          <span class="text-xs sm:text-sm font-black tracking-widest text-slate-100 uppercase font-sans">SL CLUB</span>
          <span class="text-[10px] text-amber-400 tracking-wider">COMING SOON</span>
        </div>
      </div>

      <!-- Right Header Actions -->
      <div class="flex items-center gap-2 sm:gap-3">
        <!-- 1. ADMIN MODE CONTROLS (Only visible if Role is Admin) -->
        <template v-if="isAdmin">
          <div class="flex items-center gap-1.5 px-2.5 py-1 rounded-full bg-amber-400/20 border border-amber-400/50 text-amber-300 text-[11px] font-bold">
            <span class="material-symbols-outlined text-sm">shield_person</span>
            <span class="hidden sm:inline">ADMIN</span>
          </div>

          <!-- Create Link Button -->
          <button
            @click="openNameModal"
            type="button"
            class="flex items-center gap-1.5 px-3 py-1.5 rounded-full bg-gradient-to-r from-amber-400 to-amber-500 text-black text-xs font-bold shadow-md hover:scale-105 active:scale-95 transition-all cursor-pointer"
          >
            <span class="material-symbols-outlined text-sm">add_circle</span>
            <span>បង្កើត Link ភ្ញៀវ</span>
          </button>

          <!-- Guest List Button -->
          <button
            @click="isGuestListModalOpen = true"
            type="button"
            class="flex items-center gap-1 px-3 py-1.5 rounded-full bg-white/10 hover:bg-white/20 border border-white/20 text-white text-xs font-semibold backdrop-blur-md transition-all cursor-pointer"
            title="មើលបញ្ជីភ្ញៀវដែលបានអញ្ជើញ"
          >
            <span class="material-symbols-outlined text-sm">list_alt</span>
            <span class="hidden sm:inline">បញ្ជីភ្ញៀវ ({{ invitedGuests.length }})</span>
          </button>

          <!-- Exit Admin Button -->
          <button
            @click="exitAdmin"
            type="button"
            class="p-1.5 rounded-full bg-red-500/20 hover:bg-red-500/40 text-red-300 border border-red-500/30 transition-all cursor-pointer"
            title="ចាកចេញពី Admin (មើលជាភ្ញៀវធម្មតា)"
          >
            <span class="material-symbols-outlined text-sm">logout</span>
          </button>
        </template>

        <!-- 2. GUEST ROLE (Clean & Prestigious - No Admin tools) -->
        <template v-else>
          <div class="flex items-center gap-1.5 px-3 py-1.5 rounded-full bg-white/5 border border-white/15 text-xs text-slate-300 font-medium">
            <span class="w-2 h-2 rounded-full bg-amber-400 animate-ping"></span>
            <span>បើកសម្ពោធ៖ ថ្ងៃទី ០៩ វិច្ឆិកា</span>
          </div>
        </template>
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
                  <defs>
                    <linearGradient id="chromeGrad" x1="0%" y1="0%" x2="100%" y2="100%">
                      <stop offset="0%" stop-color="#ffffff"/>
                      <stop offset="25%" stop-color="#e2e8f0"/>
                      <stop offset="50%" stop-color="#94a3b8"/>
                      <stop offset="75%" stop-color="#f8fafc"/>
                      <stop offset="100%" stop-color="#cbd5e1"/>
                    </linearGradient>
                  </defs>
                  <path d="M50 8 V20 M50 80 V92" stroke="url(#chromeGrad)" stroke-width="4.5" stroke-linecap="round"/>
                  <path d="M66 32 C66 22 58 16 50 16 C40 16 34 22 34 30 C34 42 66 42 66 58 C66 68 58 74 50 74 C40 74 34 68 34 56" stroke="url(#chromeGrad)" stroke-width="6.5" stroke-linecap="round" stroke-linejoin="round"/>
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

          <!-- ================= MIDDLE SECTION: VIP GUEST INVITATION BADGE ================= -->
          <!-- NOTE: GUEST CANNOT EDIT (Read-Only). ONLY ADMIN CAN CLICK TO EDIT IF IN ADMIN MODE -->
          <div class="relative z-10 w-full my-auto py-3">
            <div
              :class="isAdmin ? 'cursor-pointer hover:border-amber-400/50' : 'cursor-default'"
              @click="isAdmin ? openNameModal() : null"
              class="relative w-full rounded-xl p-4 sm:p-5 bg-white/10 backdrop-blur-md border border-white/20 shadow-[0_8px_32px_rgba(0,0,0,0.5),inset_0_1px_10px_rgba(255,255,255,0.15)] flex flex-col items-center justify-center text-center overflow-hidden group transition-all"
            >
              <!-- Shimmer light reflection effect -->
              <div class="absolute inset-0 -translate-x-full group-hover:translate-x-full bg-gradient-to-r from-transparent via-white/15 to-transparent transition-transform duration-1000"></div>

              <!-- Top Star Crest -->
              <div class="flex items-center gap-2 mb-1.5 text-amber-300">
                <span class="material-symbols-outlined text-sm">stars</span>
                <span class="text-[11px] sm:text-xs font-bold uppercase tracking-widest">VIP HONORABLE GUEST</span>
                <span class="material-symbols-outlined text-sm">stars</span>
              </div>
              <div class="h-0.5 w-16 bg-gradient-to-r from-transparent via-white/40 to-transparent my-1"></div>

              <!-- DYNAMIC GUEST NAME (Read-Only for Guest - No permission to edit) -->
              <div v-if="guestName" class="flex flex-col items-center py-1">
                <span class="text-xs sm:text-sm text-amber-300 font-semibold tracking-wide font-['Kantumruy_Pro',sans-serif]">
                  សូមគោរពអញ្ជើញ
                </span>
                <h3 class="text-base sm:text-xl md:text-2xl font-black text-transparent bg-clip-text bg-gradient-to-r from-amber-200 via-white to-amber-300 py-0.5 filter drop-shadow-[0_0_14px_rgba(251,191,36,0.7)] font-['Kantumruy_Pro','Battambang',sans-serif] leading-tight">
                  {{ guestName }}
                </h3>
                <span class="text-[10px] sm:text-[11px] text-slate-300/90 mt-0.5">
                  ចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB
                </span>
              </div>

              <!-- DEFAULT GENERAL GUEST TEXT (When no name is in URL) -->
              <div v-else class="flex flex-col items-center">
                <p class="text-sm sm:text-base font-semibold text-white/90">
                  សូមគោរពអញ្ជើញ ឯកឧត្តម លោកជំទាវ លោក លោកស្រី
                </p>
                <p class="text-[11px] sm:text-xs text-slate-300/80 mt-1">
                  ចូលរួមពិធីបើកសម្ពោធជាផ្លូវការ SL CLUB
                </p>
              </div>

              <!-- Edit Indicator (ONLY SHOWN TO ADMIN, NEVER SHOWN TO GUEST) -->
              <div v-if="isAdmin" class="mt-2 inline-flex items-center gap-1 px-2 py-0.5 rounded-full bg-amber-400/20 border border-amber-400/40 text-[10px] text-amber-300">
                <span class="material-symbols-outlined text-xs">edit</span>
                <span>Admin៖ ចុចដើម្បីកែ</span>
              </div>
            </div>
          </div>

          <!-- Bottom Section: Location Info (Bottom Left) & Scroll Cue -->
          <div class="relative z-10 flex items-end justify-between pt-2">
            <!-- Location Details (Matches Mockup exact text) -->
            <div class="flex items-start gap-2.5 max-w-[85%] cursor-pointer hover:opacity-90 transition-opacity" @click="openMaps">
              <div class="w-7 h-7 sm:w-8 sm:h-8 rounded bg-black/80 p-0.5 border border-white/30 shrink-0 shadow-md">
                <img src="/logo.jpg" alt="SL" class="w-full h-full object-contain" />
              </div>

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
        <div class="w-full max-w-2xl flex flex-col items-center text-center">

          <!-- Section 2 Header Text -->
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
            <div class="absolute inset-0 bg-radial-[circle_at_50%_50%] from-amber-400/25 via-amber-500/10 to-transparent blur-3xl rounded-full scale-125"></div>

            <div class="relative flex flex-col items-center p-6 sm:p-10 rounded-3xl bg-black/60 backdrop-blur-xl border border-amber-400/40 shadow-[0_0_50px_rgba(251,191,36,0.3),inset_0_0_30px_rgba(255,255,255,0.05)]">
              <div class="flex items-baseline justify-center">
                <span class="text-7xl sm:text-9xl md:text-[11rem] font-black text-transparent bg-clip-text bg-gradient-to-b from-white via-amber-200 to-amber-500 font-mono tracking-tighter filter drop-shadow-[0_0_35px_rgba(251,191,36,0.6)] leading-none select-none">
                  {{ formattedDays }}
                </span>
              </div>

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

          <!-- Prompt Button to Scroll down to Gallery Section -->
          <button
            @click="scrollToGallery"
            type="button"
            class="mt-6 sm:mt-8 inline-flex items-center gap-2.5 px-6 py-3 rounded-full bg-gradient-to-r from-amber-400/20 via-amber-400/30 to-amber-400/20 hover:from-amber-400/35 hover:to-amber-400/50 border border-amber-400/60 text-amber-300 text-xs sm:text-sm font-bold shadow-[0_0_25px_rgba(251,191,36,0.25)] hover:scale-105 active:scale-95 transition-all cursor-pointer group"
          >
            <span class="material-symbols-outlined text-base text-amber-400 group-hover:rotate-12 transition-transform">photo_library</span>
            <span>ទស្សនារូបភាពទិដ្ឋភាពក្នុងក្លឹប (៥ រូបភាព)</span>
            <span class="material-symbols-outlined text-base animate-bounce">arrow_downward</span>
          </button>
        </div>
      </section>

      <!-- ================= SECTION 3: CLUB INTERIOR LUXURY GALLERY (មួយផ្នែកទៀតនៅក្រោម COUNTDOWN) ================= -->
      <section id="teaser-gallery" class="w-full flex flex-col items-center px-4 py-16 sm:py-24 relative z-10">
        <div class="w-full max-w-6xl flex flex-col items-center">

          <!-- Section 3 Header Badge -->
          <div class="inline-flex items-center gap-2 px-4 py-1.5 rounded-full bg-amber-400/10 border border-amber-400/30 text-amber-300 text-xs sm:text-sm font-semibold mb-4 shadow-[0_0_20px_rgba(251,191,36,0.2)]">
            <span class="material-symbols-outlined text-base text-amber-400">photo_camera</span>
            <span>ទិដ្ឋភាពជាក់ស្តែងក្នុងក្លឹប • EXCLUSIVE 3D PREVIEW</span>
          </div>

          <h2 class="text-2xl sm:text-4xl md:text-5xl font-black text-transparent bg-clip-text bg-gradient-to-r from-amber-200 via-white to-amber-200 tracking-wide font-['Kantumruy_Pro','Battambang',sans-serif] filter drop-shadow-[0_0_25px_rgba(255,255,255,0.4)] text-center leading-tight px-2">
            ទិដ្ឋភាពប្រណិតក្នុងក្លឹប SL CLUB
          </h2>

          <p class="mt-3 text-xs sm:text-sm md:text-base text-slate-300 font-['Kantumruy_Pro',sans-serif] text-center max-w-2xl leading-relaxed">
            រចនាបទបែបអនាគត លំដាប់ VIP កម្រិតកំពូល បំពាក់ដោយបច្ចេកវិទ្យាភ្លើងឡាស៊ែរ 3D និងប្រព័ន្ធសំឡេងកម្រិតពិភពលោក
          </p>

          <span class="mt-2 text-[11px] sm:text-xs text-amber-400/90 font-medium tracking-wide flex items-center gap-1.5">
            <span class="material-symbols-outlined text-xs">zoom_in</span>
            <span>(ចុចលើរូបនីមួយៗដើម្បីពង្រីកមើលទំហំធំពេញលេញ • Tap any photo to enlarge)</span>
          </span>

          <!-- ================= BENTO LUXURY GALLERY GRID (5 PHOTOS) ================= -->
          <div class="mt-10 w-full grid grid-cols-12 gap-4 sm:gap-6">
            <!-- PHOTO 1: Hero Large Featured (Arena & 3D Lights) -->
            <div
              @click="openLightbox(0)"
              class="col-span-12 lg:col-span-7 relative group cursor-pointer rounded-2xl overflow-hidden bg-[#12131a] border border-white/15 hover:border-amber-400/80 transition-all duration-500 shadow-[0_10px_30px_rgba(0,0,0,0.8)] hover:shadow-[0_0_35px_rgba(251,191,36,0.35)] aspect-[16/10]"
            >
              <img
                :src="galleryImages[0].src"
                :alt="galleryImages[0].title"
                class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-700 ease-out"
                loading="lazy"
              />
              <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-black/25 to-transparent"></div>

              <!-- Top Badge & Zoom Icon -->
              <div class="absolute top-3 left-3 sm:top-4 sm:left-4 z-10 flex items-center gap-2">
                <span class="px-2.5 py-1 rounded-full bg-black/80 backdrop-blur-md border border-amber-400/50 text-amber-300 text-[10px] sm:text-xs font-bold tracking-wider uppercase shadow-md">
                  {{ galleryImages[0].tag }}
                </span>
              </div>
              <div class="absolute top-3 right-3 sm:top-4 sm:right-4 z-10 w-8 h-8 sm:w-9 sm:h-9 rounded-full bg-black/70 backdrop-blur-md border border-white/30 text-white flex items-center justify-center group-hover:bg-amber-400 group-hover:text-black group-hover:scale-110 transition-all shadow-md">
                <span class="material-symbols-outlined text-base">fullscreen</span>
              </div>

              <!-- Bottom Caption -->
              <div class="absolute bottom-0 inset-x-0 p-4 sm:p-5 z-10 flex flex-col text-left">
                <h3 class="text-base sm:text-lg md:text-xl font-bold text-white group-hover:text-amber-300 transition-colors font-['Kantumruy_Pro',sans-serif] leading-snug">
                  {{ galleryImages[0].title }}
                </h3>
                <p class="text-xs text-slate-300 font-sans tracking-wide mt-0.5">
                  {{ galleryImages[0].sub }}
                </p>
              </div>
            </div>

            <!-- PHOTO 2: Skybox Mezzanine -->
            <div
              @click="openLightbox(1)"
              class="col-span-12 sm:col-span-6 lg:col-span-5 relative group cursor-pointer rounded-2xl overflow-hidden bg-[#12131a] border border-white/15 hover:border-amber-400/80 transition-all duration-500 shadow-[0_10px_30px_rgba(0,0,0,0.8)] hover:shadow-[0_0_35px_rgba(251,191,36,0.35)] aspect-[16/10]"
            >
              <img
                :src="galleryImages[1].src"
                :alt="galleryImages[1].title"
                class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-700 ease-out"
                loading="lazy"
              />
              <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-black/25 to-transparent"></div>

              <div class="absolute top-3 left-3 sm:top-4 sm:left-4 z-10 flex items-center gap-2">
                <span class="px-2.5 py-1 rounded-full bg-black/80 backdrop-blur-md border border-amber-400/50 text-amber-300 text-[10px] sm:text-xs font-bold tracking-wider uppercase shadow-md">
                  {{ galleryImages[1].tag }}
                </span>
              </div>
              <div class="absolute top-3 right-3 sm:top-4 sm:right-4 z-10 w-8 h-8 sm:w-9 sm:h-9 rounded-full bg-black/70 backdrop-blur-md border border-white/30 text-white flex items-center justify-center group-hover:bg-amber-400 group-hover:text-black group-hover:scale-110 transition-all shadow-md">
                <span class="material-symbols-outlined text-base">fullscreen</span>
              </div>

              <div class="absolute bottom-0 inset-x-0 p-4 sm:p-5 z-10 flex flex-col text-left">
                <h3 class="text-base sm:text-lg font-bold text-white group-hover:text-amber-300 transition-colors font-['Kantumruy_Pro',sans-serif] leading-snug">
                  {{ galleryImages[1].title }}
                </h3>
                <p class="text-xs text-slate-300 font-sans tracking-wide mt-0.5">
                  {{ galleryImages[1].sub }}
                </p>
              </div>
            </div>

            <!-- PHOTO 3: Center Hexagon DJ Booth -->
            <div
              @click="openLightbox(2)"
              class="col-span-12 sm:col-span-6 lg:col-span-4 relative group cursor-pointer rounded-2xl overflow-hidden bg-[#12131a] border border-white/15 hover:border-amber-400/80 transition-all duration-500 shadow-[0_10px_30px_rgba(0,0,0,0.8)] hover:shadow-[0_0_35px_rgba(251,191,36,0.35)] aspect-[4/3]"
            >
              <img
                :src="galleryImages[2].src"
                :alt="galleryImages[2].title"
                class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-700 ease-out"
                loading="lazy"
              />
              <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-black/25 to-transparent"></div>

              <div class="absolute top-3 left-3 z-10">
                <span class="px-2.5 py-1 rounded-full bg-black/80 backdrop-blur-md border border-amber-400/50 text-amber-300 text-[10px] font-bold tracking-wider uppercase shadow-md">
                  {{ galleryImages[2].tag }}
                </span>
              </div>
              <div class="absolute top-3 right-3 z-10 w-8 h-8 rounded-full bg-black/70 backdrop-blur-md border border-white/30 text-white flex items-center justify-center group-hover:bg-amber-400 group-hover:text-black group-hover:scale-110 transition-all shadow-md">
                <span class="material-symbols-outlined text-sm">fullscreen</span>
              </div>

              <div class="absolute bottom-0 inset-x-0 p-3.5 sm:p-4 z-10 flex flex-col text-left">
                <h3 class="text-sm sm:text-base font-bold text-white group-hover:text-amber-300 transition-colors font-['Kantumruy_Pro',sans-serif] leading-snug">
                  {{ galleryImages[2].title }}
                </h3>
                <p class="text-[11px] text-slate-300 font-sans tracking-wide mt-0.5">
                  {{ galleryImages[2].sub }}
                </p>
              </div>
            </div>

            <!-- PHOTO 4: VIP Lounge Sofas -->
            <div
              @click="openLightbox(3)"
              class="col-span-12 sm:col-span-6 lg:col-span-4 relative group cursor-pointer rounded-2xl overflow-hidden bg-[#12131a] border border-white/15 hover:border-amber-400/80 transition-all duration-500 shadow-[0_10px_30px_rgba(0,0,0,0.8)] hover:shadow-[0_0_35px_rgba(251,191,36,0.35)] aspect-[4/3]"
            >
              <img
                :src="galleryImages[3].src"
                :alt="galleryImages[3].title"
                class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-700 ease-out"
                loading="lazy"
              />
              <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-black/25 to-transparent"></div>

              <div class="absolute top-3 left-3 z-10">
                <span class="px-2.5 py-1 rounded-full bg-black/80 backdrop-blur-md border border-amber-400/50 text-amber-300 text-[10px] font-bold tracking-wider uppercase shadow-md">
                  {{ galleryImages[3].tag }}
                </span>
              </div>
              <div class="absolute top-3 right-3 z-10 w-8 h-8 rounded-full bg-black/70 backdrop-blur-md border border-white/30 text-white flex items-center justify-center group-hover:bg-amber-400 group-hover:text-black group-hover:scale-110 transition-all shadow-md">
                <span class="material-symbols-outlined text-sm">fullscreen</span>
              </div>

              <div class="absolute bottom-0 inset-x-0 p-3.5 sm:p-4 z-10 flex flex-col text-left">
                <h3 class="text-sm sm:text-base font-bold text-white group-hover:text-amber-300 transition-colors font-['Kantumruy_Pro',sans-serif] leading-snug">
                  {{ galleryImages[3].title }}
                </h3>
                <p class="text-[11px] text-slate-300 font-sans tracking-wide mt-0.5">
                  {{ galleryImages[3].sub }}
                </p>
              </div>
            </div>

            <!-- PHOTO 5: Bar & Dance Floor -->
            <div
              @click="openLightbox(4)"
              class="col-span-12 sm:col-span-6 lg:col-span-4 relative group cursor-pointer rounded-2xl overflow-hidden bg-[#12131a] border border-white/15 hover:border-amber-400/80 transition-all duration-500 shadow-[0_10px_30px_rgba(0,0,0,0.8)] hover:shadow-[0_0_35px_rgba(251,191,36,0.35)] aspect-[4/3]"
            >
              <img
                :src="galleryImages[4].src"
                :alt="galleryImages[4].title"
                class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-700 ease-out"
                loading="lazy"
              />
              <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-black/25 to-transparent"></div>

              <div class="absolute top-3 left-3 z-10">
                <span class="px-2.5 py-1 rounded-full bg-black/80 backdrop-blur-md border border-amber-400/50 text-amber-300 text-[10px] font-bold tracking-wider uppercase shadow-md">
                  {{ galleryImages[4].tag }}
                </span>
              </div>
              <div class="absolute top-3 right-3 z-10 w-8 h-8 rounded-full bg-black/70 backdrop-blur-md border border-white/30 text-white flex items-center justify-center group-hover:bg-amber-400 group-hover:text-black group-hover:scale-110 transition-all shadow-md">
                <span class="material-symbols-outlined text-sm">fullscreen</span>
              </div>

              <div class="absolute bottom-0 inset-x-0 p-3.5 sm:p-4 z-10 flex flex-col text-left">
                <h3 class="text-sm sm:text-base font-bold text-white group-hover:text-amber-300 transition-colors font-['Kantumruy_Pro',sans-serif] leading-snug">
                  {{ galleryImages[4].title }}
                </h3>
                <p class="text-[11px] text-slate-300 font-sans tracking-wide mt-0.5">
                  {{ galleryImages[4].sub }}
                </p>
              </div>
            </div>
          </div>

          <!-- ================= ACTION & SOCIAL BUTTONS ================= -->
          <div class="mt-14 sm:mt-16 flex flex-wrap items-center justify-center gap-3 sm:gap-4 w-full">
            <button
              @click="openMaps"
              type="button"
              class="flex items-center gap-2 px-5 py-3 rounded-full bg-gradient-to-r from-slate-800 to-slate-900 hover:from-slate-700 hover:to-slate-800 border border-white/20 text-white font-semibold text-xs sm:text-sm shadow-lg hover:shadow-xl hover:scale-105 active:scale-95 transition-all cursor-pointer"
            >
              <span class="material-symbols-outlined text-red-400 text-lg">location_on</span>
              <span>ទីតាំងលើ Google Maps</span>
            </button>

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

            <a
              href="tel:0963333333"
              class="flex items-center gap-2 px-5 py-3 rounded-full bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-black font-bold text-xs sm:text-sm shadow-lg shadow-amber-500/30 hover:scale-105 active:scale-95 transition-all"
            >
              <span class="material-symbols-outlined text-lg">call</span>
              <span>ទំនាក់ទំនងសាកសួរ</span>
            </a>
          </div>

          <!-- Bottom Footer Note & Discreet Admin Access Link -->
          <div class="mt-12 flex flex-col items-center gap-2">
            <p class="text-[11px] sm:text-xs text-slate-500 font-sans tracking-widest uppercase">
              © 2026 SL CLUB BATTAMBANG. ALL RIGHTS RESERVED.
            </p>
            <!-- Discreet Admin Login Button at bottom -->
            <button
              v-if="!isAdmin"
              @click="openAdminLogin"
              type="button"
              class="text-[10px] text-slate-600 hover:text-amber-400 transition-colors flex items-center gap-1 cursor-pointer"
            >
              <span class="material-symbols-outlined text-xs">lock</span>
              <span>Admin Portal</span>
            </button>
          </div>
        </div>
      </section>

    </div>

    <!-- ================= ADMIN LINK CREATOR MODAL ================= -->
    <Transition name="fade">
      <div
        v-if="isAdmin && isNameModalOpen"
        class="fixed inset-0 z-[100] flex items-center justify-center p-4 bg-black/80 backdrop-blur-md"
        @click.self="closeNameModal"
      >
        <div class="relative w-full max-w-lg rounded-2xl bg-[#12131a] border border-amber-400/40 p-6 sm:p-8 shadow-[0_0_50px_rgba(251,191,36,0.3)] flex flex-col gap-5 text-left">
          <!-- Close Button -->
          <button
            @click="closeNameModal"
            type="button"
            class="absolute top-4 right-4 w-8 h-8 rounded-full bg-white/10 hover:bg-white/20 text-slate-300 hover:text-white flex items-center justify-center transition-colors cursor-pointer"
          >
            <span class="material-symbols-outlined text-sm">close</span>
          </button>

          <!-- Modal Header -->
          <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-xl bg-amber-400/20 border border-amber-400/40 flex items-center justify-center text-amber-300">
              <span class="material-symbols-outlined text-2xl">person_add</span>
            </div>
            <div>
              <div class="flex items-center gap-2">
                <span class="px-2 py-0.5 rounded text-[10px] font-bold bg-amber-400/20 text-amber-300 border border-amber-400/30">ADMIN PORTAL</span>
              </div>
              <h3 class="text-lg sm:text-xl font-bold text-white font-['Kantumruy_Pro',sans-serif]">
                បង្កើតសំបុត្រអញ្ជើញផ្ញើជូនភ្ញៀវ
              </h3>
            </div>
          </div>

          <!-- Name Input -->
          <div class="flex flex-col gap-1.5">
            <label class="text-xs font-semibold text-amber-300">
              ឈ្មោះភ្ញៀវកិត្តិយស (Guest Name)
            </label>
            <input
              v-model="inputGuestName"
              type="text"
              placeholder="ឧទាហរណ៍៖ ឯកឧត្តម ជា សុខា ឬ លោក ចាន់ តារា"
              class="w-full px-4 py-3 rounded-xl bg-white/5 border border-white/20 text-white placeholder-slate-500 focus:outline-none focus:border-amber-400 transition-colors text-sm sm:text-base font-['Kantumruy_Pro',sans-serif]"
              autofocus
            />
          </div>

          <!-- Generated Link Preview -->
          <div class="flex flex-col gap-1 p-3 rounded-xl bg-black/70 border border-white/10">
            <span class="text-[11px] text-slate-400 flex items-center gap-1">
              <span class="material-symbols-outlined text-xs text-amber-400">link</span>
              <span>Link ផ្ទាល់ខ្លួនរបស់ភ្ញៀវ (ភ្ញៀវមិនមានសិទ្ធិកែឈ្មោះឡើយ)៖</span>
            </span>
            <div class="text-xs text-amber-300 font-mono break-all select-all">
              {{ generatedGuestUrl }}
            </div>
          </div>

          <!-- ================= ជម្រើសនៃការផ្ញើទាំង ២ (TELEGRAM & MESSAGE) ================= -->
          <div class="flex flex-col gap-3 pt-1">
            <span class="text-xs font-bold text-slate-300 flex items-center gap-1.5">
              <span class="material-symbols-outlined text-sm text-amber-400">send</span>
              <span>ជ្រើសរើសមធ្យោបាយផ្ញើ (មាន Banner រូបភាពទាំងពីរ)៖</span>
            </span>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
              <!-- ជម្រើសទី ១: ផ្ញើតាម TELEGRAM (Telegram Share with Banner) -->
              <button
                @click="sendViaTelegram()"
                type="button"
                class="flex items-center justify-center gap-2.5 py-3.5 px-4 rounded-xl bg-[#229ED9] hover:bg-[#1e8ec3] text-white font-bold text-xs sm:text-sm shadow-lg shadow-[#229ED9]/25 hover:scale-[1.02] active:scale-[0.98] transition-all cursor-pointer"
              >
                <svg class="w-5 h-5 fill-current" viewBox="0 0 24 24">
                  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm4.64 6.8c-.15 1.58-.8 5.42-1.13 7.19-.14.75-.42 1-.68 1.03-.58.05-1.02-.38-1.58-.75-.88-.58-1.38-.94-2.23-1.5-.99-.65-.35-1.01.22-1.59.15-.15 2.71-2.48 2.76-2.69a.2.2 0 00-.05-.18c-.06-.05-.14-.03-.21-.02-.09.02-1.49.95-4.22 2.79-.4.27-.76.41-1.08.4-.36-.01-1.04-.2-1.55-.37-.63-.2-1.12-.31-1.08-.66.02-.18.27-.36.75-.55 2.92-1.27 4.86-2.11 5.83-2.51 2.78-1.16 3.35-1.36 3.73-1.36.08 0 .27.02.39.12.1.08.13.19.14.27-.01.06.01.24 0 .38z"/>
                </svg>
                <div class="flex flex-col items-start leading-tight">
                  <span class="text-xs sm:text-sm">១. ផ្ញើតាម Telegram</span>
                  <span class="text-[10px] text-sky-100 font-normal">ភ្ជាប់ Banner ស្វ័យប្រវត្តិ</span>
                </div>
              </button>

              <!-- ជម្រើសទី ២: ផ្ញើតាម MESSAGE (Messenger, SMS, Chat) -->
              <button
                @click="sendViaMessage()"
                type="button"
                class="flex items-center justify-center gap-2.5 py-3.5 px-4 rounded-xl bg-gradient-to-r from-blue-600 via-indigo-600 to-violet-600 hover:from-blue-500 hover:to-violet-500 text-white font-bold text-xs sm:text-sm shadow-lg shadow-blue-600/25 hover:scale-[1.02] active:scale-[0.98] transition-all cursor-pointer"
              >
                <span class="material-symbols-outlined text-xl">chat</span>
                <div class="flex flex-col items-start leading-tight">
                  <span class="text-xs sm:text-sm">២. ផ្ញើតាម Message</span>
                  <span class="text-[10px] text-blue-100 font-normal">Messenger / SMS / iMessage</span>
                </div>
              </button>
            </div>

            <!-- Copy Full Message for Manual Pasting -->
            <button
              @click="copyFullMessage()"
              type="button"
              class="w-full flex items-center justify-center gap-2 py-3 px-4 rounded-xl bg-gradient-to-r from-amber-400 to-amber-500 hover:from-amber-300 hover:to-amber-400 text-black font-bold text-xs sm:text-sm shadow-md transition-all cursor-pointer"
            >
              <span class="material-symbols-outlined text-base">
                {{ copiedMessageSuccess ? 'check_circle' : 'content_copy' }}
              </span>
              <span>{{ copiedMessageSuccess ? 'បានចម្លងសារ និង Link រួចរាល់!' : 'ចម្លងសារអញ្ជើញទាំងមូល (Copy Full Message)' }}</span>
            </button>

            <!-- Quick Copy Link Only -->
            <div class="flex items-center justify-between px-1 pt-1">
              <button
                @click="copyGuestLink()"
                type="button"
                class="text-xs text-slate-400 hover:text-amber-300 flex items-center gap-1 cursor-pointer transition-colors"
              >
                <span class="material-symbols-outlined text-sm">link</span>
                <span>{{ copiedSuccess ? 'បានចម្លង Link រួច!' : 'ចម្លងតែ Link សុទ្ធ' }}</span>
              </button>

              <button
                @click="isGuestListModalOpen = true; closeNameModal()"
                type="button"
                class="text-xs text-slate-400 hover:text-white flex items-center gap-1 cursor-pointer transition-colors"
              >
                <span class="material-symbols-outlined text-sm">history</span>
                <span>មើលបញ្ជីភ្ញៀវទាំងអស់ ({{ invitedGuests.length }})</span>
              </button>
            </div>
          </div>

          <!-- Note for Admin -->
          <div class="flex items-start gap-2 p-3 rounded-xl bg-white/5 border border-white/10 text-[11px] text-slate-300 leading-relaxed">
            <span class="material-symbols-outlined text-amber-400 text-base shrink-0">verified_user</span>
            <span>
              <strong>សុវត្ថិភាពខ្ពស់៖</strong> នៅពេលភ្ញៀវចុច Link នេះ គាត់នឹងឃើញតែលិខិតអញ្ជើញដែលមានឈ្មោះរបស់គាត់ប៉ុណ្ណោះ។ <strong>ភ្ញៀវគ្មានសិទ្ធិ ឬប៊ូតុងកែប្រែឈ្មោះឡើយ!</strong>
            </span>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ================= ADMIN GUEST LIST MODAL ================= -->
    <Transition name="fade">
      <div
        v-if="isAdmin && isGuestListModalOpen"
        class="fixed inset-0 z-[100] flex items-center justify-center p-4 bg-black/80 backdrop-blur-md"
        @click.self="isGuestListModalOpen = false"
      >
        <div class="relative w-full max-w-xl max-h-[85vh] rounded-2xl bg-[#12131a] border border-amber-400/40 p-6 shadow-2xl flex flex-col text-left overflow-hidden">
          <div class="flex items-center justify-between pb-4 border-b border-white/10">
            <div class="flex items-center gap-2">
              <span class="material-symbols-outlined text-amber-400 text-2xl">groups</span>
              <h3 class="text-lg font-bold text-white">បញ្ជីភ្ញៀវដែលបានអញ្ជើញ ({{ invitedGuests.length }})</h3>
            </div>
            <button
              @click="isGuestListModalOpen = false"
              type="button"
              class="w-8 h-8 rounded-full bg-white/10 hover:bg-white/20 text-slate-300 hover:text-white flex items-center justify-center cursor-pointer"
            >
              <span class="material-symbols-outlined text-sm">close</span>
            </button>
          </div>

          <!-- Guest List Items -->
          <div class="flex-1 overflow-y-auto py-4 space-y-2.5">
            <div v-if="invitedGuests.length === 0" class="text-center py-10 text-slate-400 text-sm">
              មិនទាន់មានភ្ញៀវនៅក្នុងបញ្ជីនៅឡើយ។ សូមចុច «បង្កើត Link ភ្ញៀវ» ដើម្បីបន្ថែម។
            </div>

            <div
              v-for="guest in invitedGuests"
              :key="guest.id"
              class="p-3.5 rounded-xl bg-white/5 border border-white/10 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3 hover:border-amber-400/40 transition-colors"
            >
              <div class="flex flex-col">
                <span class="text-sm font-bold text-white">{{ guest.name }}</span>
                <span class="text-[10px] text-slate-400 font-mono truncate max-w-xs">{{ guest.url }}</span>
              </div>

              <!-- Action buttons for each guest -->
              <div class="flex items-center gap-2 shrink-0">
                <!-- Send Telegram -->
                <button
                  @click="sendViaTelegram(guest.name, guest.url)"
                  type="button"
                  class="p-2 rounded-lg bg-sky-600/30 hover:bg-sky-600 text-sky-300 hover:text-white transition-colors cursor-pointer"
                  title="ផ្ញើតាម Telegram"
                >
                  <svg class="w-4 h-4 fill-current" viewBox="0 0 24 24">
                    <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm4.64 6.8c-.15 1.58-.8 5.42-1.13 7.19-.14.75-.42 1-.68 1.03-.58.05-1.02-.38-1.58-.75-.88-.58-1.38-.94-2.23-1.5-.99-.65-.35-1.01.22-1.59.15-.15 2.71-2.48 2.76-2.69a.2.2 0 00-.05-.18c-.06-.05-.14-.03-.21-.02-.09.02-1.49.95-4.22 2.79-.4.27-.76.41-1.08.4-.36-.01-1.04-.2-1.55-.37-.63-.2-1.12-.31-1.08-.66.02-.18.27-.36.75-.55 2.92-1.27 4.86-2.11 5.83-2.51 2.78-1.16 3.35-1.36 3.73-1.36.08 0 .27.02.39.12.1.08.13.19.14.27-.01.06.01.24 0 .38z"/>
                  </svg>
                </button>

                <!-- Send Message -->
                <button
                  @click="sendViaMessage(guest.name, guest.url)"
                  type="button"
                  class="p-2 rounded-lg bg-indigo-600/30 hover:bg-indigo-600 text-indigo-300 hover:text-white transition-colors cursor-pointer"
                  title="ផ្ញើតាម Message"
                >
                  <span class="material-symbols-outlined text-base">chat</span>
                </button>

                <!-- Copy Link -->
                <button
                  @click="copyGuestLink(guest.url)"
                  type="button"
                  class="p-2 rounded-lg bg-amber-400/20 hover:bg-amber-400 text-amber-300 hover:text-black transition-colors cursor-pointer"
                  title="ចម្លង Link"
                >
                  <span class="material-symbols-outlined text-base">content_copy</span>
                </button>

                <!-- Delete -->
                <button
                  @click="deleteGuestFromHistory(guest.id)"
                  type="button"
                  class="p-2 rounded-lg bg-red-500/20 hover:bg-red-500 text-red-300 hover:text-white transition-colors cursor-pointer"
                  title="លុបចេញពីបញ្ជី"
                >
                  <span class="material-symbols-outlined text-base">delete</span>
                </button>
              </div>
            </div>
          </div>

          <!-- Bottom Button -->
          <div class="pt-3 border-t border-white/10 flex justify-end">
            <button
              @click="openNameModal(); isGuestListModalOpen = false"
              type="button"
              class="px-4 py-2 rounded-xl bg-gradient-to-r from-amber-400 to-amber-500 text-black font-bold text-xs cursor-pointer"
            >
              + បន្ថែមភ្ញៀវថ្មី
            </button>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ================= ADMIN LOGIN PIN MODAL ================= -->
    <Transition name="fade">
      <div
        v-if="isAdminLoginModalOpen"
        class="fixed inset-0 z-[100] flex items-center justify-center p-4 bg-black/85 backdrop-blur-md"
        @click.self="isAdminLoginModalOpen = false"
      >
        <div class="relative w-full max-w-sm rounded-2xl bg-[#12131a] border border-amber-400/40 p-6 shadow-2xl flex flex-col gap-4 text-center">
          <div class="w-12 h-12 rounded-full bg-amber-400/20 border border-amber-400/40 flex items-center justify-center text-amber-400 mx-auto">
            <span class="material-symbols-outlined text-2xl">admin_panel_settings</span>
          </div>

          <div>
            <h3 class="text-lg font-bold text-white">ចូលកាន់ Admin Portal</h3>
            <p class="text-xs text-slate-400 mt-1">សូមបញ្ចូលលេខកូដសម្ងាត់ Admin PIN (2026)</p>
          </div>

          <div class="flex flex-col gap-2">
            <input
              v-model="adminPinInput"
              type="password"
              placeholder="លេខកូដ PIN..."
              class="w-full text-center tracking-widest text-lg font-mono px-4 py-3 rounded-xl bg-white/5 border text-white placeholder-slate-500 focus:outline-none"
              :class="adminPinError ? 'border-red-500' : 'border-white/20 focus:border-amber-400'"
              @keyup.enter="verifyAdminPin"
              autofocus
            />
            <span v-if="adminPinError" class="text-xs text-red-400">លេខកូដមិនត្រឹមត្រូវឡើយ (PIN: 2026)</span>
          </div>

          <div class="flex items-center gap-2 pt-1">
            <button
              @click="isAdminLoginModalOpen = false"
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
    <!-- ================= FULLSCREEN LUXURY LIGHTBOX MODAL ================= -->
    <Transition name="fade">
      <div
        v-if="isLightboxOpen && selectedGalleryIndex !== null"
        class="fixed inset-0 z-[150] flex flex-col items-center justify-between p-3 sm:p-6 bg-black/95 backdrop-blur-2xl select-none"
        @click.self="closeLightbox"
      >
        <!-- Top Controls Bar -->
        <div class="w-full max-w-5xl flex items-center justify-between z-20 pt-1 pb-3 px-2 border-b border-white/10">
          <div class="flex items-center gap-2 sm:gap-3">
            <img src="/logo.jpg" alt="SL Club" class="w-8 h-8 rounded-lg object-contain border border-amber-400/40" />
            <div class="flex flex-col">
              <span class="text-xs sm:text-sm font-bold text-white tracking-widest uppercase font-sans">SL CLUB</span>
              <span class="text-[10px] text-amber-400 tracking-wider">3D INTERIOR GALLERY</span>
            </div>
          </div>

          <!-- Counter & Close -->
          <div class="flex items-center gap-3">
            <span class="px-3 py-1 rounded-full bg-white/10 border border-white/20 text-xs text-amber-300 font-mono font-bold">
              {{ selectedGalleryIndex + 1 }} / {{ galleryImages.length }}
            </span>
            <button
              @click="closeLightbox"
              type="button"
              class="w-9 h-9 sm:w-10 sm:h-10 rounded-full bg-white/10 hover:bg-white/20 border border-white/20 text-white flex items-center justify-center transition-all cursor-pointer hover:scale-110 active:scale-95"
              title="បិទ (Close - Esc)"
            >
              <span class="material-symbols-outlined text-lg">close</span>
            </button>
          </div>
        </div>

        <!-- Center Image Stage with Next/Prev Arrows -->
        <div class="relative w-full max-w-5xl flex-1 flex items-center justify-center my-auto py-2 px-1">
          <!-- Previous Button -->
          <button
            @click.stop="prevGalleryImage"
            type="button"
            class="absolute left-1 sm:left-3 z-30 w-10 h-10 sm:w-12 sm:h-12 rounded-full bg-black/70 hover:bg-amber-400 text-white hover:text-black border border-white/30 hover:border-amber-400 flex items-center justify-center transition-all cursor-pointer shadow-2xl hover:scale-110 active:scale-90"
            title="រូបមុន (Previous)"
          >
            <span class="material-symbols-outlined text-2xl sm:text-3xl">chevron_left</span>
          </button>

          <!-- Main Image -->
          <div class="relative max-h-[62vh] sm:max-h-[70vh] flex items-center justify-center">
            <img
              :src="galleryImages[selectedGalleryIndex].src"
              :alt="galleryImages[selectedGalleryIndex].title"
              class="max-h-[62vh] sm:max-h-[70vh] w-auto max-w-full object-contain rounded-2xl shadow-[0_0_80px_rgba(0,0,0,0.9),0_0_35px_rgba(251,191,36,0.25)] border border-white/20 transition-all duration-300"
            />
          </div>

          <!-- Next Button -->
          <button
            @click.stop="nextGalleryImage"
            type="button"
            class="absolute right-1 sm:right-3 z-30 w-10 h-10 sm:w-12 sm:h-12 rounded-full bg-black/70 hover:bg-amber-400 text-white hover:text-black border border-white/30 hover:border-amber-400 flex items-center justify-center transition-all cursor-pointer shadow-2xl hover:scale-110 active:scale-90"
            title="រូបបន្ទាប់ (Next)"
          >
            <span class="material-symbols-outlined text-2xl sm:text-3xl">chevron_right</span>
          </button>
        </div>

        <!-- Bottom Caption & Thumbnails Strip -->
        <div class="w-full max-w-5xl flex flex-col items-center gap-3 z-20 pb-2 px-2">
          <!-- Caption -->
          <div class="text-center px-4">
            <h4 class="text-base sm:text-lg md:text-xl font-bold text-white font-['Kantumruy_Pro',sans-serif]">
              {{ galleryImages[selectedGalleryIndex].title }}
            </h4>
            <p class="text-xs sm:text-sm text-amber-300/90 font-sans tracking-wide mt-0.5">
              {{ galleryImages[selectedGalleryIndex].sub }}
            </p>
          </div>

          <!-- Thumbnails -->
          <div class="flex items-center justify-center gap-2 sm:gap-3 overflow-x-auto py-1 max-w-full">
            <button
              v-for="(img, idx) in galleryImages"
              :key="img.id"
              @click.stop="selectedGalleryIndex = idx"
              type="button"
              class="relative w-14 sm:w-18 aspect-[16/10] rounded-lg overflow-hidden border-2 transition-all cursor-pointer shrink-0"
              :class="selectedGalleryIndex === idx ? 'border-amber-400 scale-105 shadow-[0_0_15px_rgba(251,191,36,0.8)]' : 'border-white/20 opacity-50 hover:opacity-100'"
            >
              <img :src="img.src" :alt="img.title" class="w-full h-full object-cover" />
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>
