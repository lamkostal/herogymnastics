<template>
  <UContainer class="py-2 md:py-8">
    <!-- //Group Setup Buttons// -->
    <div class="flex md:justify-end gap-1">
      <UButton  color="indigo" size="xs" activeClass="bg-red-500 my-10" variant="outline" >ΩΜΟΙ</UButton>
      <UButton  color="indigo" size="xs" activeClass="bg-red-500" variant="outline">ΠΛΑΤΗ</UButton>
      <UButton  color="indigo" size="xs" activeClass="bg-red-500" variant="outline" >ΜΕΣΗ</UButton>
    </div>
    <!-- Init Buttons  -->
    <UButton icon="i-tabler-player-play" v-if="state === 'notInit'" @click="init()" class="hover:transition-all my-1"
      size="xl" block>
      Start Exercises</UButton>
    <UButton icon="i-tabler-player-play" v-if="state === 'stopped'" @click="resume()" class="hover:transition-all my-1"
      size="xl" block>
      Resume Exercises</UButton>
    <UButton icon="i-tabler-pause" :disabled="isBreak" v-if="state === 'running'" @click="pause()"
      class="hover:transition-all my-1" size="xl" block>Pause
      Exercises</UButton>

    <!-- //cards wrapper// -->
    <div id="card-wrapper" class=" my-1 md:my-8 flex justify-center md:justify-between md:gap-1  relative flex-wrap">

    <!-- //Buttons wrapper// -->
      <div class="w-full justify-between md:justify-start items-end md:items-start md:w-3/12 flex md:flex-col md:gap-1 mb-2">
      
         <!-- Duration Buttons  -->
       <div class="md:gap-1">
          <p class="text-xs md:text-lg text-black"><strong>Set Exercise Duration:</strong></p>
          <div class="flex gap-1">
            <UButton v-for="(but, index) in durButtons" variant="outline" color="gray" @click.passive="setDuration(but.time, index)" class="mt-1"
              :class="{ activeclass: index === activeIndex }" size="lg" :disabled="disabled">{{ but.time }}s</UButton>
          </div>
       </div>
          <!-- Prev-Next Buttons  -->
      <div class="md:w-full">
        <UButton  class=" mr-1 " size="lg" :disabled="isPrevDisbled || isBreak" color="indigo" icon="i-tabler-player-track-prev"
          @click="previous()">
        </UButton>
        <UButton  class=" mr-1 md:mt-1" size="lg" :disabled="isNextDisbled || isBreak" color="indigo" icon="i-tabler-player-track-next" @click="next()"
          ></UButton>
      </div>

      
      </div>
      <!-- VIDEO -->
       <div class="w-full md:w-4/12 justify-self-center justify-center items-center flex">
   
        <Transition name="video">
          <div v-show="isBreak === false" class=" overflow-hidden    ">
            <figure class="">
              <video ref="videoref" loop="" muted="" playsinline="" style="max-width:100%" data-loaded="true" >
                <source type="video/mp4" :src="sources[videoIndex].src">
              </video>
            </figure>
          </div>
        </Transition>
      <!-- COUNTER -->

      
          <div v-show="isBreak" >
            <figure>
              <Transition name="video">
                <video  preload="auto" v-show="isCounterVisible" ref="timer" muted="" playsinline="" style="max-width:100%"
                poster="https://res.cloudinary.com/lamkos/image/upload/v1734518873/hero%20gymnastics/%CE%91%CF%83%CE%BA%CE%B7%CF%83%CE%B5%CE%B9%CF%82%CE%95%CE%BD%CE%B4_g19j43.png"
                  data-loaded="true">
                  <source type="video/mp4"
                    src="https://res.cloudinary.com/lamkos/video/upload/v1734016487/hero%20gymnastics/simple_count_fmengs.mp4">
                </video>
              </Transition>
            </figure>
          </div>
       

      </div>
    
      <!-- PROGRESS BAR -->
      <div class="w-full md:w-4/12 flex flex-col md:justify-start md:gap-9 md:items-end text-black">
      <p v-show="isBreak && state!=='notInit'" class="text-sm md:text-xl text-center font-bold self-center">Take a 5 seconds break</p>
        <p v-show="state==='notInit'" class="text-sm md:text-xl text-center font-bold self-center">Begining in 5s</p>

        <p  v-show="isBreak!==true" class="text-sm md:text-xl text-center font-bold self-center ">{{ videoIndex + 1+"/"+sources.length }} - {{ " " + sources[videoIndex].title }}</p>
        <UProgress class="w-full" size="2xl" :value="time" :max="duration">
          <template #indicator="{ percent }">
            
            <div class="text-indigo-500  font-bold text-sm md:text-lg text-right">
              <Transition mode="out-in" name="global"><span v-show="percent>70" class="mr-2">Almost there!</span></Transition>
             
              <span class="text-xl" >{{Math.ceil(duration-(duration*(percent/100)))}}</span>
              <UIcon name="i-ic-baseline-timer" class="ml-1"  />
              
            </div>
          </template>
        </UProgress>

        <p v-show="isBreak!==true" class="md:text-lg md:my-2 text-indigo-500 " v-if="udjastedIndex">COMING NEXT : {{ sources[udjastedIndex].title }}</p>
        <p class="md:text-xl	md:my-2 text-indigo-500 t" v-else>This is the last exersise, hang on!</p>

      </div>

    </div>
    <!-- //Modal// -->
    <UModal v-model="isOpen">
      <div>
        <div class="p-4 justify-between flex flex-col items-center">
          <p class="text-xl">Congratulations!!</p>

          <p class="text-lg">Exercising is fun and is good for your health!</p>
          <video autoplay loop
            src="https://cdnl.iconscout.com/lottie/premium/thumb/fireworks-animation-download-in-lottie-json-gif-static-svg-file-formats--crackers-celebration-firecrackers-festival-days-animations-4164693.mp4"></video>

          <!-- <Placeholder class="h-48" /> -->
          <UButton color="indigo" icon="ic-twotone-restart-alt" @click="init()" class="mr-2 my-1" size="xl">Try again
          </UButton>
        </div>
      </div>

    </UModal>



  </UContainer>


