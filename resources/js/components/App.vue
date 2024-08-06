<template>
  <div class="container mt-4">
    <h1 class="mb-4">PHP - Simple To Do List App</h1>
    <div class="mb-3">
      <input v-model="newTask" @keyup.enter="addTask" placeholder="Enter a new task" class="form-control" />
    </div>
    <div class="mb-3">
      <button @click="addTask" class="btn btn-primary">Add Task</button>
      <button @click="fetchTasks" class="btn btn-info">Show Incomplete Tasks</button>
      <button @click="fetchAllTasks" class="btn btn-secondary">Show All Tasks</button>
    </div>
    <table class="table table-striped table-bordered">
      <thead class="thead-dark">
        <tr>
          <th>#</th>
          <th>Task</th>
          <th>Status</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(task, index) in tasks" :key="task.id">
          <td>{{ index + 1 }}</td>
          <td>{{ task.task }}</td>
          <td>{{ task.is_completed ? 'Done' : 'Not Completed' }}</td>
          <td>
            <!-- Show button only if the task is not completed -->
            <button v-if="!task.is_completed" @click="toggleComplete(task, index)" class="btn btn-success">
              Complete
            </button>
            <button @click="confirmDelete(task.id)" class="btn btn-danger">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      tasks: [],
      newTask: ''
    };
  },
  methods: {
    async fetchTasks() {
      try {
        const response = await axios.get('/api/tasks'); // Fetch only incomplete tasks
        this.tasks = response.data;
      } catch (error) {
        console.error("Error fetching tasks:", error);
      }
    },
    async fetchAllTasks() {
      try {
        const response = await axios.get('/api/tasks/showAll'); // Fetch all tasks
        this.tasks = response.data;
      } catch (error) {
        console.error("Error fetching all tasks:", error);
      }
    },
    async addTask() {
      if (this.newTask.trim() === '') return;

      try {
        const response = await axios.post('/api/tasks', { task: this.newTask });
        this.tasks.push(response.data);
        this.newTask = '';
      } catch (error) {
        console.error("Error adding task:", error);
      }
    },
    async toggleComplete(task, index) {
      if (!task) {
        console.error("Task is undefined in toggleComplete method");
        return;
      }

      task.is_completed = true; // Mark as completed

      try {
        await axios.put(`/api/tasks/${task.id}`, { is_completed: task.is_completed });
        this.tasks.splice(index, 1); // Remove the task from the list
      } catch (error) {
        console.error("Error updating task completion:", error);
      }
    },
    async confirmDelete(taskId) {
      if (confirm('Are you sure to delete this task?')) {
        try {
          await axios.delete(`/api/tasks/${taskId}`);
          this.tasks = this.tasks.filter(task => task.id !== taskId);
        } catch (error) {
          console.error("Error deleting task:", error);
        }
      }
    }
  },
  created() {
    this.fetchTasks(); // Fetch incomplete tasks initially
  }
};
</script>

<style>
.table {
  width: 100%;
  margin-top: 20px;
  border-collapse: collapse;
}

.table th, .table td {
  border: 1px solid #ddd;
  padding: 8px;
}

.table th {
  background-color: #f2f2f2;
  text-align: center;
}

.table td {
  text-align: center;
}

.btn {
  margin: 0 5px;
}
</style>
