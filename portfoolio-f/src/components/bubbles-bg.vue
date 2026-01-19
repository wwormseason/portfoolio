<template>
  <div>
    <div
      v-for="bubble in bubbles"
      :id="`bubble-${bubble.bubbleCount}`"
      class="bubble"
      :key="bubble"
      @click="bubble.pop = true"
    ></div>
  </div>
</template>

<script setup>
import { onMounted, reactive, onUpdated, toRaw } from 'vue'
const bubbles = reactive([])
const spriteSets = {
  0: [
    [0, 0],
    [-1, 0],
    [-2, 0],
  ],
  1: [
    [0, -1],
    [-1, -1],
    [-2, -1],
  ],
  2: [
    [0, -2],
    [-1, -2],
    [-2, -2],
  ],
  3: [
    [0, -3],
    [-1, -3],
    [-2, -3],
  ],
  4: [
    [0, -4],
    [-1, -4],
    [-2, -4],
  ],
}
let bubbleCount = 0
let newBubbleMade = false
let bubbleCounter = 0
let frameCount = 0

onMounted(() => {
  newBubble()
  newBubble()
  newBubble()
})

onUpdated(() => {
  startBubbles()
  frameCount++
})

function newBubble() {
  if (bubbles.length < 3) {
    bubbleSettings()
  } else if (bubbles.length > 3 && !newBubbleMade && bubbles.length < 10) {
    bubbleSettings()
    newBubbleMade = true
  }
}

function bubbleSettings() {
  let n = Math.floor(Math.random() * 5)
  let bubblePosX = Math.floor(Math.random() * window.innerWidth - 32)
  let bubblePosY = Math.floor(window.innerHeight + 32)
  bubbles.push({
    bubbleCount: bubbleCount,
    variant: spriteSets[n],
    bubblePosX: bubblePosX,
    bubblePosY: bubblePosY,
    nextPosX: bubblePosX,
    nextPosY: bubblePosY,
    bubbleSpeed: 3,
    pop: false,
    popFrame: 0,
    popped: false,
    moved: false,
  })
  bubbleCount += 1
}

function startBubbles() {
  bubbles.forEach((bubble) => {
    if (frameCount % 2 === 0) {
      const thisBubble = toRaw(bubble)
      window.requestAnimationFrame(onAnimationFrame)

      let lastFrameTimeStamp
      function onAnimationFrame(timestamp) {
        if (thisBubble) {
          if (!lastFrameTimeStamp) {
            lastFrameTimeStamp = timestamp
          }
          if (timestamp - lastFrameTimeStamp > 100) {
            lastFrameTimeStamp = timestamp
            move()
          }
          window.requestAnimationFrame(onAnimationFrame)
        }
      }

      function setBubble() {
        let thisBubbleHTML = document.getElementById('bubble-' + thisBubble.bubbleCount)
        if (thisBubbleHTML) {
          if (thisBubble.bubblePosY <= 64) {
            thisBubble.pop = true
          }
          if (thisBubble.popped) {
            let index = bubbles.indexOf(thisBubble)
            bubbles.splice(index, 1)
            thisBubble.popped = false
          }
          if (thisBubble.pop) {
            thisBubble.popFrame += 1
            if (thisBubble.popFrame > 2) {
              thisBubble.pop = false
              thisBubble.popped = true
              thisBubble.popFrame = 0
            }
          }
          thisBubbleHTML.style.backgroundPosition = `${thisBubble.variant[thisBubble.popFrame][0] * 64}px ${thisBubble.variant[thisBubble.popFrame][1] * 64}px`
          thisBubbleHTML.style.left = `${thisBubble.bubblePosX - 32}px`
          thisBubbleHTML.style.top = `${thisBubble.bubblePosY - 32}px`
        }
      }

      function idle() {
        setBubble()
        if (Math.floor(Math.random() * 10) === 0 && !thisBubble.moved) {
          thisBubble.nextPosY -= 64
          let s = Math.floor(Math.random() * 2)
          if (s < 1) {
            thisBubble.nextPosX += 64
          } else if (s >= 1) {
            thisBubble.nextPosX -= 64
          }
          move()
        }
      }

      function move() {
        if (newBubbleMade) {
          if (bubbleCounter < 20) {
            bubbleCounter += 1
          } else {
            newBubbleMade = false
          }
        }
        const diffX = thisBubble.bubblePosX - thisBubble.nextPosX
        const diffY = thisBubble.bubblePosY - thisBubble.nextPosY
        const distance = Math.sqrt(diffX ** 2 + diffY ** 2)
        if (distance < thisBubble.bubbleSpeed || distance < 48) {
          idle()
          return
        }
        thisBubble.bubblePosX -= (diffX / distance) * thisBubble.bubbleSpeed
        thisBubble.bubblePosY -= (diffY / distance) * thisBubble.bubbleSpeed
        thisBubble.bubblePosX = Math.min(
          Math.max(32, thisBubble.bubblePosX),
          window.innerWidth - 32,
        )
        thisBubble.bubblePosY = Math.min(
          Math.max(32, thisBubble.bubblePosY),
          window.innerHeight - 32,
        )
        bubbleCreation()
        setBubble()
        idle()
      }

      function bubbleCreation() {
        if (Math.floor(Math.random() * 100) === 0) {
          newBubble()
        }
      }
    }
  })
}
</script>

<style scoped>
.bubble {
  width: 64px;
  height: 64px;
  position: fixed;
  image-rendering: pixelated;
  z-index: 33550335;
  background-image: url('../assets/bubbleSpriteSheet.gif');
}
</style>
