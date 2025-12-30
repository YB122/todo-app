<template>
  <div class="todo-app-wrapper min-vh-100 py-4" :class="wrapperClasses">
    <div class="container">
      <h1 class="text-center mb-5 app-title">My Vue ToDo App</h1>

      <!-- Input and Button -->
      <div class="d-flex mb-4">
        <input
          ref="taskInput"
          v-model="task"
          type="text"
          placeholder="Enter Task"
          class="form-control me-2 flex-grow-1 task-input shadow-none"
          @keyup.enter="submitTask"
        />
        <button
          @click="submitTask"
          class="btn btn-warning rounded-0 fw-bold px-4 shadow-none"
        >
          Add Task
        </button>
        <button
          @click="toggleDarkMode"
          class="btn ms-2 border-0 shadow-none theme-toggle"
          type="button"
          aria-label="Toggle Theme"
        >
          <span v-if="isDarkMode" key="sun">
            <i class="fa-solid fa-sun text-warning fs-3"></i>
          </span>
          <span v-else key="moon">
            <i class="fa-solid fa-moon text-dark fs-3"></i>
          </span>
        </button>
      </div>

      <!-- Task List -->
      <div
        class="table-responsive"
        :class="{ 'blurred-content': editedTask !== null }"
      >
        <table class="table table-bordered mt-2 custom-table">
          <thead>
            <tr>
              <th scope="col">Task</th>
              <th scope="col" style="width: 120px">Status</th>
              <th scope="col" class="text-center" style="width: 80px">EDIT</th>
              <th scope="col" class="text-center" style="width: 80px">
                DELETE
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(task, index) in tasks" :key="task.id">
              <td class="align-middle">
                <span
                  :class="{ finished: task.status === 'Completed' }"
                  class="text-break task-name"
                >
                  {{ task.name }}
                </span>
              </td>
              <td class="align-middle">
                <span
                  :class="statusClasses(task.status)"
                  @click="changeStatus(index)"
                  class="pointer status-badge fw-bold"
                >
                  {{ task.status }}
                </span>
              </td>
              <td class="text-center align-middle">
                <div @click="editTask(index)">
                  <span class="fa fa-pen pointer text-primary edit-icon"></span>
                </div>
              </td>
              <td class="text-center align-middle">
                <div @click="deleteTask(index)">
                  <span
                    class="fa fa-trash pointer text-danger delete-icon"
                  ></span>
                </div>
              </td>
            </tr>
          </tbody>
        </table>

        <!-- Empty State -->
        <div v-if="tasks.length === 0" class="text-center mt-5 opacity-50">
          <p class="fs-4">No tasks yet. Add one above! ✨</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "TodoApp",
  data() {
    return {
      task: "",
      editedTask: null,
      isDarkMode: false,
      availableStatuses: ["To-Do", "In Progress", "Completed"],
      tasks: [
        {
          id: Date.now() + 1,
          name: "Steal bananas from the store.",
          status: "To-Do",
        },
        { id: Date.now() + 2, name: "Wash the car.", status: "Completed" },
        {
          id: Date.now() + 3,
          name: "Write Vue.js app.",
          status: "In Progress",
        },
      ],
    };
  },
  created() {
    try {
      const savedTasks = localStorage.getItem("vue-todo-tasks");
      if (savedTasks) {
        const parsed = JSON.parse(savedTasks);
        if (Array.isArray(parsed)) {
          this.tasks = parsed.map((t) => ({
            ...t,
            id: t.id || Math.random().toString(36).substr(2, 9),
          }));
        }
      }

      const savedDarkMode = localStorage.getItem("vue-todo-dark-mode");
      if (savedDarkMode !== null) {
        this.isDarkMode = JSON.parse(savedDarkMode);
      }
    } catch (e) {
      console.error("Init Error:", e);
    }
  },
  watch: {
    tasks: {
      handler(newTasks) {
        localStorage.setItem("vue-todo-tasks", JSON.stringify(newTasks));
      },
      deep: true,
    },
    isDarkMode: {
      handler(val) {
        localStorage.setItem("vue-todo-dark-mode", JSON.stringify(val));
        if (val) {
          document.body.classList.add("dark-mode");
        } else {
          document.body.classList.remove("dark-mode");
        }
      },
      immediate: true,
    },
  },
  computed: {
    wrapperClasses() {
      return { "dark-mode": this.isDarkMode };
    },
  },
  methods: {
    submitTask() {
      if (!this.task || this.task.trim() === "") return;
      if (this.editedTask === null) {
        this.tasks.push({
          id: Date.now(),
          name: this.task.trim(),
          status: "To-Do",
        });
      } else {
        if (this.tasks[this.editedTask]) {
          this.tasks[this.editedTask].name = this.task.trim();
        }
        this.editedTask = null;
      }
      this.task = "";
    },
    editTask(index) {
      if (!this.tasks[index]) return;
      this.task = this.tasks[index].name;
      this.editedTask = index;
      this.$nextTick(() => {
        if (this.$refs.taskInput) this.$refs.taskInput.focus();
      });
    },
    deleteTask(index) {
      if (!this.tasks[index]) return;
      this.tasks.splice(index, 1);
      this.editedTask = null;
    },
    changeStatus(index) {
      if (!this.tasks[index]) return;
      const currentIndex = this.availableStatuses.indexOf(
        this.tasks[index].status
      );
      const nextIndex = (currentIndex + 1) % this.availableStatuses.length;
      this.tasks[index].status = this.availableStatuses[nextIndex];
    },
    statusClasses(status) {
      return {
        "text-danger": status === "To-Do",
        "text-warning": status === "In Progress",
        "text-success": status === "Completed",
      };
    },
    toggleDarkMode() {
      this.isDarkMode = !this.isDarkMode;
    },
  },
};
</script>

<style>
:root {
  --app-bg: #ffffff;
  --app-text: #212529;
  --table-bg: #ffffff;
}
body.dark-mode {
  --app-bg: #1a1a1a;
  --app-text: #ffffff;
  --table-bg: #212529;
}
body {
  margin: 0;
  padding: 0;
  background-color: var(--app-bg) !important;
  color: var(--app-text) !important;
  transition: background-color 0.3s ease, color 0.3s ease;
  min-height: 100vh;
}
</style>

<style scoped>
.todo-app-wrapper {
  background-color: var(--app-bg);
  color: var(--app-text);
  transition: all 0.3s ease;
}
.app-title {
  color: var(--app-text);
}
.task-input {
  background-color: var(--app-bg) !important;
  color: var(--app-text) !important;
}
.dark-mode .task-input {
  background-color: #2b3035 !important;
  border-color: #495057 !important;
}
.dark-mode .task-input::placeholder {
  color: #ffffff !important;
  opacity: 0.7;
}
.custom-table {
  background-color: var(--table-bg) !important;
  color: var(--app-text) !important;
  border-color: #dee2e6;
}
.dark-mode .custom-table {
  border-color: #373b3e;
}
.custom-table th,
.custom-table td {
  background-color: var(--table-bg) !important;
  color: var(--app-text) !important;
  border-color: inherit;
}
.status-badge {
  cursor: pointer;
}
.pointer {
  cursor: pointer;
}
.finished {
  text-decoration: line-through;
  opacity: 0.5;
}
.text-break {
  word-break: break-word;
}
.blurred-content {
  filter: blur(4px);
  opacity: 0.4;
  pointer-events: none;
}
/* Hardcoded status colors */
.text-danger {
  color: #ff4d4d !important;
}
.text-warning {
  color: #ffc107 !important;
}
.text-success {
  color: #28a745 !important;
}
</style>
