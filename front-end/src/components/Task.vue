<template>
  <div class="task-list">
    <h1>Task List</h1>
    <!-- Search and Filter -->
    <div class="filters">
      <input type="text" v-model="searchQuery" placeholder="Search tasks..." class="search-input" />
      <select v-model="filterStatus" class="filter-select">
        <option value="">All</option>
        <option value="completed">Completed</option>
        <option value="pending">Pending</option>
      </select>
    </div>

    <!-- Add New Task -->
    <div class="add-task">
      <h3>Add New Task</h3>
      <input type="text" v-model="newTask.title" placeholder="Task title" class="input" />
      <input
        type="text"
        v-model="newTask.description"
        placeholder="Task description"
        class="input"
      />
      <input type="date" v-model="newTask.dueDate" class="input" />
      <button @click="createTask" class="btn">Add Task</button>
    </div>

    <!-- Loading Indicator -->
    <div v-if="loading" class="loading">Loading...</div>

    <!-- Task List -->
    <div v-else class="tasks-container">
      <ul class="tasks-list">
        <li v-for="task in filteredTasks" :key="task.id" class="task-item" @click="editTask(task)">
          <h3>{{ task.title }}</h3>
          <p><strong>Description:</strong> {{ task.description }}</p>
          <p><strong>Due Date:</strong> {{ formatDate(task.dueDate) }}</p>
          <p>
            <strong>Status:</strong>
            <span :class="{ completed: task.status === 'completed' }">
              {{ task.status }}
            </span>
          </p>
          <button @click.stop="toggleStatus(task)" class="btn">
            Mark as {{ task.status === 'completed' ? 'Incomplete' : 'Complete' }}
          </button>
        </li>
      </ul>
    </div>

    <!-- Error Handling -->
    <div v-if="error" class="error">
      <p>Error loading tasks: {{ error }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tasks: [], // Tasks loaded from the API
      loading: true,
      error: null,
      searchQuery: '',
      filterStatus: '',
      newTask: {
        title: '',
        description: '',
        dueDate: '',
        status: 'pending', // Default status
      },
    }
  },
  computed: {
    filteredTasks() {
      let filtered = this.tasks

      // Filter by status
      if (this.filterStatus) {
        filtered = filtered.filter((task) => task.status === this.filterStatus)
      }

      // Filter by search query
      if (this.searchQuery) {
        filtered = filtered.filter((task) =>
          task.title.toLowerCase().includes(this.searchQuery.toLowerCase()),
        )
      }

      return filtered
    },
  },
  mounted() {
    this.fetchTasks()
  },
  methods: {
    async fetchTasks() {
      try {
        const response = await fetch('http://localhost:9191/api/tasks')
        if (!response.ok) {
          throw new Error('Failed to fetch tasks')
        }
        const data = await response.json()
        this.tasks = data
      } catch (err) {
        this.error = err.message
      } finally {
        this.loading = false
      }
    },
    async createTask() {
      const taskData = {
        title: this.newTask.title,
        description: this.newTask.description,
        dueDate: this.newTask.dueDate,
        completed: this.newTask.completed,
      }
      try {
        const response = await fetch('http://localhost:9191/api/tasks', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(taskData),
        })

        if (response.ok) {
          const result = await response.json()
          this.tasks.push(result) // Add the new task to the list
          this.newTask = { title: '', description: '', dueDate: '', status: 'pending' }
        } else {
          alert('Error creating task')
        }
      } catch (error) {
        alert('Request failed: ' + error.message)
      }
    },
    editTask(task) {
      const title = prompt('Edit task title', task.title)
      const description = prompt('Edit task description', task.description)
      const dueDate = prompt('Edit due date (YYYY-MM-DD)', task.dueDate)

      if (title && description && dueDate) {
        task.title = title
        task.description = description
        task.dueDate = dueDate
      }
    },
    toggleStatus(task) {
      task.status = task.status === 'completed' ? 'pending' : 'completed'
    },
    formatDate(date) {
      if (!date) return 'N/A'
      return new Date(date).toLocaleDateString() // Formats as MM/DD/YYYY
    },
  },
}
</script>

<style scoped>
.task-list {
  font-family: Arial, sans-serif;
  padding: 20px;
  margin: 0 auto;
  max-width: 1200px;
  width: 100%;
  box-sizing: border-box;
}

.filters {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.add-task {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.input {
  padding: 10px;
  width: 100%;
  max-width: 400px;
  box-sizing: border-box;
}

.btn {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 4px;
}

.btn:hover {
  background-color: #0056b3;
}

.tasks-container {
  max-height: 400px;
  overflow-y: auto;
  border: 1px solid #ccc;
  padding: 10px;
  background-color: #f9f9f9;
  border-radius: 8px;
}

.tasks-list {
  list-style: none;
  padding: 0;
}

.task-item {
  border: 1px solid #ccc;
  padding: 15px;
  margin-bottom: 10px;
  background-color: white;
  border-radius: 8px;
}

.task-item.completed {
  background-color: #d4edda;
  border-color: #c3e6cb;
}

.loading,
.error {
  text-align: center;
  font-size: 18px;
}

@media (max-width: 768px) {
  .filters,
  .add-task {
    flex-direction: column;
    align-items: stretch;
  }

  .input {
    width: 100%;
  }
}
</style>
