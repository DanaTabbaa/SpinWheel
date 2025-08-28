<template>

  <main class="container py-4 my-5" style="background: linear-gradient(90deg, #f6fa00, #ffffff);">

    <div class="row align-content-start justify-content-around">

      <div class="col-12 text-center">
        <img     width="70"  src="https://heroes-woad.vercel.app/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Flogo-2.4c6f6315.png&w=48&q=75" />
      </div>

      <div class="col-12 col-lg-7 px-lg-5 order-0 order-lg-1">

        <div class="sticky-top">

       <div class="fs-2 text-center custom-text-container">
  <div class="welcome-text">
    Welcome To Hero's Station
  </div>
  <div v-if="winnerResult" class="winner-text">
    <span v-if="winnerResult.text != '   Game Over   '">ربحت:</span>  
    <span >{{ winnerResult.text }}</span> <span  v-if="winnerResult.text != '   Game Over   '">🎉</span>
  </div>
  <div v-else-if="isSpinning" class="spinning-text" style="color: black;">
    Spinning...
  </div>
</div>

          <ShiningDots
              :color="shiningDotsColor"
              :border-color="shiningDotsBorderColor"
              :shine-color="shiningDotsShineColor"
              :border-width="shiningDotsBorderWidth"
              :size="shiningDotsSize"
              :count="shiningDotsCount">

            <VueWheelSpinner
                ref="spinner"
                class="temp"
                :slices="slices"
                :slice-text-align="center"
                :slice-text-position="sliceTextPosition"
                :slice-font-style="sliceFontStyle"
                :winner-index="defaultWinner"
                :sounds="sounds"
                :cursor-angle="cursorAngle"
                :cursor-position="cursorPosition"
                :cursor-distance="cursorDistance"
                @spin-start="onSpinStart"
                @spin-end="onSpinEnd">

              <template #cursor>
                <img class="cursor-img" :src="cursorImage" alt="Cursor">
              </template>

              <template #default>
                <button
                    class="spin-button "
                    style="background-color:#f6fa00;color: black !important;"
                    :disabled="isSpinning || hasSpun"
                    @click="spinRandom()"
                    @mouseover="handleSpinButtonHover"
                    @mouseleave="handleSpinButtonLeave">
                  أبدأ
                </button>
              </template>

            </VueWheelSpinner>

          </ShiningDots>
<div v-if="showWinnerModal" class="modal-overlay" @click="closeWinnerModal">
  <div class="modal-content" @click.stop>
    <h2>🎉 مبروك! 🎉</h2>
    <p>{{ winnerResult.text }}</p>
    <button @click="closeWinnerModal">إغلاق</button>
  </div>
</div>
          <!-- <div>
            <button
                class="btn btn-success btn-lg rounded-pill w-100 py-4"
                :disabled="isSpinning"
                @click="spinRandom()">
                <span class="spinner-border spinner-border-sm me-2" v-if="isSpinning" role="status">
                  <span class="visually-hidden">Loading...</span>
                </span>
              Spin for Random
            </button>
          </div> -->

        </div>

      </div>

    </div>

  </main>

</template>

<script>
import VueWheelSpinner from 'vue-wheel-spinner';
import 'bootstrap/js/src/dropdown.js'

import cursorImage from './assets/cursor.svg';
import wonSound from './sounds/won.mp3';
import clickSound from './sounds/click.mp3';
import hoverSound from './sounds/hover.mp3';
import leaveSound from './sounds/leave.mp3';
import spinningSound from './sounds/spinning.mp3';
import ShiningDots from "@/components/ShiningDots.vue";

