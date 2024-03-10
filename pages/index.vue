<script setup>
import { useBoardStore } from '../stores/boardStore'

const boardStore = useBoardStore()
const route = useRoute()
const router = useRouter()

const newColumnName = ref('')

const isModalOpen = computed(() => {
  return route.name === 'index-tasks-id'
})

function addColumn() {
  boardStore.addColumn(newColumnName.value)
  newColumnName.value = ''
}

function closeModal() {
  router.push('/')
}
</script>

<template>
  <div class="board-wrapper">
    <main class="board">
      <BoardColumn
        v-for="(column, columnIndex) in boardStore.board.columns"
        :key="column.id"
        :column="column"
        :columnIndex="columnIndex"
      />
    </main>
    <div v-show="isModalOpen" class="task-bg" @click.self="closeModal">
      <NuxtPage :key="route.fullPath" />
    </div>
  </div>
<div>
  <UContainer class="column fixed bottom-0 left-0 right-0 w-40 px-2 py-1 rounded-sm border border-gray-300">
    <UInput
      v-model="newColumnName"
      type="text"
      placeholder="Create new column"
      icon="i-heroicons-plus-circle-solid"
      @keyup.enter="addColumn"
    />
  </UContainer>
</div>
</template>
