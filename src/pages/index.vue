<script setup lang="ts">
import type { CSSProperties } from 'vue'

defineOptions({
  name: 'IndexPage',
})

const progressBarWidth = 400

interface VideoType {
  id: string
  url: string
  start: number
  timeRange: number
  isPlayed?: boolean
  background?: string
}

// actions
const isPlay = ref(false)
const currentTime = ref(0)

const timer = ref()

// videos
const videoRefs = ref<Array<HTMLVideoElement>>([])
const videos: VideoType[] = [
  {
    id: '1',
    url: 'https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4',
    start: 0,
    timeRange: 5,
    isPlayed: false,
    background: 'rgb(253, 209, 57)',
  },
  {
    id: '2',
    url: 'https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4',
    start: 1,
    timeRange: 5,
    isPlayed: false,
    background: 'rgb(29, 206, 113)',
  },
]

const endingTime: ComputedRef<number> = computed(() => {
  return Math.max(...videos.map((v) => {
    return v.start + v.timeRange
  }))
})

const secondToPixel = computed(() => {
  return progressBarWidth / endingTime.value
})

const progress = computed<CSSProperties>(() => {
  return {
    transform: `translateX(${secondToPixel.value * currentTime.value > progressBarWidth ? progressBarWidth - 4 : secondToPixel.value * currentTime.value}px)`,
  }
})

const isEnded: ComputedRef<boolean> = computed(() => {
  return currentTime.value >= endingTime.value
})

function generateStyle(v: VideoType): CSSProperties {
  const offsetWidth = secondToPixel.value * v.start
  const width = v.timeRange * secondToPixel.value

  return {
    width: `${width}px`,
    transform: `translateX(${offsetWidth}px)`,
  }
}

function onToggle() {
  isPlay.value = !isPlay.value

  if (isPlay.value)
    onPlay()
  else onPause()
}

function onRePlay() {
  currentTime.value = 0
  videos.forEach((v, i) => {
    v.isPlayed = false
    videoRefs.value[i].currentTime = 0
  })
  onPlay()
}

function onPlay() {
  if (currentTime.value < endingTime.value) {
    videos.forEach((video, index) => {
      const videoEnd = video.start + video.timeRange

      if (video && !video.isPlayed && currentTime.value >= video.start && currentTime.value < videoEnd) {
        video.isPlayed = true
        videoRefs.value[index].play()
      }
    })
    currentTime.value += 0.25
    timer.value = setTimeout(onPlay, 250)
  }
}

function onPause() {
  timer.value && clearTimeout(timer.value)

  let currentIndex = null

  videos.forEach((v, i) => {
    if (v.isPlayed) {
      currentIndex = i
      videoRefs.value[i].pause()
      v.isPlayed = false
    }
  })

  if (currentIndex || currentIndex === 0)
    currentTime.value = videoRefs.value[currentIndex].currentTime + videos[currentIndex].start

  timer.value = null
}

function setRefs(r: Element | ComponentPublicInstance | null) {
  r && videoRefs.value.push(r as HTMLVideoElement)
}

// TODO：All video time update will trigger this function Single case. Get the max time and use it to be currentTime
// const computedTime = useThrottleFn((event: Event, index: number) => {
//   const videoTime = (event.target as HTMLVideoElement)?.currentTime + videos[index].start
//   currentTime.value = videoTime
// }, 100)
</script>

<template>
  <div h-screen w-screen flex flex-col items-center justify-center>
    <div>
      <div flex>
        <video v-for="item in videos" :key="item.id" :ref="setRefs" controls w-300px>
          <source :src="item.url" type="video/mp4">
          您的浏览器不支持 video 标签。
        </video>
      </div>
    </div>
    <div relative box-border h-50px w-400px rd-2 bg-white>
      <div absolute left-0 h-full w-4px bg-teal transition-transform :style="progress" />
      <div v-for="item in videos" :key="item.id" m-t-10px h-5px rd-5px :style="{ ...generateStyle(item), background: item.background }" />
    </div>

    <div v-if="!isEnded" m-t-20px cursor-pointer btn @click="onToggle">
      {{ isPlay ? '暂停' : '播放' }}{{ currentTime }}
    </div>
    <div v-else m-t-20px cursor-pointer btn @click="onRePlay">
      <div />
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
