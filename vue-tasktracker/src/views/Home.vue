<template>
  <div>
    <div v-show="showAddTask">
      <AddTask @add-task="addTask" />
    </div>
    <Tasks
      @toggle-reminder="toggleReminder"
      @delete-task="deleteTask"
      :tasks="tasks"
    />
  </div>
</template>

<script>
import Tasks from '../components/Tasks'
import AddTask from '../components/AddTask'

export default {
  name: 'Home',
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
      showAddTask: false,
    }
  },
  methods: {
    // Add task
    async addTask(task) {
      const res = await fetch('api/tasks', {
        method: 'POST',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(task),
      })

      const data = await res.json()

      this.tasks.push(data)
    },

    // Delete Task
    async deleteTask(id) {
      if (confirm('Are you sure you want to delete?')) {
        const res = await fetch(`api/tasks/${id}`, {
          method: 'DELETE',
        })

        if (res.status === 200) {
          this.tasks = this.tasks.filter((task) => task.id !== id)
        } else {
          alert('Error deleting task')
        }
      }
    },

    // Toggle Reminder
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id)
      const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder }

      const res = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(updTask),
      })

      const data = await res.json()

      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: data.reminder } : task
      )
    },

    // Fetch Tasks
    async fetchTasks() {
      const res = await fetch('api/tasks')
      const data = await res.json()
      return data
    },

    // Fetch Task
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`)
      const data = await res.json()
      return data
    },
  },

  // Create Task
  async created() {
    this.tasks = await this.fetchTasks()
  },
}
</script>
