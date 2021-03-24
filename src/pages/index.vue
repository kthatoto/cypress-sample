<template lang="pug">
.container
  h1 ToDo application
  el-input(v-model="newTaskName")
  el-button(type="primary" @click="createTask") 作成

  ul
    li(v-for="task in tasks" :key="task.id")
      span
        el-checkbox(v-model="task.done")
        span {{ task.name }}
      el-button(v-if="task.done" type="danger" size="mini") 削除
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs } from '@vue/composition-api'

interface Task {
  id: number
  name: string
  done: boolean
}

export default defineComponent({
  setup() {
    const state = reactive<{
      tasks: Task[]
      newTaskName: string
    }>({
      tasks: [],
      newTaskName: ""
    })

    const createTask = () => {
      if (!state.newTaskName) return
      const maxId = Math.max(...state.tasks.map(task => task.id), 0)
      console.log(maxId)
      state.tasks.push({
        id: maxId + 1,
        name: state.newTaskName,
        done: false
      })
      state.newTaskName = ""
    }

    const doneTask = (id: number) => {
      const targetIndex = state.tasks.findIndex(task => task.id === id)
      if (targetIndex < 0) return
      state.tasks.splice(targetIndex, 1, {
        ...state.tasks[targetIndex],
        done: true
      })
    }

    const undoneTask = (id: number) => {
      const targetIndex = state.tasks.findIndex(task => task.id === id)
      if (targetIndex < 0) return
      state.tasks.splice(targetIndex, 1, {
        ...state.tasks[targetIndex],
        done: false
      })
    }

    const deleteTask = (id: number) => {
      const targetIndex = state.tasks.findIndex(task => task.id === id)
      if (targetIndex < 0) return
      state.tasks.splice(targetIndex, 1)
    }

    const deleteDoneTasks = () => {
      state.tasks = state.tasks.filter((task: Task) => !task.done)
    }

    return {
      ...toRefs(state),
      createTask,
      doneTask,
      deleteTask,
      deleteDoneTasks,
    }
  }
})
</script>

<style lang="stylus" scoped>
</style>
