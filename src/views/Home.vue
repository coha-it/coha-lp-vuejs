<template lang="pug">
.home
  .page-sections(:style="{'transform': getPageTransform1(), 'transform': getPageTransform2(), 'transition-duration': getPageTransitionDuration()}")
    section.page-section(v-for="(e, i) in new Array(iSections)" :key="i")
      
      transition(name='fade')
        div(v-show='i != iSection')
          | Seite nummer {{ i }}

      transition(name='fade')
        div(v-show='i == iSection')
          | hello

  .page-navigator
    .page-pointer(
      v-for="(e,i) in new Array(iSections)"
      :key="i"
      :class="{'active': i == iSection}"
      @click="naviClicked(i)"
    )
      .dot
    //- A
    //- B
    //- C
</template>

<script>
// @ is an alias to /src
import A from '@/components/SectionA.vue'
import B from '@/components/SectionB.vue'
import C from '@/components/SectionC.vue'

export default {
  name: 'Home',

  components: {
    A, B, C
  },

  data () {
    return {
      iSection: 0,
      iSections: 10,
      iLastY: null,
      iLastSpeed: 0,
      iExtraScroll: 0,
      bIsScrolling: false,

      // Touches
      fLastTouchY: null,
      fLastTouchX: null,
      fTouchSensitivity: 8,

      // Options
      iScrollSensitivity: .5,
      iTransition: 750,
      iTransitionExtra: 0,
    }
  },

  created () {
    window.addEventListener('touchmove', this.handleTouch);
    window.addEventListener('wheel', this.handleScroll);
    window.addEventListener('resize', this.handleResize);
  },

  methods: {

    naviClicked (key) {

      if (this.iSection !== key) {
        let diff = Math.abs(this.iSection - key)

        this.iTransitionExtra = diff >= 5 ? 600 : (diff * 120)
        this.bIsScrolling = true


        switch (true) {
          case this.iSection < key:
            // Down
            this.scrollDown(diff)
            break;

          case this.iSection > key:
            // Up
            this.scrollUp(diff)
            break
        }

        setTimeout(() => {
          this.stoppedScrolling()
        }, this.iTransition * 0.85 + this.iTransitionExtra);
        
      }
    },

    handleResize () {
      // Fix Viewports
      // First we get the viewport height and we multiple it by 1% to get a value for a vh unit
      // Then we set the value in the --vh custom property to the root of the document
      document.documentElement.style.setProperty('--vh', `${window.innerHeight * 0.01}px`);
    },

    handleTouch (event) {
      var iPageY = event.touches[0].pageY
      var iPageX = event.touches[0].pageX

      if (
        this.fLastTouchY &&
        !this.bIsScrolling &&
        this.fLastTouchX &&
        Math.abs(this.fLastTouchX - iPageX) < 2
      ) {
        let diff = iPageY - this.fLastTouchY
        let speed = Math.round(Math.abs(diff) / 15)

        this.iTransitionExtra = (speed - 1) * 500

        switch (true) {
          case (diff < -(this.fTouchSensitivity)):
            this.scrollDown(speed)
            this.bIsScrolling = true
            break

          case (diff > this.fTouchSensitivity):
            this.scrollUp(speed)
            this.bIsScrolling = true
            break
        }

        // Stop scrolling
        setTimeout(() => {
          this.stoppedScrolling();
        }, this.iTransition * 0.5);

      }

      this.fLastTouchY = iPageY
      this.fLastTouchX = iPageX
    },

    handleScroll (event) {

      // Get last scrolled speed
      if (this.iLastSpeed < Math.abs(event.deltaY)) {
        this.iLastSpeed = Math.abs(event.deltaY)
      }

      // Only if mouse is wheeling Scrolling
      // And if it's not Scrolling
      if (this.mouseIsWheeling(event) && !this.bIsScrolling)
      {
        this.scrollUpOrDown(event, 1);
        this.bIsScrolling = true

        // Stopp scrolling after transition (*.75)
        setTimeout(() => {
          this.stoppedScrolling();
        }, this.iTransition * 0.75);

        // Check if there is an extra-scroll
        setTimeout(() => {
          this.checkExtraScroll(event)
        }, this.iTransition * 0.9);

        // Start Animation
        setTimeout(() => {
          this.startSectionAnimation()
        }, this.iTransition + this.iTransitionExtra);
      }
    },

    startSectionAnimation () {
      console.log('Start Section Animation')
    },

    checkExtraScroll (event) {
      switch (true) {
        case this.iLastSpeed > 300 && this.iExtraScroll < 2:
          this.iExtraScroll = 2
          this.iTransitionExtra += 200
          this.scrollUpOrDown(event, 2)
          break;

        case this.iLastSpeed > 225 && this.iExtraScroll < 1:
          this.iExtraScroll = 2
          this.iTransitionExtra += 200
          this.scrollUpOrDown(event, 1)
          break;
      }
    },

    stoppedScrolling () {
      this.bIsScrolling = false
      this.iLastSpeed = 0
      this.iExtraScroll = 0
      this.iTransitionExtra = 0
      this.fLastTouchY = null
      this.fLastTouchX = null
    },

    scrollUpOrDown (event, amount) {
      let iDeltaY = event.deltaY
      let iScrollSensitivity = this.iScrollSensitivity
      switch (true) {
        case iDeltaY < -(iScrollSensitivity):
          this.scrollUp(amount)
          break;

        case iDeltaY > iScrollSensitivity:
          this.scrollDown(amount)
          break;
      }
    },

    scrollUp (amount = 1) {
      if(this.iSection - amount >= 0) {
        this.iSection -= amount
      }
    },

    scrollDown (amount = 1) {
      if(this.iSection + amount < this.iSections) {
        this.iSection += amount
      }
    },

    getPageTransform1 () {
      return 'translateY(-'+ (this.iSection * 100) +'vh)';
    },

    getPageTransform2 () {
      return 'translateY( calc(var(--vh, 1vh) * -'+ (this.iSection * 100) +'))';
    },

    getPageTransitionDuration () {
      if (this.bIsScrolling) {
        return (this.iTransition + this.iTransitionExtra) + 'ms';
      }
      return '50ms';
    },

    mouseIsWheeling (event) {
      // Devine Ys
      let iY = event.deltaY
      let iLastY = this.iLastY
      let bWheeling = Math.abs(iY) > Math.abs(iLastY)

      // Set Last Y
      this.iLastY = (iY)

      // return if Wheeling
      return bWheeling
    }
  }
}
</script>

