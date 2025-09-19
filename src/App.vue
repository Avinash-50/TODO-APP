<template>
  <div id="app">
    <div class="container">
      <header>
        <h1><i class="fas fa-tasks"></i> Vue.js To-Do List</h1>
        <p class="app-description">Manage your tasks efficiently</p>
      </header>

      <div class="main-content">
        <div class="input-section">
          <input 
            v-model="newTask" 
            @keyup.enter="addTask" 
            class="task-input" 
            placeholder="Add a new task..."
          >
          <button @click="addTask" class="add-btn">
            <i class="fas fa-plus"></i> Add
          </button>
        </div>

        <div class="tabs">
          <div 
            :class="['tab', activeTab === 'all' ? 'active' : '']" 
            @click="activeTab = 'all'"
          >
            All Tasks
          </div>
          <div 
            :class="['tab', activeTab === 'active' ? 'active' : '']" 
            @click="activeTab = 'active'"
          >
            Active
          </div>
          <div 
            :class="['tab', activeTab === 'completed' ? 'active' : '']" 
            @click="activeTab = 'completed'"
          >
            Completed
          </div>
        </div>

        <ul class="task-list">
          <li 
            v-for="task in filteredTasks" 
            :key="task.id" 
            :class="['task-item', task.completed ? 'completed' : '']"
          >
            <input 
              type="checkbox" 
              v-model="task.completed" 
              class="task-checkbox"
            >
            <div class="task-content">
              <div class="task-text">{{ task.text }}</div>
              <div class="task-date">
                <i class="far fa-calendar"></i> 
                {{ formatDate(task.createdAt) }}
              </div>
            </div>
            <button @click="deleteTask(task.id)" class="delete-btn">
              <i class="fas fa-trash"></i>
            </button>
          </li>
        </ul>

        <div v-if="filteredTasks.length === 0" class="empty-state">
          <i class="fas fa-inbox"></i>
          <h3>No tasks found</h3>
          <p>Try adding a new task or changing the active filter</p>
        </div>

        <div class="stats">
          <div class="stat-item">
            <div class="stat-value">{{ totalTasks }}</div>
            <div class="stat-label">Total Tasks</div>
          </div>
          <div class="stat-item">
            <div class="stat-value">{{ activeTasks }}</div>
            <div class="stat-label">Active Tasks</div>
          </div>
          <div class="stat-item">
            <div class="stat-value">{{ completedTasks }}</div>
            <div class="stat-label">Completed</div>
          </div>
        </div>
      </div>

      <footer>
        <p>Your tasks are automatically saved to local storage</p>
      </footer>
    </div>
  </div>
</template>

<script>
// Simulated JSON Server API functions
const jsonServerAPI = {
  // Get tasks from localStorage
  getTasks() {
    return new Promise((resolve) => {
      const tasks = JSON.parse(localStorage.getItem('vue-todo-tasks') || '[]');
      // Simulate network delay
      setTimeout(() => resolve(tasks), 100);
    });
  },
  
  // Save tasks to localStorage
  saveTasks(tasks) {
    return new Promise((resolve) => {
      localStorage.setItem('vue-todo-tasks', JSON.stringify(tasks));
      // Simulate network delay
      setTimeout(() => resolve({ success: true }), 100);
    });
  },
  
  // Filter tasks by completion status
  filterTasks(tasks, completed) {
    return new Promise((resolve) => {
      const filtered = tasks.filter(task => task.completed === completed);
      // Simulate network delay
      setTimeout(() => resolve(filtered), 100);
    });
  }
};

