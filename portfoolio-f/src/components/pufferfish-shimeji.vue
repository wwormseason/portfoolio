<template>
  <div>
    <div
      ref="fish"
      id="fish"
      aria-hidden="true"
      @click="((puff = true), (puffStart = frameCount), (initPuff = true))"
    ></div>
  </div>
</template>

<script setup>
import { onMounted, useTemplateRef } from 'vue'

const fish = useTemplateRef('fish')
let puff = false
let initPuff = false

let reduceMotion = false
let persistPosition = true
let fishPosX = 64
let fishPosY = 64
let nextPosX = 64
let nextPosY = 64
let frameCount = 0
let idleTime = 0
let fishSpeed = 10
let direction = 'leftUn'
let puffStart
let diffX = 0
let diffY = 0
let distance = 0
const spriteSets = {
  rightUn: [
    [0, 0],
    [-1, 0],
  ],
  leftUn: [
    [0, -1],
    [-1, -1],
  ],
  rightPuffIns: [[-2, 0]],
  leftPuffIns: [[-2, -1]],
  rightPuff: [
    [-3, 0],
    [-4, 0],
  ],
  leftPuff: [
    [-3, -1],
    [-4, -1],
  ],
}

onMounted(() => {
  if (
    window.matchMedia(`(prefers-reduced-motion: reduce)`) == true ||
    window.matchMedia(`(prefers-reduced-motion:reduce)`).matches === true
  ) {
    reduceMotion = true
  }
  startFish()
})

function startFish() {
  if (reduceMotion) {
    return
  } else {
    const curS = document.currentScript
    if (curS && curS.dataset.persistPosition) {
      if (curS.dataset.persistPosition === '') {
        persistPosition = true
      } else {
        persistPosition = JSON.parse(curS.dataset.persistPosition.toLowerCase())
      }
    }

    if (persistPosition) {
      let fishStore = JSON.parse(window.localStorage.getItem('fish'))
      if (fishStore !== null) {
        fishPosX = fishStore.fishPosX
        fishPosY = fishStore.fishPosY
        frameCount = fishStore.frameCount
        idleTime = fishStore.idleTime
        fish.value.style.backgroundPosition = fish.bgPos
      }
    }
    fish.value.style.left = `${fishPosX - 32}px`
    fish.value.style.top = `${fishPosY - 32}px`

    if (persistPosition) {
      window.addEventListener('beforeunload', function (event) {
        this.window.localStorage.setItem(
          'fish',
          JSON.stringify({
            fishPosX: fishPosX,
            fishPosY: fishPosY,
            frameCount: frameCount,
            idleTime: idleTime,
            bgPos: fish.value.style.backgroundPosition,
          }),
        )
      })
    }

    window.requestAnimationFrame(onAnimationFrame)
  }

  let lastFrameTimeStamp
  function onAnimationFrame(timestamp) {
    if (fish) {
      if (!lastFrameTimeStamp) {
        lastFrameTimeStamp = timestamp
      }
      if (timestamp - lastFrameTimeStamp > 100) {
        lastFrameTimeStamp = timestamp
        frame()
      }
      window.requestAnimationFrame(onAnimationFrame)
    }
  }

  function setSprite(name, frame) {
    const sprite = spriteSets[name][frame % spriteSets[name].length]
    fish.value.style.backgroundPosition = `${sprite[0] * 64}px ${sprite[1] * 64}px`
  }

  function idle() {
    idleTime += 1
    puffTime()
    whichSprite()
    setSprite(direction, frameCount)
    if (idleTime > 10 && Math.floor(Math.random() * 50) === 0) {
      nextPosX = Math.floor(Math.random() * window.innerWidth)
      nextPosY = Math.floor(Math.random() * window.innerHeight)
    }
  }

  function frame() {
    frameCount += 1
    diffX = fishPosX - nextPosX
    diffY = fishPosY - nextPosY
    distance = Math.sqrt(diffX ** 2 + diffY ** 2)
    if (distance < fishSpeed || distance < 48) {
      idle()
      return
    }
    whichSprite()
    puffTime()
    setSprite(direction, frameCount)
    fishPosX -= (diffX / distance) * fishSpeed
    fishPosY -= (diffY / distance) * fishSpeed
    fishPosX = Math.min(Math.max(32, fishPosX), window.innerWidth - 32)
    fishPosY = Math.min(Math.max(32, fishPosY), window.innerHeight - 32)
    fish.value.style.left = `${fishPosX - 32}px`
    fish.value.style.top = `${fishPosY - 32}px`
  }

  function puffTime() {
    if (puff) {
      if (frameCount - puffStart > 20) {
        puff = false
      }
    }
  }

  function whichSprite() {
    if (initPuff) {
      direction = diffX / distance > 0.1 ? 'leftPuffIns' : 'rightPuffIns'
      direction = diffX / distance < -0.1 ? 'rightPuffIns' : 'leftPuffIns'
      initPuff = false
    } else if (puff) {
      direction = diffX / distance > 0.1 ? 'leftPuff' : 'rightPuff'
      direction = diffX / distance < -0.1 ? 'rightPuff' : 'leftPuff'
    } else {
      direction = diffX / distance > 0.1 ? 'leftUn' : 'rightUn'
      direction = diffX / distance < -0.1 ? 'rightUn' : 'leftUn'
    }
  }
}
</script>

<style scoped>
#fish {
  width: 64px;
  height: 64px;
  position: fixed;
  image-rendering: pixelated;
  z-index: 33550336;
  background-image: url('../assets/pufferfishSpriteSheet.gif');
}
</style>
