<template>
  <div class="h-[720px] w-[1280px] relative bg-contain bg-top bg-[url('/img/slot-bg.png')]">
    <div class="">
      <!-- <img src="/img/bg_slot.png" class="object-cover absolute" /> -->
      <div class="absolute top-[23%] left-[15%] h-[24.2rem] w-[55rem] top bg-gradient-to-b from-[#121257] via-white to-[#121257]"></div>

      <div ref="wrapperRef" class="absolute top-[23%] left-[16.5%] h-[24.2rem] overflow-hidden w-fit">
        <div v-for="box in numLen" :key="box" class="box border-r last-of-type:border-none border-black">
          <div>0</div>
          <div v-for="n in 9" :key="n">{{n}}</div>
          <div>0</div>
          <div v-for="n in 9" :key="n">{{n}}</div>
          <div>0</div>
          <div v-for="n in 9" :key="n">{{n}}</div>
          <div class="">0</div>
        </div>
      </div>
      <!-- <div class="absolute top-[23%] left-[15%] h-[24.2rem] w-[55rem] top bg-gradient-to-b from-black/90 via-black/0 to-black/90"></div> -->
      <img src="/img/slot_frame.png" class="absolute" alt="">

      <div class="group absolute w-32 bottom-[8%] left-[13%] cursor-pointer" @click="handleReset()">
        <img src="/img/btn_rest.png" class="group-active:hidden" alt="">
        <img src="/img/btn_rest_down.png" class="hidden group-active:block" alt="">
      </div>
      <button class="group absolute w-32 bottom-[7.5%] left-[39%] cursor-pointer" @click="switchOpenType" :disabled="numIndex !== 0 || isRolling">
        <div class="" v-if="isSingle">
          <img src="/img/btn_auto_bl.png" class="group-active:hidden" alt="">
          <img src="/img/btn_auto_down_bl.png" class="hidden group-active:block" alt="">
        </div>
        <div class="" v-else>
          <img src="/img/btn_auto_y.png" class="group-active:hidden" alt="">
          <img src="/img/btn_auto_down_y.png" class="hidden group-active:block" alt="">
        </div>
      </button>

      <div class="group absolute w-40 bottom-[6%] right-[16%] cursor-pointer" @click="handleSpinClicked()">
        <img src="/img/btn_spin.png" class="group-active:hidden" alt="">
        <img src="/img/btn_spin_down.png" class="hidden group-active:block" alt="">
      </div>
    </div>

  </div>
</template>
<script lang="ts">
import { computed, defineComponent,onMounted,ref,watchEffect } from "@vue/runtime-core";
import lotterys from './assets/lotterys.json'

export default defineComponent({
  setup(props) {
    const isRolling = ref(false)
    const wrapperRef = ref<HTMLDivElement>()
    const numLen = 7 // 開獎號碼長度
    const groupLen = 4 // 總組數
    const getRandomNum = () => Math.floor(Math.random() * Math.pow(10, numLen)).toString().padEnd(numLen, "0")
    const count = ref(0) // 當前開獎組
    const numIndex = ref(0) // 第N個數字
    const isSingle = ref(false)
    const isBlur = ref(false)

    // 製作開獎號碼組
    // const numList = ref<string[]>(Array(groupLen).fill("").map(t=> getRandomNum()))
    const numList = ref<string[]>(lotterys)
    const r = getComputedStyle(document.querySelector(':root') as Element);
    const numH = r.getPropertyValue("--num-h")
    const numDiff = r.getPropertyValue("--num-diff")

    onMounted(() => {
      const ramdomNum = getRandomNum()
      wrapperRef.value?.querySelectorAll(".box").forEach((box, i) => {
        const _box = box as HTMLDivElement
        _box.style.transform = `translateY(calc( -1 * (${numH} * ${ramdomNum.charAt(i)} + ${numH} * 10 + ${numDiff})))`
      })
    })

    const switchOpenType = () => {
     isSingle.value = !isSingle.value
     if(isBlur.value) {
       count.value++
     }
     isBlur.value = false
    }

    const handleStart = () => {
      if(!wrapperRef.value) return
      if(isBlur.value) {
        ++count.value
      }
      numIndex.value = 0
      isBlur.value = true
      wrapperRef.value?.querySelectorAll(".box").forEach(box => {
        const _box = box as HTMLDivElement
        _box.classList.remove("stop")
        _box.classList.add('start')
      })
      isRolling.value = true
    }
    const handleStop = () => {
       wrapperRef.value?.querySelectorAll(".box").forEach((box, i) => {
        const _box = box as HTMLDivElement
        _box.classList.remove("start")
        _box.classList.add('stopping')
        _box.style.setProperty("--target-num", numList.value[count.value]?.charAt(i) || Math.floor(Math.random() * 10).toString())
        _box.classList.replace("stopping", "stop")
      })
      isRolling.value = false
    }
    const handleSingleStart = () => {
      if(!wrapperRef.value) return
      if(isBlur.value && numIndex.value === 0) {
        ++count.value
      }
      isBlur.value = true
      const _box = wrapperRef.value?.querySelectorAll(".box")[numIndex.value] as HTMLDivElement
      _box.classList.remove("stop")
      _box.classList.add('start')
      isRolling.value = true
    }
    const handleSingleStop = () => {
      if(!wrapperRef.value) return
      const _box = wrapperRef.value?.querySelectorAll(".box")[numIndex.value] as HTMLDivElement
      _box.classList.remove("start")
      _box.classList.add('stopping')
      _box.style.setProperty("--target-num", numList.value[count.value]?.charAt(numIndex.value) || Math.floor(Math.random() * 10).toString())
      _box.classList.replace("stopping", "stop")
      isRolling.value = false

      if(numIndex.value + 1 >= numLen) {
        numIndex.value = 0

      }else {
        ++numIndex.value
      }
    }


    const handleReset = () => {
      numIndex.value = 0
      wrapperRef.value?.querySelectorAll(".box").forEach((box, i) => {
        const _box = box as HTMLDivElement
        _box.style.setProperty("--target-num", '0')
        _box.classList.add("stop")
      })
    }

    const handleSpinClicked = () => {
      if(isRolling.value) {
        if(isSingle.value) {
          handleSingleStop()
        }else {
          handleStop()
        }
      }else {
       if(isSingle.value) {
          handleSingleStart()
        }else {
          handleStart()
        }
      }
    }
    return {
      isRolling,
      handleStart,
      handleSingleStart,
      handleSingleStop,
      handleStop,
      handleReset,
      handleSpinClicked,
      switchOpenType,
      wrapperRef,
      numList,
      count,
      numLen,
      isSingle,
      groupLen,
      numIndex,
      bgImgPath: `${import.meta.env.VITE_CDN_BASE_URL}/slot-bg.png`
    }
  }
})
</script>
<style lang="scss">
:root {
  font-size: 16px;
  --target-num: 2;
  --num-h: 24.2rem;
  --num-w: 7.3rem;
  --num-size: 10rem;
  --num-diff: -5rem;
}