</template>


<script setup>
let udjastedIndex = computed(() => {
  return videoIndex.value !== sources.value.length - 1 ? videoIndex.value + 1 : false
})
let initTimeout = ref(undefined)
let prevTimeout = ref(undefined)
let nextTimeout = ref(undefined)
let isPrevDisbled = ref(false)
let isNextDisbled = ref(false)
let timer = ref(null)
let isBreak = ref(true)
let isCounterVisible = ref(true)
let isOpen = ref(false)
let disabled = ref(false)
let duration = ref(10)
let state = ref('notInit')
let time = ref(0)
let interval = ref(undefined)
let videoref = ref(null)
let videoIndex = ref(0)
let durButtons = ref([{ time: 30 }, { time: 20 }, { time: 10 }])
let activeIndex = ref(2)
let sources = ref(
  [{ src: "https://res.cloudinary.com/lamkos/video/upload/v1734091805/hero%20gymnastics/1_tuullt.mp4", title: "Hands Up" },
  { src: "https://res.cloudinary.com/lamkos/video/upload/v1734091808/hero%20gymnastics/2_jqff5t.mp4", title: "Hands Down" },
  { src: "https://res.cloudinary.com/lamkos/video/upload/v1734091809/hero%20gymnastics/3_haesbd.mp4", title: "Left Leg" },
  { src: "https://res.cloudinary.com/lamkos/video/upload/v1734091805/hero%20gymnastics/4_cilmx2.mp4", title: "Right Leg" },
  ]
)


