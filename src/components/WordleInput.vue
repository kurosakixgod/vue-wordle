<script setup lang="ts">
import { nextTick, ref } from "vue"

const props = defineProps<{
  word?: string
}>()

type Input = {
  isLocked: boolean
  values: string[]
  refs: (HTMLInputElement | null)[]
}

const length = 5
const word = props.word || "hello"
const maxAttempts = 6

const inputs = ref<Input[]>(
  Array.from({ length: maxAttempts }, (_, i) => ({
    isLocked: i !== 0,
    values: Array.from({ length }, () => ""),
    refs: Array.from({ length }, () => null),
  }))
)

type GameStatus = 'playing' | 'won' | 'lost'
const gameStatus = ref<GameStatus>('playing')

const check = async (inputIndex: number) => {
  const guess = inputs.value[inputIndex].values.join("")

  // Подсчитываем количество каждой буквы в загаданном слове
  const letterCounts = new Map<string, number>()
  for (const letter of word) {
    letterCounts.set(letter, (letterCounts.get(letter) || 0) + 1)
  }

  // Массив результатов для каждой позиции
  const results: ('correct' | 'present' | 'absent')[] = Array(length).fill('absent')

  // Первый проход: отмечаем точные совпадения (зелёные)
  for (let i = 0; i < length; i++) {
    if (guess[i] === word[i]) {
      results[i] = 'correct'
      letterCounts.set(guess[i], letterCounts.get(guess[i])! - 1)
    }
  }

  // Второй проход: отмечаем буквы на неправильных позициях (жёлтые)
  for (let i = 0; i < length; i++) {
    if (results[i] !== 'correct' && letterCounts.get(guess[i])! > 0) {
      results[i] = 'present'
      letterCounts.set(guess[i], letterCounts.get(guess[i])! - 1)
    }
  }

  // Применяем цвета с анимацией
  inputs.value[inputIndex].refs.forEach((input, index) => {
    if (!input) return
    setTimeout(() => {
      const color = results[index] === 'correct' ? 'green'
                  : results[index] === 'present' ? 'yellow'
                  : 'red'
      input.style.backgroundColor = color
    }, 100 * (index + 1))
  })
}

const handleKeyDown = async (inputIndex: number, index: number, event: KeyboardEvent) => {
  // Блокируем ввод, если игра завершена
  if (gameStatus.value !== 'playing') {
    return
  }

  if (event.key === "Backspace") {
    if (index > 0 && !inputs.value[inputIndex].values[index]) {
      event.preventDefault()
      inputs.value[inputIndex].values[index - 1] = ""
      await nextTick()
      inputs.value[inputIndex].refs[index - 1]?.focus()
    }
  }

  if (event.key === "Enter" && inputs.value[inputIndex].values.every((value) => value !== "")) {
    await check(inputIndex)
    const guess = inputs.value[inputIndex].values.join("")

    if (guess === word) {
      // Выигрыш!
      gameStatus.value = 'won'
      inputs.value[inputIndex].isLocked = true
      setTimeout(() => {
        alert(`🎉 Поздравляем! Вы угадали слово "${word}" за ${inputIndex + 1} попыток!`)
      }, 600)
    } else if (inputIndex === maxAttempts - 1) {
      // Проигрыш - закончились попытки
      gameStatus.value = 'lost'
      inputs.value[inputIndex].isLocked = true
      setTimeout(() => {
        alert(`😢 Игра окончена! Загаданное слово: "${word}"`)
      }, 600)
    } else {
      // Переходим к следующей попытке
      inputs.value[inputIndex].isLocked = true
      inputs.value[inputIndex + 1].isLocked = false
    }
  }
}

const handleInput = (inputIndex: number, index: number) => {
  if (index < length - 1) {
    inputs.value[inputIndex].refs[index + 1]?.focus()
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