@keyframes rolling {
  0% {
    transform: translateY(calc(-100% + var(--num-h)));
  }
  100% {
    transform: translateY(0%);
  }
}

@keyframes stop {
  0% {
    transform: translateY(calc(-100% + var(--num-h) * 10 - var(--num-h) * var(--target-num)));
  }

  100% {
    transform: translateY(calc((-1) * (var(--num-h) * var(--target-num) + var(--num-h) * 10 + var(--num-diff))));
  }
  /* translateY(calc( -1 * (${numH} * ${ramdomNum.charAt(i)} + ${numH} * 10 + ${numDiff}))) */
}

body {
  font-family:'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif
}



 .box {
      /* animation: rolling 2s ease-in; */
      display: inline-block;
      transform-origin: bottom;
      text-align: center;
      width: min-content;
      position: relative;
      animation-name: rolling;
      animation-duration: 0s;
      animation-iteration-count: infinite;
      animation-timing-function: linear;
      &.start {
        animation-duration: 1s;
      }
      &.stopping {
        animation-duration: 1s;
      }
      &.stop {
        animation-name: stop;
        animation-duration: calc(0.02s * (10 - var(--target-num)));
        animation-iteration-count: 1;
        animation-fill-mode: forwards;
        animation-timing-function: ease-out;
      }

      > div {
        width: var(--num-w);
        height: var(--num-h);
        font-size: var(--num-size);
        /* border-right: 1px solid rgba(#fff,0.2); */
      }
    }

.num {
  font-size: 1.5rem;
  font-weight: bold;
  color:gray;
  width: fit-content;
  margin-top: 1rem;
  > div {
    padding: 0.7rem 1rem;
    border-bottom: 1px solid #ccc;
    &.active {
      color:chocolate;
    }
  }
}

.footer {
  display: flex;
  column-gap: 0.5em;
  justify-content: center;
}

.btn {
  $bgcolor: chocolate;
  background-color: $bgcolor;
  border: none;
  padding: 0.7em 1.5em;
  color: #fff;
  font-size: 1.2em;
  cursor: pointer;
  border-radius: 0.3em;
  font-family:Verdana, Geneva, Tahoma, sans-serif;
  &:hover {
    background-color: darken($bgcolor, 10%);
  }
  &.secondary {
    $bgcolor: cadetblue;
    background-color: $bgcolor;
    &:hover {
      background-color: darken($bgcolor, 10%);
    }
  }
  &:disabled {
    cursor: not-allowed;
    background-color: gray;
    color: #555;
  }
  &.auto-btn {
    background-color: rgb(147, 8, 147);
    &.off, &:disabled {
      background-color: rgb(104, 104, 104);
    }
  }
}


</style>
