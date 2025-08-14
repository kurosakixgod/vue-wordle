<script setup lang="ts">
import { nextTick, ref, watch } from "vue"

const { word: wordProp } = defineProps<{
  word?: string
}>()

type Input = {
  isLocked: boolean
  values: string[]
  refs: HTMLInputElement[]
}

const length = 5
const word = "hello"

// const inputs = ref<Input[]>(Array(length).fill({ isLocked: true, values: [] }))
const inputs = ref<Input[]>([
  { isLocked: false, values: Array(length).fill(""), refs: Array(length).fill(null) },
  { isLocked: true, values: Array(length).fill(""), refs: Array(length).fill(null) },
  { isLocked: true, values: Array(length).fill(""), refs: Array(length).fill(null) },
])

const check = async (inputIndex: number) => {
  inputs.value[inputIndex].refs.forEach((input, index) => {
    if (input.value === word[index]) {
      setTimeout(() => {
        input.style.backgroundColor = "green"
      }, 100 * (index + 1))
    } else if (word.includes(input.value)) {
      setTimeout(() => {
        input.style.backgroundColor = "yellow"
      }, 100 * (index + 1))
    } else {
      setTimeout(() => {
        input.style.backgroundColor = "red"
      }, 100 * (index + 1))
    }
  })
}

const handleKeyDown = async (inputIndex: number, index: number, event: KeyboardEvent) => {
  if (event.key === "Backspace") {
    if (index > 0 && !inputs.value[inputIndex].values[index]) {
      event.preventDefault()
      inputs.value[inputIndex].values[index - 1] = ""
      await nextTick()
      inputs.value[inputIndex].refs[index - 1].focus()
    }
  }

  if (event.key === "Enter" && inputs.value[inputIndex].values.every((value) => value !== "")) {
    await check(inputIndex)
    if (inputs.value[inputIndex].values.join("") !== word) {
      inputs.value[inputIndex].isLocked = true
      inputs.value[inputIndex + 1].isLocked = false
    }
  }
}

const handleInput = (inputIndex: number, index: number) => {
  if (index < length - 1) {
    inputs.value[inputIndex].refs[index + 1].focus()
  }
}
</script>

<template>
  <div class="container">
    <div v-for="(input, inputIndex) in inputs" :key="inputIndex" class="item">
      <div v-for="(_, index) in input.values" :key="index">
        <input
          type="text"
          maxlength="1"
          class="input"
          v-model="input.values[index]"
          :disabled="input.isLocked"
          :ref="el => input.refs[index] = el as HTMLInputElement"
          @input="handleInput(inputIndex, index)"
          @keydown="handleKeyDown(inputIndex, index, $event)"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  perspective: 1000px;
}

.item {
  display: flex;
  align-items: center;
  justify-content: center;
}

.input {
  text-align: center;
  width: 100px;
  height: 100px;
  background-color: #f0f0f0;
  border-radius: 16px;
  border: 1px solid #000;
  padding: 10px;
  margin: 10px;
  color: #000;
  font-size: 25px;
  transition: all 0.1s ease;
  backface-visibility: hidden;

  &:focus {
    transform: scale(1.1);
  }

  &:disabled {
    background-color: #000;
    color: #fff;
  }
}
</style>
