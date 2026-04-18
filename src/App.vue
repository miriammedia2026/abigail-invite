<template>
  <div class="invite">
    <img :src="bg" class="bg" />

    <div class="content">

      <h1>
        <span>Chugga Chugga</span>
        <span>TWO TWO!</span>
      </h1>

      <h2>All aboard!</h2>

      <p class="sub">
        <span class="name">Abigail Woolly</span>
        <span class="turning">is turning</span>
        <span class="age">TWO!</span>
      </p>

      <p class="journey">Your journey begins here…</p>

      <div class="details">
        <div class="stop"></div>

        <div class="train-wrap">
          <img :src="train" class="train train-1" />
          <img :src="train" class="train train-2" />
          <div class="train-smoke"></div>
        </div>

        <!-- 🚉 -->
        <a
          :href="calendarLink"
          class="detail-box"
          :class="{ show: visibleCards >= 1 }"
        >
          <div class="label">DEPARTURE</div>
          <div class="value">April 26, 2026 at 11AM</div>
          <div class="sub-value">Tap to add to calendar</div>
        </a>

        <!-- 🎟 -->
        <div class="stop"></div>

        <div class="detail-box rsvp" :class="{ show: visibleCards >= 2 }">
          <div class="label">BOARDING PASS</div>
          <div class="value">
            {{ success ? "Boarding Status" : "Reserve Your Seat" }}
          </div>

          <div v-if="!success">
            <input v-model="form.name" placeholder="Your Name" />

            <select v-model="form.attending">
              <option value="">Can you attend?</option>
              <option value="yes">Yes, I’ll be there</option>
              <option value="no">Sorry, can’t make it</option>
            </select>

            <select v-if="form.attending === 'yes'" v-model="form.guests">
              <option value="">Guests</option>
              <option v-for="n in 7" :key="n" :value="n">{{ n }}</option>
            </select>

            <button @click="submitForm" :disabled="loading">
              {{ loading ? "Sending..." : "Submit" }}
            </button>
          </div>

          <div v-else class="success-box">
            <template v-if="isYes">
              <div class="stamp">
                <span class="stamp-icon">🎟️</span>
                <span class="stamp-text">APPROVED</span>
              </div>

              <p>Boarding Pass Confirmed!</p>
              <p>See you at the party!</p>
            </template>

            <template v-else-if="isNo">
              <p style="font-size: 18px">We’ll miss you!</p>
              <p>Thank you for letting us know</p>
            </template>
          </div>
        </div>

        <!-- 📍 -->
        <div class="stop"></div>
        <a
          :href="mapLink"
          class="detail-box"
          :class="{ show: visibleCards >= 3 }"
        >
          <div class="label">DESTINATION</div>
          <div class="value">15681 Wildflower Lane</div>
          <div class="sub-value">Kenefic, OK 74748</div>
        </a>

        <!-- 🎁 -->
        <div class="stop"></div>
        <div class="detail-box gifts" :class="{ show: visibleCards >= 4 }">
          <div class="label">GIFT STATION</div>
          <div class="gift-title">Optional gifts for Abigail</div>

          <div class="links">
            <a :href="registryLink" target="_blank">
              <div class="link-main">🧸 Toy Station</div>
              <div class="link-sub">Amazon Registry</div>
            </a>

            <a :href="paypalLink" target="_blank">
              <div class="link-main">📦 Parcel Station</div>
              <div class="link-sub">PayPal</div>
            </a>

            <a :href="cashappLink" target="_blank">
              <div class="link-main">💌 Envelope Station</div>
              <div class="link-sub">Cash App</div>
            </a>
          </div>
        </div>
      </div>
    </div>

    <div class="smoke">
      <span v-for="n in 6" :key="n"></span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, computed } from "vue";
import train from "./assets/images/train.png";
import bg from "./assets/images/background-main.jpg";

/* FORM */
const form = ref({
  name: "",
  attending: "",
  guests: "",
});

const success = ref(false);
const loading = ref(false);
const attendingResult = ref(null);

async function submitForm() {
  if (!form.value.name || !form.value.attending) {
    alert("Please fill out all fields");
    return;
  }

  if (form.value.attending === "yes" && !form.value.guests) {
    alert("Please select number of guests");
    return;
  }

  try {
    loading.value = true;

    const response = await fetch(
      "https://script.google.com/macros/s/AKfycbwFYcM_U70WsbSIMBLuCS-8JyoZCZczsnjY0BIO9H5ILuHr7gXP_Pm2YlE4rQJglD8A/exec",
      {
        method: "POST",
        body: JSON.stringify(form.value),
      }
    );

    if (response.ok) {
      attendingResult.value = form.value.attending;
      success.value = true;
    } else {
      alert("Something went wrong. Please try again.");
    }
  } catch (error) {
    console.error(error);
    alert("Network error. Try again.");
  } finally {
    loading.value = false;
  }
}

