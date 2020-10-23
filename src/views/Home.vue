<template lang="pug">
.home
  .page-sections(:style="{'transform': getPageTransform1(), 'transform': getPageTransform2(), 'transition-duration': getPageTransitionDuration()}")
    section.page-section(
      v-for="(e, i) in new Array(iSections)"
      :key="i"
      v-on:mousewheel="handleScroll"
      v-on:touchmove="handleTouch"
    )

      textarea(v-on:mousewheel.prevent="preventDefault")
      
      transition(name='fade')
        div(v-show='i != iSection')
          | Flowting - Seite nummer {{ i }}

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
      iLastSpeedY: 0,
      iExtraScroll: 0,
      bIsScrolling: false,

      // Touches
      fLastTouchY: null,
      fLastTouchX: null,
      fTouchSensitivity: 8,
      iTouchSpeedSensitivity: 20, // 15

      // Options
      iScrollSensitivity: 4, // 0.5, but x-achis won't work
      iTransition: 750,
      iTransitionExtra: 0,
    }
  },

  created () {
    // window.addEventListener('touchmove', this.handleTouch);
    // window.addEventListener('mousewheel', this.handleScroll);
    window.addEventListener('resize', this.handleWindowResize);
  },

  methods: {

    preventDefault (e) {
      console.log('preventDefault')
      return e.preventDefault()
    },

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

    handleWindowResize () {
      // Fix Viewports
      // First we get the viewport height and we multiple it by 1% to get a value for a vh unit
      // Then we set the value in the --vh custom property to the root of the document
      document.documentElement.style.setProperty('--vh', `${window.innerHeight * 0.01}px`);
    },

    handleTouch (event) {
      var touches = event.touches
      var iPageY = touches[0].pageY
      var iPageX = touches[0].pageX
      var fDiffX = Math.abs(this.fLastTouchX - iPageX)

      if (
        this.fLastTouchY &&           // If there is a last Touch Y position
        this.fLastTouchX &&           // If there is a last Touch X position
        !this.bIsScrolling &&         // If it's not scrolling
        fDiffX < 2 &&                  // If its the Y-Axis not much X-Axis
        touches.length <= 1     // If only 1 Finger is Touching
      ) {
        let fDiffY = iPageY - this.fLastTouchY
        let speed = Math.round(Math.abs(fDiffY) / this.iTouchSpeedSensitivity)
        speed = speed > 0 ? speed : 1

        this.iTransitionExtra = (speed - 1) * 500

        switch (true) {
          case (fDiffY > this.fTouchSensitivity) && !this.isOnFirstSection():
            this.scrollUp(speed)
            this.bIsScrolling = true
            break

          case (fDiffY < -(this.fTouchSensitivity)) && !this.isOnLastSection():
            this.scrollDown(speed)
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
      let iSpeedY = Math.abs(event.deltaY)

      if (this.iLastSpeedY < iSpeedY) {
        this.iLastSpeedY = iSpeedY
      }

      // Only if mouse is wheeling Scrolling
      // And if it's not Scrolling
      if (
        this.mouseIsWheeling(event) && // Mouse is Wheeling
        !this.bIsScrolling &&          // Isn't Scrolling
        this.scrollUpOrDown(event, 1)  // Not on First or Last Page
      ) {
        // Scrolling
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
        case this.iLastSpeedY > 300 && this.iExtraScroll < 2:
          this.iExtraScroll = 2
          this.iTransitionExtra += 200
          this.scrollUpOrDown(event, 2)
          break;

        case this.iLastSpeedY > 225 && this.iExtraScroll < 1:
          this.iExtraScroll = 2
          this.iTransitionExtra += 200
          this.scrollUpOrDown(event, 1)
          break;
      }
    },

    stoppedScrolling () {
      this.bIsScrolling = false
      this.iLastSpeedY = 0
      this.iExtraScroll = 0
      this.iTransitionExtra = 0
      this.fLastTouchY = null
      this.fLastTouchX = null
    },

    scrollUpOrDown (event, amount) {
      let iDeltaY = event.deltaY
      let iScrollSensitivity = this.iScrollSensitivity
      switch (true) {
        case iDeltaY < -(iScrollSensitivity) && !this.isOnFirstSection():
          this.scrollUp(amount)
          return true

        case iDeltaY > iScrollSensitivity && !this.isOnLastSection():
          this.scrollDown(amount)
          return true
        
        default:
          // Can't Scroll
          return false
      }
    },

    isOnFirstSection () {
      return this.iSection === 0
    },

    isOnLastSection () {
      return this.iSection +1 === this.iSections
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
    cursor: pointer
    transform: scale(.75)

    &.active
      pointer-events: none
      cursor: default
      transform: scale(1)
      .dot
        opacity: .8

    .dot
      transition: opacity 400ms
      opacity: .3
      background: #000
      padding: 5px
      border-radius: 100%



// Animations
.fade-enter-active,
.fade-leave-active 
  transition: 1s

.fade-enter,
.fade-leave-to
  opacity: 0


</style>
