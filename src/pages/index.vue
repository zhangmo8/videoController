<script setup lang="ts">
import type { CSSProperties } from 'vue'

defineOptions({
  name: 'IndexPage',
})

interface VideoType {
  id: string
  url: string
  start: number
  timeRange: number
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
  },
  {
    id: '2',
    url: 'https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4',
    start: 4,
    timeRange: 5,
  },
]

const endingTime: ComputedRef<number> = computed(() => {
  return Math.max(...videos.map((v) => {
    return v.start + v.timeRange
  }))
})

function getRandomColor() {
  const letters = '0123456789ABCDEF'
  let color = '#'
  for (let i = 0; i < 6; i++)
    color += letters[Math.floor(Math.random() * 16)]

  return color
}

function generateStyle(v: VideoType): CSSProperties {
  const atom = 400 / endingTime.value
  const offsetWidth = atom * v.start
  const width = v.timeRange * atom

  return {
    background: getRandomColor(),
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

function onPlay() {
  timer.value = setTimeout(() => {
    currentTime.value += 1
  }, 1000)
}

function onPause() {
  clearTimeout(timer.value)
  timer.value = null
}

function setRefs(r: Element | ComponentPublicInstance | null) {
  r && videoRefs.value.push(r as HTMLVideoElement)
}
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
      <div absolute h-full w-4px bg-teal />
      <div v-for="item in videos" :key="item.id" m-t-10px h-5px rd-5px :style="generateStyle(item)" />
    </div>

    <div m-t-20px cursor-pointer btn @click="onToggle">
      {{ isPlay ? '暂停' : '播放' }}
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
