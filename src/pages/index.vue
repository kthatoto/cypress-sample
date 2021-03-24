<template lang="pug">
.container
  h1 ToDo application
  .form
    .console
      el-input(v-model="newTaskName")
      .buttons
        el-button(type="primary" @click="createTask") 作成
        el-button(type="danger" @click="deleteDoneTasks") 完了したものを削除
    v-date-picker.calendar(v-model="date" color="blue" :attributes="attributes")

  ul.list
    li(v-for="task in orderedTasks" :key="task.id" :class="{ focusing: focusingByDate(task.id) }")
      span
        el-checkbox(:value="task.done" @input="toggleTask(task.id)" size="medium")
        label.content
          span.name {{ task.name }}
          span.date(:class="dateClass(task.date)") 期日：{{ dateFormat(task.date) }}
      el-button(v-if="task.done" type="danger" size="mini" @click="deleteTask(task.id)") 削除
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs, computed } from '@vue/composition-api'

interface Task {
  id: number
  name: string
  date: Date
  done: boolean
}

export default defineComponent({
  setup() {
    const state = reactive<{
      tasks: Task[]
      newTaskName: string
      date: Date
    }>({
      tasks: [],
      newTaskName: "",
      date: new Date()
    })

    const createTask = () => {
      if (!state.newTaskName) return
      if (!state.date) return
      const maxId = Math.max(...state.tasks.map(task => task.id), 0)
      console.log(maxId)
      state.tasks.push({
        id: maxId + 1,
        name: state.newTaskName,
        date: state.date,
        done: false
      })
      state.newTaskName = ""
    }

    const toggleTask = (id: number) => {
      const targetIndex = state.tasks.findIndex(task => task.id === id)
      if (targetIndex < 0) return
      const targetTask = state.tasks[targetIndex]
      state.tasks.splice(targetIndex, 1, {
        ...targetTask,
        done: !targetTask.done
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

    const dateEqual = (date1: Date, date2: Date) => {
      return date1.getFullYear() === date2.getFullYear() && date1.getMonth() === date2.getMonth() && date1.getDate() === date2.getDate()
    }

    const dateFormat = (date: Date) => {
      return `${date.getMonth() + 1}/${date.getDate()}`
    }
    const dateClass = (date: Date) => {
      const now = new Date()
      const today = new Date(now.getFullYear(), now.getMonth(), now.getDate())
      const due = new Date(date.getFullYear(), date.getMonth(), date.getDate())
      if (dateEqual(today, due)) return "pink"
      if (today > due) return "red"
      return ""
    }

    const attributes = computed(() => {
      const doneDates = state.tasks.filter((task: Task) => task.done).map((task: Task) => task.date)
      const notDoneDates = state.tasks.filter((task: Task) => !task.done).map((task: Task) => task.date)

      return [
        { dates: doneDates, bar: "green" },
        { dates: notDoneDates, bar: "red" }
      ]
    })

    const orderedTasks = computed(() => {
      return state.tasks.sort((t1: Task, t2: Task) => t1.date.getTime() - t2.date.getTime())
    })

    const focusingByDate = (id: number) => {
      const task = state.tasks.find((t: Task) => t.id === id)
      if (!task) return false
      return dateEqual(task.date, state.date)
    }

    return {
      ...toRefs(state),
      createTask,
      toggleTask,
      deleteTask,
      deleteDoneTasks,
      dateFormat,
      dateClass,
      attributes,
      orderedTasks,
      focusingByDate
    }
  }
})
</script>

<style lang="stylus" scoped>
.container
  max-width: 800px
  margin: auto
  .form
    display: flex
    justify-content: space-between

  .console
    .el-input
      width: 300px
      margin-bottom: 10px
    .buttons
      width: 300px
      display: flex
      justify-content: space-between
      .el-button
        width: 140px
        padding: 12px 0
        text-align: center
  .calendar
    width: 400px

  .list
    li
      width: 370px
      min-height: 36px
      list-style: none
      display: flex
      justify-content: space-between
      align-items: baseline
      word-break: break-all
      padding: 0 5px
      &.focusing
        background-color: #6495ED

      > span
        width: 300px
        display: flex
      .el-checkbox
        margin-right: 10px
        padding-top: 7px
      .content
        display: flex
        flex-direction: column
      .name
        font-size: 20px
      .date
        font-size: 14px
        &.pink
          color: #ff7f50
        &.red
          color: red
</style>