export default {
  name: 'App',
  data() {
    return {
      tasks: [],
      newTask: '',
      activeTab: 'all'
    }
  },
  computed: {
    filteredTasks() {
      switch (this.activeTab) {
        case 'active':
          return this.tasks.filter(task => !task.completed);
        case 'completed':
          return this.tasks.filter(task => task.completed);
        default:
          return this.tasks;
      }
    },
    totalTasks() {
      return this.tasks.length;
    },
    activeTasks() {
      return this.tasks.filter(task => !task.completed).length;
    },
    completedTasks() {
      return this.tasks.filter(task => task.completed).length;
    }
  },
  methods: {
    async addTask() {
      if (!this.newTask.trim()) return;
      
      const newTask = {
        id: Date.now(),
        text: this.newTask.trim(),
        completed: false,
        createdAt: new Date().toISOString()
      };
      
      this.tasks.unshift(newTask);
      this.newTask = '';
      
      await jsonServerAPI.saveTasks(this.tasks);
    },
    async deleteTask(id) {
      this.tasks = this.tasks.filter(task => task.id !== id);
      await jsonServerAPI.saveTasks(this.tasks);
    },
    formatDate(dateString) {
      const options = { 
        year: 'numeric', 
        month: 'short', 
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      };
      return new Date(dateString).toLocaleDateString('en-US', options);
    },
    async loadTasks() {
      this.tasks = await jsonServerAPI.getTasks();
    }
  },
  watch: {
    tasks: {
      deep: true,
      handler: async function() {
        await jsonServerAPI.saveTasks(this.tasks);
      }
    }
  },
  async created() {
    await this.loadTasks();
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #6a11cb 0%, #2575fc 100%);;
  color: #333;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}
.container {
  width: 100%;
  max-width: 800px;
  background: white;
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  overflow: hidden;
}
header {
  background: #4a6fc7;
  color: white;
  padding: 20px;
  text-align: center;
}
h1 {
  font-size: 28px;
  margin-bottom: 10px;
}
.app-description {
  font-size: 16px;
  opacity: 0.9;
}
.main-content {
  padding: 25px;
}
.input-section {
  display: flex;
  margin-bottom: 25px;
  gap: 10px;
}
.task-input {
  flex: 1;
  padding: 14px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.3s;
}
.task-input:focus {
  border-color: #4a6fc7;
  outline: none;
}
.add-btn {
  background: #4a6fc7;
  color: white;
  border: none;
  padding: 0 20px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  transition: background 0.3s;
}
.add-btn:hover {
  background: #3a5ca9;
}
.tabs {
  display: flex;
  margin-bottom: 20px;
  border-bottom: 2px solid #eee;
}
.tab {
  padding: 12px 20px;
  cursor: pointer;
  font-weight: 600;
  color: #777;
  transition: all 0.3s;
}
.tab.active {
  color: #4a6fc7;
  border-bottom: 3px solid #4a6fc7;
}
.task-list {
  list-style: none;
  margin-bottom: 30px;
}
.task-item {
  display: flex;
  align-items: center;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 12px;
  background: #f8f9fa;
  transition: transform 0.2s, box-shadow 0.2s;
}
.task-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}
.task-checkbox {
  margin-right: 15px;
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.task-text {
  flex: 1;
  font-size: 16px;
}
.task-date {
  font-size: 12px;
  color: #888;
  margin-top: 4px;
}
.completed .task-text {
  text-decoration: line-through;
  color: #888;
}
.delete-btn {
  background: #ff4757;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
}
.delete-btn:hover {
  background: #ff2e43;
}
.empty-state {
  text-align: center;
  padding: 30px;
  color: #888;
}
.empty-state i {
  font-size: 50px;
  margin-bottom: 15px;
  color: #ccc;
}
.stats {
  background: #f1f5f9;
  padding: 15px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}
.stat-item {
  text-align: center;
}
.stat-value {
  font-size: 24px;
  font-weight: 700;
  color: #4a6fc7;
}
.stat-label {
  font-size: 14px;
  color: #666;
}
footer {
  text-align: center;
  padding: 20px;
  background: #f1f5f9;
  color: #666;
  font-size: 14px;
}
@media (max-width: 600px) {
  .input-section {
    flex-direction: column;
  }
  .task-item {
    flex-direction: column;
    align-items: flex-start;
  }
  .task-actions {
    align-self: flex-end;
    margin-top: 10px;
  }
  .stats {
    flex-direction: column;
    gap: 15px;
  }
}
</style>