/* STATE */
const visibleCards = ref(0);

/* ANIMATION */
onMounted(() => {
  const duration = 6500;
  const totalCards = 4;
  const start = performance.now();

  function animate(now) {
    const raw = Math.min((now - start) / duration, 1);
    const progress = 1 - Math.pow(1 - raw, 2);

    const newIndex = Math.min(
      Math.floor(progress * totalCards) + 1,
      totalCards
    );

    if (newIndex !== visibleCards.value) {
      visibleCards.value = newIndex;
      nextTick(() => triggerBounce(newIndex));
    }

    if (progress < 1) requestAnimationFrame(animate);
  }

  requestAnimationFrame(animate);
});

function triggerBounce(index) {
  const cards = document.querySelectorAll(".detail-box");
  const el = cards[index - 1];
  if (!el) return;

  el.classList.add("bounce");
  setTimeout(() => el.classList.remove("bounce"), 400);
}

/* UPDATED ADDRESS */
const address = "15681 Wildflower Lane, Kenefic, OK 74748";

/* MAP */
const mapLink = /iPhone|iPad|Mac/i.test(navigator.userAgent)
  ? `http://maps.apple.com/?q=${encodeURIComponent(address)}`
  : `https://maps.google.com?q=${encodeURIComponent(address)}`;

/* GIFTS */
const registryLink =
  "https://www.amazon.com/registries/gl/guest-view/1KBPQF8ME7P2A";
const paypalLink = "https://www.paypal.com/pool/9nMCZ75mix";
const cashappLink = "https://cash.app/f/POOL?id=zipwbb2e";

/* CALENDAR */
const isApple = /iPhone|iPad/i.test(navigator.userAgent);

/* 11AM CST */
const start = "20260426T160000Z";
const end = "20260426T180000Z";

const googleCalendarLink = computed(
  () =>
    `https://www.google.com/calendar/render?action=TEMPLATE&text=${encodeURIComponent(
      "Abigail Turns TWO! 🚂"
    )}&dates=${start}/${end}&location=${encodeURIComponent(
      address
    )}&details=${encodeURIComponent(
      "Chugga Chugga TWO TWO! Come celebrate."
    )}`
);

const appleCalendarFile =
  "https://miriammedia2026.github.io/abigail-invite/abigail-party.ics";

const calendarLink = computed(() =>
  isApple ? appleCalendarFile : googleCalendarLink.value
);

/* RSVP STATES */
const isYes = computed(
  () => success.value && attendingResult.value === "yes"
);

const isNo = computed(
  () => success.value && attendingResult.value === "no"
);
</script>

<style>
/* =========================================================
   🌍 RESET + BASE
========================================================= */
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  overflow-y: auto;
  overflow-x: hidden;
  background: transparent;
}

body {
  overscroll-behavior: none;
  font-family:
    "Fredoka",
    -apple-system,
    BlinkMacSystemFont,
    sans-serif;
}

/* =========================================================
   🖼 BACKGROUND
========================================================= */
.bg {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: -1;
}

/* =========================================================
   📦 LAYOUT
========================================================= */
.invite {
  height: 100dvh;
  width: 100vw;

  display: flex;
  justify-content: center;
  align-items: flex-start;

  text-align: center;
  position: relative;
  padding: 20px 0 40px;
}