<style lang="sass">
html, body
  margin: 0
  padding: 0
  overflow: hidden

.page-sections
  position: fixed
  top: 0
  left: 0
  transform: translateY(-0vh)
  transition-duration: 500ms
  transition-timing-function: ease-out
  transition-timing-function: cubic-bezier(0.28, 0.54, 0.55, 0.98)
  transition-timing-function: cubic-bezier(0.39, 0.58, 0.57, 1)

.page-section
  height: 100vh
  width: 100vw
  height: calc(var(--vh, 1vh) * 100)
  width: calc(var(--vw, 1vw) * 100)
  background: #d0d0d0
  display: flex
  flex-flow: column
  align-items: center
  justify-content: center
  text-align: center
  font-family: Consolas
  font-size: 16px

  // Random Colors
  $i: 1
  $bgs: #d9cce0, #b5c6d6, #b5d6cc, #cad6b5, #d6c8b5, #d3b5d6, #b4c3cc, #969ede, #96dea0, #c8de96
  @each $bg in $bgs
    &:nth-of-type(10n + #{$i})
      background-color: #{$bg}
    $i: $i + 1

// PAGE NAVIGATOR
.page-navigator 
    position: fixed;
    right: 0;
    top: 0;
    bottom: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    align-self: center;
    flex-direction: column;
    width: 50px;


.page-pointer 
    display: inline-block;
    padding: 5px 10px
    transition: 400ms;
    cursor: pointer

    &.active
      pointer-events: none
      cursor: default
      .dot
        padding: 5px
        opacity: .8

    .dot
      opacity: .3
      background: #000;
      padding: 3px;
      border-radius: 100%;



// Animations
.fade-enter-active,
.fade-leave-active 
  transition: 1s

.fade-enter,
.fade-leave-to
  opacity: 0


</style>