function setDuration(dur, index) {
  if (state.value === 'stopped' || state.value === 'notInit') {
    duration.value = dur
  }
  activeIndex.value = index


}
function init() {
  videoIndex.value = 0
  isCounterVisible.value = true
  videoref.value.load()
  isOpen.value = false
  clearInterval(interval.value)
  isBreak.value = true
  state.value = "running"
  timer.value.play()
  initTimeout = setTimeout(() => {
    time.value = 0
    isBreak.value = false
    videoref.value.play()
    interval.value = setInterval(() => { time.value += 0.1; }, 100)
    isCounterVisible.value = false
  }, 5000)
  // clearTimeout()
}

function resume() {
  state.value = "running"
  videoref.value.play()
  interval.value = setInterval(() => { time.value += 0.1; }, 100)
  isCounterVisible.value = false

}
function pause() {
  videoref.value.pause()
  state.value = "stopped"
  clearInterval(interval.value)
}
function previous() {
  clearTimeout(nextTimeout.value)
  clearTimeout(prevTimeout.value)
  clearTimeout(initTimeout.value)
  pause()
  if (videoIndex.value > 0) {
    videoIndex.value--
    videoref.value.load()
    time.value = 0
    isBreak.value = true
    state.value = "running"
    isCounterVisible.value = true
    timer.value.load()
    timer.value.play()
    prevTimeout = setTimeout(() => {
      resume()
      isBreak.value = false
    }, 5000)

  } else {
    pause()
  }

}

function next() {
  clearTimeout(nextTimeout.value)
  clearTimeout(prevTimeout.value)
  clearTimeout(initTimeout.value)
  pause()
  if (videoIndex.value + 1 < sources.value.length) {
    videoIndex.value++
    videoref.value.load()
    time.value = 0
    isBreak.value = true
    state.value = "running"
    isCounterVisible.value = true
    timer.value.load()
    timer.value.play()
    nextTimeout = setTimeout(() => {
      resume()
      isBreak.value = false
    }, 5000)
  } else {
    pause()
  }

}

watchEffect(() => {

  if (time.value > duration.value && videoIndex.value < sources.value.length - 1) {
    next()
  }
  if (videoIndex.value === sources.value.length - 1) {
    isNextDisbled.value = true
  } else { isNextDisbled.value = false }

  if (videoIndex.value > 0) {
    isPrevDisbled.value = false
  } else { isPrevDisbled.value = true }

  if (state.value === 'running') {
    disabled.value = true
  } else { disabled.value = false }

  if (time.value > (duration.value) && videoIndex.value === sources.value.length - 1) {
    isOpen.value = true
    state.value = 'notInit'
    videoref.value.pause()
    clearInterval(interval.value)
    time.value = 0
    disabled.value = false
  }
  // console.log(time.value)
  // console.log("interval=" + " " + interval.value)
  // console.log("isprevdis" + ' ' + isPrevDisbled.value)
  // console.log("videoIndex:" + " " + videoIndex.value)
  // console.log("state.value :" + " " + state.value)
  // console.log("nextTimeout:" + " " + nextTimeout.value)
  // console.log("prevTimeout:" + " " + prevTimeout.value)
  // console.log("initTimeout.value:" + " " + initTimeout.value)
  console.log("disabled:" + " " + disabled.value)
  console.log("state.value:" + " " + state.value)
});
// , { deep: true }
// onMounted(()=>console.log(`index:${videoIndex.value}`))



</script>

<style >

body{
  background-color: #fff;
}
.activeclass {
  background-color: #6366f1 !important;
  color: #fff !important;
  transition: all 0.3s ease;
}
.video-enter-active{
  transition: opacity 0.3s ease;
}
.video-leave-active {
  transition: opacity 0s ;
}

.video-enter-from,
.video-leave-to {
  opacity: 0;
}

.global-enter-active,
.global-leave-active {
  transition: opacity 0.4s ease;
}

.global-enter-from,
.global-leave-to {
  opacity: 0;
}


</style>