export default {
  components: {
    ShiningDots,
    VueWheelSpinner
  },
  data() {
    return {
     showWinnerModal: false,
    hasSpun: false,
      winnerResult: null,
      slices: this.createColorTextArray(11),
      sliceTextPosition: 'edge',
      sliceFontStyle: 'bold 12px Arial',
      isSpinning: false,
      defaultWinner: 0,
      sounds: {
        won: wonSound,
        spinButtonClick: clickSound,
        spinButtonHover: hoverSound,
        spinButtonLeave: leaveSound,
        spinning: spinningSound
      },
      cursorImage,
      cursorAngle: 0,
      cursorPosition: 'edge',
      cursorDistance: 0,
      shiningDotsColor: '#ffffff',
      shiningDotsShineColor: '#ffd800',
      shiningDotsBorderColor: '#1e254c',
      shiningDotsBorderWidth: 30,
      shiningDotsSize: 8,
      shiningDotsCount: 60
    };
  },
  watch: {
    slices: {
      handler() {
        this.$refs.spinner.drawWheel();
      },
      deep: true
    },
    shiningDotsBorderWidth() {
      this.$refs.spinner.drawWheel();
    },
  },
  methods: {
    getRandomColor() {
      return '#' + Math.floor(Math.random() * 16777215).toString(16);
    },
    getContrastingColor(bgColor) {
      let color = bgColor;
      if (bgColor.charAt(0) === '#') {
        color = bgColor.substring(1, 7);
      }

      const r = parseInt(color.substring(0, 2), 16);
      const g = parseInt(color.substring(2, 4), 16);
      const b = parseInt(color.substring(4, 6), 16);

      const brightness = (r * 299 + g * 587 + b * 114) / 1000;

      return brightness > 125 ? '#000000' : '#ffffff';
    },
    createColorTextArray(count) {
       const names=[
        '  غيم بولينغ   '   ,
        '  ساعة بلياردو   '   ,
        '  ساعة بينغ بونغ   '   ,
        '   pc ساعتين '   ,
        '   غيم اخطبوط '   ,
        '  ماتش فيشة   '   ,
        '   تكت سينما   '   ,
        '   تكت دانس فلور   '   ,
        '   اشتراك شهري مجاني   ',
        '   مشروب فري     ' ,
        '   خصم 25% كرانشي   ',
       '   Golden Card   '   ,
        '   ماتش كارتينغ   ',
        '   Game Over   '
      ]
      const results = [];
      const colors = [
        '#f6fa00',
        '#020202',
      ];  // Predefined colors
      for (let i = 0; i < names.length; i++) {
        let color = colors[i % colors.length]; // Alternate colors
        results.push({
          color: color,  // Alternate colors
          text: names[i],        // Convert number to string
          textColor: this.getContrastingColor(color) // Get contrasting color
        });
      }
      console.log(results);
      return results;
    },
    playAudio(audio) {
      if (audio) {
        audio.volume = 0.5
        audio.play();
      }
    },
    handleSpinButtonClick() {
      if (this.buttonClickAudio) {
        this.playAudio(this.buttonClickAudio)
      }
      this.$refs.spinner.spinWheel(this.defaultWinner);
    },
    handleSpinButtonHover() {
      if (this.buttonHoverAudio) {
        this.playAudio(this.buttonHoverAudio)
      }
    },
    handleSpinButtonLeave() {
      if (this.buttonLeaveAudio) {
        this.playAudio(this.buttonLeaveAudio)
      }
    },
    handleCursorPositionChange() {
      if (this.cursorPosition === 'center') {
        if (!this.cursorDistance) {
          this.cursorDistance = 50;
        }
      } else {
        this.cursorDistance = 0;
      }
    },
    handleAddNewSlice() {
      let randomColor = this.getRandomColor();
      this.slices.push({
        color: randomColor,
        text: 'New Slice',
        textColor: this.getContrastingColor(randomColor)
      });
    },

  closeWinnerModal() {
    this.showWinnerModal = false;
  },
    spinFor(index) {
      this.defaultWinner = index;
      this.$refs.spinner.spinWheel(index);
    },
    spinRandom() {
      const randomSlice = Math.floor(Math.random() * this.slices.length);
      this.$refs.spinner.spinWheel(randomSlice);
    },
    onSpinStart() {
      this.winnerResult = null;
      this.isSpinning = true;
    },
    onSpinEnd(winnerIndex) {
      this.isSpinning = false;
      this.winnerResult = this.slices[winnerIndex];
      if(this.winnerResult.text != '   Game Over   '){
       this.showWinnerModal = true;

      }

        // Store flag in localStorage
  localStorage.setItem('hasSpunWheel', 'true');

  // Disable the spin button permanently
  this.hasSpun = true;
    }
  },
  mounted() {
    this.buttonHoverAudio = new Audio(hoverSound);
    this.buttonLeaveAudio = new Audio(leaveSound);
    this.buttonClickAudio = new Audio(clickSound);
    const hasSpun = localStorage.getItem('hasSpunWheel');
  if (hasSpun === 'true') {
    this.hasSpun = true;
  }
  }
};
</script>