/* =========================================================
   📄 CONTENT (HERO UNCHANGED)
========================================================= */
.content {
  max-width: 360px;
  width: 92%;
  margin: 0 auto; /* 👈 ensures true horizontal centering */
  padding-top: 23px;
  animation: fadeIn 1s ease;

  display: flex;
  flex-direction: column;
  align-items: center; /* 👈 centers EVERYTHING inside */
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* =========================================================
   🔤 TYPOGRAPHY (UNCHANGED)
========================================================= */
h1 {
  font-family: "Fredoka", sans-serif;
  color: #6aa89c;
  font-size: clamp(45px, 8vw, 46px);
  font-weight: 700;
  margin-bottom: 18px;
}

h1 span {
  display: block;
}

h1 span:first-child {
  margin-bottom: 35px;
  letter-spacing: 2px;
}

h1 span:last-child {
  color: #f48ca6;
  text-shadow: 0 4px 14px rgba(244, 140, 166, 0.4);
  letter-spacing: 2px;
  font-size: 1.1em;
}

h2 {
  font-family: "Great Vibes", cursive;
  font-size: 32px;
  color: #6aa89c;
  text-shadow: none;
  padding-bottom: 5px;
}

.sub {
  font-size: 26px;
  color: #e07a94;
  font-weight: 600;

  display: flex;
  flex-direction: column;
  align-items: center; /* 👈 locks perfect center */
  text-align: center;

  gap: 2px; /* 👈 tighter spacing */
}

.journey {
  font-size: 14px;
  font-weight: 500;
  color: #b8aeb2;
  margin: 6px 0 16px;
}

.name {
  font-family: "Great Vibes", cursive;
  font-size: 45px;
  font-weight: 800;
  color: #2f2f2f;
  margin-bottom: 4px;

  text-shadow:
    0 1px 0 rgba(255, 255, 255, 0.6),
    0 3px 6px rgba(0, 0, 0, 0.12),
    0 8px 20px rgba(0, 0, 0, 0.15);
}

.turning {
  font-size: 18px;
  color: #888;
}

.age {
  font-size: 30px;
  font-weight: 800;
  color: #f48ca6;
  letter-spacing: 2px;
  margin-top: 2px;

  text-shadow: 0 4px 12px rgba(244, 140, 166, 0.35);
}

/* =========================================================
   🚉 DETAILS SECTION
========================================================= */
.details {
  margin-top: 60px;
  padding-top: 140px;
  padding-bottom: 60px;

  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 22px;

  position: relative;
}

.details::before {
  content: "";
  position: absolute;
  left: 50%;
  top: 4px;
  bottom: 4px;
  width: 1px;
  background: linear-gradient(to bottom, #6aa89c, #f48ca6);
  transform: translateX(-50%);
  opacity: 0.15;
}

.stop {
  width: 8px;
  height: 8px;
  background: #6aa89c;
  border-radius: 50%;
  opacity: 0.4;
  margin: 0 auto -4px;
}

/* =========================================================
   🎟️ TICKET CARDS
========================================================= */
.detail-box,
.detail-box * {
  text-decoration: none !important;
}

.detail-box {
  position: relative;

  background: white;
  border-radius: 14px;
  padding: 18px;
  margin: 6px;

  border: 2px dashed #f48ca6;

  box-shadow: 0 5px 0 rgba(0, 0, 0, 0.12);
  cursor: pointer;
  border-bottom-width: 3px; /* 👈 thicker bottom edge */
  opacity: 0;
  transform: translateY(30px) scale(0.95);
  transition: all 0.5s ease;
  width: 92%;
  max-width: 320px;
}

.detail-box.show {
  opacity: 1;
  transform: translateY(0) scale(1);
}

/* Ticket cutouts */
.detail-box::before,
.detail-box::after {
  content: "";
  position: absolute;
  width: 18px;
  height: 18px;
  background: #f8f8f8;
  border-radius: 50%;
  top: 50%;
  transform: translateY(-50%);
}

.detail-box::before {
  left: -7px;
}

.detail-box::after {
  right: -9px;
}
.detail-box:active {
  transform: translateY(3px) scale(0.98);

  /* 👇 reduces shadow = pressed effect */
  box-shadow: 0 2px 0 rgba(0, 0, 0, 0.1);
}

/* =========================================================
   🎨 TICKET COLORS
========================================================= */
.detail-box:nth-of-type(2) {
  border-color: #6aa89c;
  background: #f3fbf9;
}

.detail-box:nth-of-type(6) {
  border-color: #f48ca6;
  background: #fff4f7;
}

.detail-box.gifts {
  border-color: #f4c542; /* warm gold */
  background: #fff8e6; /* soft creamy yellow */
}

.rsvp {
  background: linear-gradient(135deg, #fff6f9, #fdecef);
  border: 2px dashed #f48ca6;
  box-shadow: 0 6px 20px rgba(244, 140, 166, 0.15);
}

.rsvp input,
.rsvp select {
  width: 100%;
  padding: 16px;
  margin-top: 12px;

  border-radius: 12px;

  border: 2px solid rgba(255, 255, 255, 0.7);
  background: rgba(255, 255, 255, 0.95);

  font-size: 17px;
  font-weight: 500;

  color: #333;

  box-sizing: border-box;

  /* ✨ soft depth */
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.05);
}

.rsvp button {
  width: 100%;
  margin-top: 14px;
  padding: 14px;

  border-radius: 12px;
  border: none;

  background: #6aa89c; /* 👈 teal */
  color: white;

  font-size: 16px;
  font-weight: bold;

  cursor: pointer;

  transition: all 0.2s ease;
}

/* ✨ hover + press */
.rsvp button:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(106, 168, 156, 0.4);
}

.rsvp button:active {
  transform: translateY(2px);
  box-shadow: 0 2px 4px rgba(106, 168, 156, 0.3);
}

.rsvp input::placeholder {
  color: #999;
}

/* 🎟 ticket text tone */
.rsvp .label {
  color: #b57a8e;
  font-size: 14px;
}

.rsvp .value {
  color: #a65c73;
  font-size: 28px;
  margin-bottom: 6px;
}
.rsvp p {
  font-weight: bold;
  font-size: 16px;
  color: #a65c73;
}
/* 🎟 STAMP ANIMATION */
.success-box {
  position: relative;
  padding-top: 10px;
}
.success-box p {
  color: #888;
}
/* 🎟 REAL STAMP LOOK */
.stamp {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  width: 110px;
  height: 110px;

  margin: 10px auto;

  border: 3px dashed #222;
  border-radius: 50%;

  background: rgba(255, 255, 255, 0.3);

  transform: rotate(-6deg);
  opacity: 0.85;

  animation: stampIn 0.5s ease-out;

  box-shadow:
    0 2px 6px rgba(0, 0, 0, 0.2),
    inset 0 0 8px rgba(0, 0, 0, 0.15);
}

.stamp-icon {
  font-size: 36px;
  opacity: 0.85;
  filter: blur(0.2px);
  margin-bottom: 2px;
}

.stamp-text {
  font-size: 14px;
  font-weight: 900;
  letter-spacing: 2px;
  text-align: center;

  color: #222;

  text-shadow:
    0 0 1px rgba(0, 0, 0, 0.6),
    0 1px 2px rgba(0, 0, 0, 0.3);

  opacity: 0.9;
}
/* =========================================================
   👀 READABILITY BOOST
========================================================= */
.label {
  font-size: 15px;
  letter-spacing: 1.2px;
  color: #555;
  font-weight: 700;
}

.value {
  font-size: 26px;
  font-weight: 800;
  color: #1f1f1f;
  line-height: 1.3;
}

.sub-value {
  font-size: 17px;
  color: #555;
  margin-top: 4px;
}

/* =========================================================
   🎁 GIFT LINKS
========================================================= */
.links {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  margin-top: 10px;
}

.links a {
  width: 80%;
  max-width: 280px;
  margin: 0 auto;

  background: white;
  border-radius: 14px;
  padding: 18px;

  border: 2px solid #d9aa2f;

  color: #333;
  text-decoration: none;

  font-size: 17px;

  cursor: pointer; /* 👈 important */

  box-shadow: 0 1px 0 #f2cd6e; /* 👈 "button edge" */

  transition: all 0.2s ease;
}
/* Hover (desktop) */
.links a:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 0 #d9aa2f;
  background: #fff6d9;
}

