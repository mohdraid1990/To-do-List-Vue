<template>
  <div class="todo-app">
    <div class="current-info">
      <div class="current-time">{{ currentTime }}</div>
      <div class="current-location">{{ city }}, {{ country }}</div>
    </div>
    <div class="task-input">
      <input
        v-model="newTask"
        ref="taskInput"
        @keyup.enter="addTask"
        placeholder="Add New Todo"
      />
      <button @click="addTask">Add TO DO</button>
      <button @click="confirmClearAllTasks" class="clear-all-btn">
        Clear All
      </button>
    </div>

    <ul class="task-list">
      <li
        v-for="(task, index) in tasks"
        :key="index"
        class="task-item"
        :class="{ completed: task.completed, 'new-task': task.newlyAdded }"
      >
        <span>
          {{ task.text }} ({{ task.timestamp }}) - {{ task.city }},
          {{ task.country }}
        </span>
        <div class="task-actions">
          <button @click="toggleTaskCompletion(index)" class="complete-btn">
            {{ task.completed ? "Undo" : "Complete Task" }}
          </button>
          <button @click="confirmRemoveTask(index)" class="remove-btn">
            Remove
          </button>
        </div>
      </li>
    </ul>
  </div>
</template>

<style lang="scss">
$primary-color: #3498db;
$secondary-color: #2ecc71;
$danger-color: #83625e;
$background-color: #e7dddd;
$text-color: #a647b3;
$completed-color: rgb(167, 104, 104);
$border-radius: 5px;
$padding: 15px;

body {
  font-family: Cambria, Cochin, Georgia, Times, "Times New Roman", serif;
}

.todo-app {
  background-color: $background-color;
  padding: 20px;
  max-width: 600px;
  margin: 50px auto;
  border-radius: $border-radius;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  opacity: 0;
  animation: fadeIn 0.5s ease forwards;

  @keyframes fadeIn {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
}

.current-info {
  text-align: center;
  margin-bottom: 20px;
  color: $primary-color;
}

.current-time {
  font-size: 24px;
  font-weight: bold;
  color: $primary-color;
}

.current-location {
  font-size: 25px;
}

.task-input {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;

  input {
    flex: 1;
    padding: $padding;
    border: 1px solid $primary-color;
    border-radius: $border-radius;
    margin-right: 10px;
    outline: none;
  }

  button {
    padding: $padding;
    background-color: $primary-color;
    color: white;
    border: none;
    border-radius: $border-radius;
    cursor: pointer;
    transition: background-color 0.3s;

    &:hover {
      background-color: darken($primary-color, 10%);
    }
  }

  .clear-all-btn {
    background-color: $danger-color;
    margin-left: 10px;

    &:hover {
      background-color: darken($danger-color, 10%);
    }
  }
}

.task-list {
  list-style-type: none;
  padding: 0;

  .task-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: white;
    padding: $padding;
    border: 1px solid #ddd;
    border-radius: $border-radius;
    margin-bottom: 10px;
    transition: background-color 0.3s;

    &.completed {
      span {
        text-decoration: line-through;
        color: $completed-color;
      }
    }

    &.new-task {
      animation: newTaskAnimation 1s ease forwards;
    }

    @keyframes newTaskAnimation {
      from {
        transform: scale(0.8);
        opacity: 0;
      }
      to {
        transform: scale(1);
        opacity: 1;
      }
    }

    span {
      flex: 1;
      font-size: 20px;
      color: #b45b5b;
    }

    .task-actions {
      display: flex;
      gap: 10px;

      .complete-btn {
        padding: $padding / 2;
        background-color: $secondary-color;
        color: white;
        border: none;
        border-radius: $border-radius;
        cursor: pointer;
        transition: background-color 0.3s;

        &:hover {
          background-color: darken($secondary-color, 10%);
        }
      }

      .remove-btn {
        padding: $padding / 2;
        background-color: $danger-color;
        color: white;
        border: none;
        border-radius: $border-radius;
        cursor: pointer;
        transition: background-color 0.3s;

        &:hover {
          background-color: darken($danger-color, 10%);
        }
      }
    }
  }
}
</style>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";

const newTask = ref("");
const tasks = ref([]);
const taskInput = ref(null);
const currentTime = ref("");
const city = ref("");
const country = ref("");

const addTask = () => {
  if (newTask.value.trim() !== "") {
    const now = new Date();
    const timestamp = now.toLocaleString();
    tasks.value.push({
      text: newTask.value,
      completed: false,
      timestamp,
      city: city.value,
      country: country.value,
      newlyAdded: true,
    });
    newTask.value = "";

    setTimeout(() => {
      tasks.value[tasks.value.length - 1].newlyAdded = false;
    }, 1000);
  }
};

const removeTask = (index) => {
  tasks.value.splice(index, 1);
};

const clearAllTasks = () => {
  if (tasks.value.length > 0) {
    if (window.confirm("Are you sure you want to delete all tasks?")) {
      tasks.value = [];
    }
  }
};

const confirmRemoveTask = (index) => {
  if (window.confirm("Are you sure you want to delete this task?")) {
    removeTask(index);
  }
};

const confirmClearAllTasks = () => {
  clearAllTasks();
};

const toggleTaskCompletion = (index) => {
  tasks.value[index].completed = !tasks.value[index].completed;
};

const updateTime = () => {
  const now = new Date();
  currentTime.value = now.toLocaleTimeString();
};

const fetchLocation = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        fetch(
          `https://api.bigdatacloud.net/data/reverse-geocode-client?latitude=${latitude}&longitude=${longitude}&localityLanguage=en`
        )
          .then((response) => response.json())
          .then((data) => {
            city.value = data.city;
            country.value = data.countryName;
          })
          .catch((error) => {
            console.error("Error fetching location:", error);
            city.value = "Unknown";
            country.value = "Unknown";
          });
      },
      (error) => {
        console.error("Error getting geolocation:", error);
        city.value = "Unknown";
        country.value = "Unknown";
      }
    );
  } else {
    console.error("Geolocation not supported");
    city.value = "Unsupported";
    country.value = "Unsupported";
  }
};

onMounted(() => {
  taskInput.value.focus();
});

updateTime();
fetchLocation();
const intervalId = setInterval(updateTime, 1000);
onBeforeUnmount(() => {
  clearInterval(intervalId);
});
</script>