<style>
.temp p{
  padding: 2px;
}
.cursor-img {
  width: 50px;
  aspect-ratio: 1 / 1;
  filter: drop-shadow(3px 3px 2px rgba(0, 0, 0, 0.19));
}

.spin-button {
  width: 100px;
  height: 100px;
  margin: 0 auto;
  aspect-ratio: 1 / 1;
  font-size: 20px;
  cursor: pointer;
  background: #eb4d4b;
  border-radius: 50%;
  transition: all 150ms;
  border: 10px solid white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  color: white !important;
  box-shadow: inset -3px -3px 2px 2px rgba(0, 0, 0, 0.19), 3px 3px 2px 2px rgba(0, 0, 0, 0.19);
  z-index: 11;
  position: relative;
  user-select: none;

  &:hover {
    box-shadow: inset -5px -5px 2px 2px rgba(0, 0, 0, 0.19), 3px 3px 2px 2px rgba(0, 0, 0, 0.19);
  }

  &:active {
    box-shadow: inset 3px 3px 2px 2px rgba(0, 0, 0, 0.19), 3px 3px 2px 2px rgba(0, 0, 0, 0.19);
  }

  &:disabled {
    background: #ccc;
    cursor: not-allowed;
    pointer-events: none;
  }

}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  /* Black gradient background */
  background: linear-gradient(135deg, rgba(0,0,0,0.9), rgba(30,30,30,0.8));
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
}

.modal-content {
  background: #121212; /* dark background */
  color: #f6fa00; /* your bright yellow */
  padding: 2.5rem 3rem;
  border-radius: 15px;
  text-align: center;
  max-width: 350px;
  box-shadow: 0 8px 25px rgba(246, 250, 0, 0.7);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  animation: modalFadeIn 0.4s ease forwards;
}

.modal-content h2 {
  font-size: 2.4rem;
  margin-bottom: 1rem;
}

.modal-content p {
  font-size: 1.3rem;
  margin-bottom: 2rem;
}

.modal-content button {
  background-color: #f6fa00;
  border: none;
  padding: 0.75rem 1.8rem;
  border-radius: 30px;
  font-weight: 700;
  color: black;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.modal-content button:hover {
  background-color: #d4d800;
}

/* Smooth fade-in animation */
@keyframes modalFadeIn {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.custom-text-container {
  font-family: 'Poppins', 'Cairo', sans-serif;
  color: #333333;
  margin-top: 30px;
}

.welcome-text {
  font-weight: 700;
  font-size: 2.2rem;
  color: #222222;
  letter-spacing: 0.03em;
  margin-bottom: 1rem;
}

.winner-text {
  font-weight: 700;
  font-size: 1.8rem;
  color: #f6fa00; /* your yellow */
  letter-spacing: 0.04em;
  margin-top: 1.5rem;
  text-shadow: 1px 1px 3px rgba(0,0,0,0.3);
}

.spinning-text {
  font-weight: 600;
  font-size: 1.6rem;
  color: #555555;
  margin-top: 1.5rem;
  font-style: italic;
}



</style>
