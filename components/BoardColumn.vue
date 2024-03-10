<script setup>
import { ref } from 'vue'
import { useBoardStore } from '../stores/boardStore'
import { useRouter } from 'vue-router'

const props = defineProps({
  column: {
    type: Object,
    required: true
  },
  columnIndex: {
    type: Number,
    required: true
  }
})

const boardStore = useBoardStore()
const router = useRouter()

const editNameState = ref(false)
const newTaskName = ref('')
const newTaskDescription = ref('')
const showTaskModal = ref(false)

function addTask() {
  if (!newTaskName.value.trim()) {

    alert('Please enter a task name.')
    return
  }

 
  const description = prompt('Enter task description:')
  if (description === null) {

    return
  }

  boardStore.addTask({
    taskName: newTaskName.value,
    taskDescription: description, 
    columnIndex: props.columnIndex
  })
  newTaskName.value = ''
  showTaskModal.value = false 
}

function editColumn() {
  editNameState.value = !editNameState.value
}

function deleteColumn() {
  boardStore.deleteColumn(props.columnIndex)
}

function dropItem(event, { toColumnIndex, toTaskIndex }) {
  const type = event.dataTransfer.getData('type')
  const fromColumnIndex = event.dataTransfer.getData('from-column-index')

  if (type === 'task') {
    const fromTaskIndex = event.dataTransfer.getData('from-task-index')

    boardStore.moveTask({
      fromTaskIndex,
      toTaskIndex,
      fromColumnIndex,
      toColumnIndex
    })
  } else if (type === 'column') {
    boardStore.moveColumn({
      fromColumnIndex,
      toColumnIndex
    })
  }
}

function goToTask(taskId) {
  router.push(`/tasks/${taskId}`)
}

function pickupColumn(event, fromColumnIndex) {
  event.dataTransfer.effectAllowed = 'move'
  event.dataTransfer.dropEffect = 'move'
  event.dataTransfer.setData('type', 'column')
  event.dataTransfer.setData('from-column-index', fromColumnIndex)
}

function pickupTask(event, { fromColumnIndex, fromTaskIndex }) {
  event.dataTransfer.effectAllowed = 'move'
  event.dataTransfer.dropEffect = 'move'
  event.dataTransfer.setData('type', 'task')
  event.dataTransfer.setData('from-column-index', fromColumnIndex)
  event.dataTransfer.setData('from-task-index', fromTaskIndex)
}
</script>

<template>
  <UContainer
    class="column"
    draggable="true"
    @dragstart.self="pickupColumn($event, columnIndex)"
    @dragenter.prevent
    @dragover.prevent
    @drop.stop="dropItem($event, { toColumnIndex: columnIndex })"
  >
    <div class="column-header mb-4">
      <div>
        <UInput v-if="editNameState" type="text" v-model="column.name" />
        <h2 v-else>{{ column.name }}</h2>
        <p>[{{ column.tasks.length }} Tasks]</p>
      </div>
      <div>
        <UButton class="bg-white-500 hover:bg-blue-600 text-white" @click="editColumn">Edit</UButton>
        <UButton class="bg-white-500 hover:bg-red-600 text-white" @click="deleteColumn"> Delete </UButton>
      </div>
    </div>
    <ul>
      <li v-for="(task, taskIndex) in column.tasks" :key="task.id">
        <UCard
          class="mb-4"
          @click="goToTask(task.id)"
          draggable="true"
          @dragstart="
            pickupTask($event, {
              fromColumnIndex: columnIndex,
              fromTaskIndex: taskIndex
            })
          "
          @drop.stop="
            dropItem($event, {
              toColumnIndex: columnIndex,
              toTaskIndex: taskIndex
            })
          "
        >
          <strong>{{ task.name }}</strong>
          <p>{{ task.description }}</p>
        </UCard>
      </li>
    </ul>
    <!-- Task creation modal -->
    <UModal v-model="showTaskModal" @close="showTaskModal = false">
      <div class="new-task p-4 bg-white rounded shadow-md">
        <h3 class="text-lg font-semibold mb-2">Create New Task</h3>
        <UInput v-model="newTaskName" placeholder="Task name" class="mb-2" />
        <UInput v-model="newTaskDescription" placeholder="Task description" class="mb-4" />
        <UButton @click="addTask" class="bg-blue-500 hover:bg-blue-600 text-white py-2 px-4 rounded">Add Task</UButton>
      </div>
    </UModal>
    <!-- New Task Button -->
    <UButton
      icon="i-heroicons-plus-circle-solid"
      @click="showTaskModal = true"
    >
      New
    </UButton>
  </UContainer>
</template>