/* Press (mobile + click) */
.links a:active {
  transform: translateY(2px);
  box-shadow: 0 1px 0 #d9aa2f;
}

.link-main {
  font-size: 19px;
  font-weight: 700;
}

.link-sub {
  font-size: 15px;
  color: #666;
  margin-top: 4px;
}

/* =========================================================
   🚂 TRAIN (UNCHANGED)
========================================================= */
.train-wrap {
  position: absolute;
  top: -400px;
  left: 0;
  width: 100%;
  height: 70px;
  pointer-events: none;
  z-index: 10;
}

.train-wrap .train {
  position: absolute;
  width: 550px;
  right: -600px;
  animation: moveTrainAcross 10s linear infinite;
}

@keyframes moveTrainAcross {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-290vw);
  }
}

/* =========================================================
   ☁️ SMOKE (UNCHANGED)
========================================================= */
.smoke span {
  position: absolute;
  bottom: -40px;
  width: 28px;
  height: 20px;
  background: rgba(180, 180, 180, 0.25);
  border-radius: 60% 40% 50% 50%;
  filter: blur(6px);
  animation: floatUp 13s ease-in-out infinite;
}

@keyframes floatUp {
  0% {
    opacity: 0;
    transform: translateY(0);
  }
  15% {
    opacity: 0.3;
  }
  100% {
    opacity: 0;
    transform: translateY(-110vh);
  }
}
</style>
