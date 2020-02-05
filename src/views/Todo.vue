<template>
  <section class="todo">
    <header>
      <input
        class="new-todo"
        autocomplete="off"
        placeholder="What will you do today?"
        v-model="newTodo"
        @keyup.enter="addTodo"
      />
      <button class="new-todo-button" @click="addTodo" v-show="newTodo.length > 0">
        <i class="fas plus-circle"></i>
      </button>
    </header>
    <section class="main" v-show="todos.length" v-cloak>
      <div class="completed-wrapper">
        <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone" />
        <label for="toggle-all" :class="{'un-mute':allDone}">
          <i class="far fa-check-square"></i>Complete all tasks
        </label>
        <button class="clear-completed" @click="removeCompleted">Clear completed</button>
      </div>
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          :key="todo.id"
          :class="{ completed: todo.completed, editing: todo == editedTodo }"
        >
          <div class="view">
            <input
              class="toggle"
              :class="{'far fa-check-circle':todo.completed,'far fa-circle':!todo.completed, }"
              type="checkbox"
              v-model="todo.completed"
            />
            <label @dblclick="editTodo(todo)" @click="markAsCompleted(todo)">{{ todo.title }}</label>
            <button class="remove-btn" @click="removeTodo(todo)">
              <i class="far fa-trash-alt"></i>
            </button>
          </div>
          <input
            class="edit"
            type="text"
            v-model="todo.title"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
          />
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length" v-cloak>
      <span class="todo-count">
        <strong>{{ remaining }}</strong>
        {{ remaining | pluralize }} left
      </span>
      <ul class="filters">
        <li>
          <a href="#/all" :class="{ selected: visibility == 'all' }">All</a>
        </li>
        <li>
          <a href="#/active" :class="{ selected: visibility == 'active' }">Uncomplete</a>
        </li>
        <li>
          <a href="#/completed" :class="{ selected: visibility == 'completed' }">Completed</a>
        </li>
      </ul>
    </footer>
  </section>
</template>

<script>
const STORAGE_KEY = "m-todo_";
const store = {
  fetch() {
    var todos = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
    todos.forEach(function(todo, index) {
      todo.id = index;
    });
    store.uid = todos.length;
    return todos;
  },
  save(todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
  }
};
// visibility filters
var filters = {
  all(todos) {
    return todos;
  },
  active(todos) {
    return todos.filter(function(todo) {
      return !todo.completed;
    });
  },
  completed(todos) {
    return todos.filter(function(todo) {
      return todo.completed;
    });
  }
};
export default {
  name: "Todo",
  data() {
    return {
      todos: store.fetch(),
      newTodo: "",
      editedTodo: null,
      visibility: "all"
    };
  },

  // watch todos change for localStorage persistence
  watch: {
    todos: {
      handler(todos) {
        store.save(todos);
      },
      deep: true
    }
  },
  computed: {
    filteredTodos() {
      return filters[this.visibility](this.todos);
    },
    remaining() {
      return filters.active(this.todos).length;
    },
    allDone: {
      get() {
        return this.remaining === 0;
      },
      set(value) {
        this.todos.forEach(function(todo) {
          todo.completed = value;
        });
      }
    }
  },

  filters: {
    pluralize(n) {
      return n <= 1 ? "task" : "tasks";
    }
  },
  methods: {
    addTodo() {
      var value = this.newTodo && this.newTodo.trim();
      if (!value) {
        return;
      }
      this.todos.push({
        id: store.uid++,
        title: value,
        completed: false
      });
      this.newTodo = "";
    },
    removeTodo(todo) {
      this.todos.splice(this.todos.indexOf(todo), 1);
    },
    markAsCompleted(todo) {
      todo.completed = !todo.completed;
    },
    editTodo(todo) {
      this.beforeEditCache = todo.title;
      this.editedTodo = todo;
    },

    doneEdit(todo) {
      if (!this.editedTodo) {
        return;
      }
      this.editedTodo = null;
      todo.title = todo.title.trim();
      if (!todo.title) {
        this.removeTodo(todo);
      }
    },

    cancelEdit(todo) {
      this.editedTodo = null;
      todo.title = this.beforeEditCache;
    },

    removeCompleted() {
      this.todos = filters.active(this.todos);
    }
  }
};

