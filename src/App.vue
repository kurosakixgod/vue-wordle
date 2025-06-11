<script setup lang="ts">
import { nextTick, ref } from "vue"

const length = 5
const inputValues = ref(Array(length).fill(""))
const inputRefs = ref<HTMLInputElement[]>(Array(length).fill(null))

const check = () => {
  console.log(inputValues.value.join(""))
}

const handleInput = async (index: number) => {
  if (index < length - 1) {
    await nextTick()
    inputRefs.value[index + 1].focus()
  }
}

const handleKeyDown = async (index: number, event: KeyboardEvent) => {
  if (event.key === "Backspace") {
    if (index > 0 && !inputValues.value[index]) {
      event.preventDefault()
      inputValues.value[index - 1] = ""
      await nextTick()
      inputRefs.value[index - 1].focus()
    }
  }
}
</script>

<template>
  <div>
    <div v-for="(_, index) in inputValues" :key="index" class="item">
      <input
        type="text"
        maxlength="1"
        class="input"
        v-model="inputValues[index]"
        :ref="el => inputRefs[index] = el as HTMLInputElement"
        @input="handleInput(index)"
        @keydown="handleKeyDown(index, $event)"
      />
    </div>
    <button @click="check">Проверить</button>
  </div>
</template>

<style scoped>
.input {
  text-align: center;
  width: 100px;
  height: 100px;
  background-color: #f0f0f0;
  border-radius: 5px;
  border: 1px solid #000;
  padding: 10px;
  margin: 10px;
  color: #000;
  font-size: 25px;
}
</style>