// handle routing
function onHashChange() {
  var visibility = window.location.hash.replace(/#\/?/, "");
  if (filters[visibility]) {
    app.visibility = visibility;
  } else {
    window.location.hash = "";
    app.visibility = "all";
  }
}

window.addEventListener("hashchange", onHashChange);
onHashChange();
</script>

<style lang="less" scoped>
.header {
  &:before {
    content: "";
    display: block;
    position: absolute;
    top: 13px;
    left: 13px;
    width: 65px;
    height: 65px;
    z-index: 1;
    opacity: 1;
  }
}

.todo {
  padding: 13px;
  margin: 32% 0 10% 0;
  position: relative;
  color: #fff;
}

.todo input::-webkit-input-placeholder,
.todo input::-moz-placeholder,
.todo input::input-placeholder {
  font-weight: 400;
  color: #ddd;
}

.new-todo,
.edit {
  position: relative;
  margin: 0;
  width: 100%;
  font-size: 24px;
  font-weight: bold;
  font-family: inherit;
  font-weight: inherit;
  line-height: 1.4em;
  border: 0;
  box-sizing: border-box;
  &:focus {
    color: rgb(20, 19, 19);
  }
}

.new-todo {
  padding: 16px;
  border: none;
  background: #fff;
  transition: background 0.3s ease;
  box-shadow: 0px 0px 0px 6px rgba(255, 255, 255, 0.3);
}

.new-todo-button {
  position: absolute;
  display: inline-block;
  width: 65px;
  height: 65px;
  right: 13px;
  top: 13px;
  transition: opacity 0.3s ease;
}

.main {
  position: relative;
  z-index: 2;
  color: #fff;
  margin-top: 11%;
  box-shadow: 0px 0px 0px 6px rgba(255, 255, 255, 0.3);
}

.completed-wrapper {
  border-bottom: 1px solid #e6e6e6;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.toggle-all {
  width: 1px;
  height: 1px;
  border: none; /* Mobile Safari */
  opacity: 0;
  position: absolute;
  right: 100%;
  bottom: 100%;
}

.toggle-all + label {
  font-size: 12px;
  top: 13px;
  left: 13px;
  z-index: 99;
  cursor: pointer;
  transition: color 0.3s ease;
  margin-left: 5px;
  &.un-mute {
    i {
      color: #fff;
    }
  }
  i {
    font-size: 15px;
    margin: 0 6px;
    color: #ccc;
  }
  &:hover {
    color: #ffffff;
  }
}

.toggle-all + label:before {
  font-size: 22px;
  opacity: 0.2;
  width: 57px;
  height: 40px;
  display: inline-block;
  vertical-align: middle;
}

.toggle-all:checked + label:before {
  opacity: 1;
}

.todo-list {
  margin: 13px 0;
  padding: 0;
  list-style: none;
}

.todo-list li {
  position: relative;
  font-size: 24px;
  border-radius: 10px;
  transition: background 0.3s ease;
}

.todo-list li:last-child {
  border-bottom: none;
}

.todo-list li.editing {
  border-bottom: none;
  padding: 0;
  background: #fff;
}

.todo-list li.editing .edit {
  display: block;
  width: calc(100% - 43px);
  padding: 12px 16px;
  margin: 0 0 0 43px;
}

.todo-list li.editing .view {
  display: none;
}

.todo-list li .toggle {
  text-align: center;
  width: 40px;
  height: auto;
  position: absolute;
  top: 13px;
  bottom: 0;
  margin: auto 0;
  border: none;
  font-size: 31px;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  color: #fff;
}

.todo-list li label {
  word-break: break-all;
  padding: 15px 15px 15px 60px;
  display: block;
  line-height: 1.2;
  transition: color 0.4s;
}

.todo-list li.completed label {
  text-decoration: line-through;
}

.todo-list li .remove-btn {
  display: none;
  position: absolute;
  top: 16px;
  right: 10px;
  bottom: 0;
  /* width: 40px; */
  /* height: 40px; */
  margin: auto 0;
  /* font-size: 30px; */
  color: #fff;
  margin-bottom: 11px;
  -webkit-transition: color 0.2s ease-out;
  transition: color 0.2s ease-out;
  transition: color 0.2s ease-out;
  cursor: pointer;
}

.todo-list li {
  &:hover {
    .remove-btn {
      color: #ccc;
      display: block;
    }
  }
}

.todo-list li .edit {
  display: none;
}

.footer {
  color: #ccc;
  padding: 10px 0 0 17px;
  border-top: 1px solid #e6e6e6;
  display: flex;
  justify-content: space-between;
}

.todo-count {
  text-align: left;
}

.todo-count strong {
  font-weight: bold;
  color: #fff;
}

.filters {
  margin: 0;
  padding: 0;
  list-style: none;
  right: 0;
  left: 0;
  li {
    display: inline-block;
    padding: 0 3px;
    a {
      color: #ccc;
      text-decoration: none;
      &:hover,
      &.selected {
        color: #fff;
        font-weight: 700;
      }
    }
  }
}

.clear-completed {
  display: inline-block;
  line-height: 40px;
  color: #fff;
  padding: 0 5px;
}

.clear-completed,
html .clear-completed:active {
  text-decoration: none;
  cursor: pointer;
}

.clear-completed:hover {
  color: #000;
}
</style>
