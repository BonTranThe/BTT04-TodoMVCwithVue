<template>
  <div id="todoapp">
    <h1 class="first-title">todos</h1>
    <i
      v-show="todos.length"
      class="fa-solid fa-angle-down"
      :class="{ tick_all: remaining === 0 }"
      @click="toggleTickAll"
    ></i>
    <header class="header">
      <input
        class="new-todo"
        v-model="newTodo"
        type="text"
        placeholder="What needs to be done?"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="main">
      <ul class="todo-list">
        <li
          class="todo"
          v-for="(todo, index) in todoss"
          :key="todo.id"
          :class="{ status_checked: todo.status }"
        >
          <div class="view">
            <input
              ref="inputtingCheck"
              :class="{ index }"
              type="checkbox"
              v-model="todo.status"
            />
            <input
              ref="valueEdit"
              type="text"
              :class="{ checked: todo.status === true }"
              :value="todo.content"
              :id="index"
              @dblclick="editTodo(index)"
              @keyup.enter="doneEdit(todo, index)"
              @blur="doneEdit(todo, index)"
              @keyup.escape="cancelEdit(index)"
              readOnly
            />
            <i
              ref="trashClose"
              class="fas fa-trash-alt"
              :class="{ index }"
              @click="deleteTask(todo)"
            ></i>
          </div>
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length">
      <div class="controls">
        <span
          ><strong>{{ remaining }} </strong>
          <span class="item_left"
            >{{
              remaining === 0 || remaining === 1 ? "item" : "items"
            }}
            left</span
          ></span
        >
        <div class="filters">
          <span
            ref="filter"
            id="all"
            @click="setVisibility('all')"
            :class="{ active: visibility === 'all' }"
            >All</span
          >
          <span
            ref="filter"
            id="active"
            @click="setVisibility('active')"
            :class="{ active: visibility === 'active' }"
            >Active</span
          >
          <span
            ref="filter"
            id="completed"
            @click="setVisibility('completed')"
            :class="{ active: visibility === 'completed' }"
            >Completed</span
          >
        </div>
        <button
          @click="removeCompleted"
          class="clear-btn"
          :class="{ showClear: todos.length <= remaining }"
        >
          Clear Completed
        </button>
      </div>
    </footer>
  </div>
</template>
<script>
const STORAGE_KEY = "vue-todomvc";
// interface Todo {
//   id: number
//   content: string
//   status: true | false
// }
const filters = {
  all: function (todos) {
    return todos;
  },
  active: function (todos) {
    return todos.filter(function (todo) {
      return !todo.status;
    });
  },
  completed: function (todos) {
    return todos.filter(function (todo) {
      return todo.status;
    });
  },
};

export default {
  data() {
    return {
      newTodo: "",
      todos: JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]"),
      visibility: "all",
    };
  },

  watch: {
    todos: {
      handler(todos) {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
      },
      deep: true,
    },
  },

  methods: {
    setVisibility(visibility) {
      this.visibility = visibility;
    },

    toggleTickAll() {
      const todos2 = this.todos.filter((todo) => todo.status === true);
      if (todos2.length === this.todos.length) {
        this.todos.forEach((item) => {
          item.status = false;
        });
      } else {
        this.todos.forEach((item) => {
          item.status = true;
        });
      }
    },

    addTodo() {
      if (this.newTodo.length === 0 || !this.newTodo.trim()) {
        return;
      } else {
        this.todos.push({
          id: Date.now(),
          content: this.newTodo,
          status: false,
        });
      }
      this.newTodo = "";
    },

    deleteTask(todo) {
      this.todos.splice(this.todos.indexOf(todo), 1);
    },

    editTodo(index) {
      //Access to input[index] (content)
      this.hide(index);
      let edit = this.$refs.valueEdit[index];
      edit.readOnly = false;
      edit.classList.remove("checked");
      edit.setSelectionRange(edit.value.length, edit.value.length);
      this.beforeEdit = edit.value;
      this.onstyleInput(index);
    },

    doneEdit(todo, index) {
      var afterEdit = this.$refs.valueEdit[index];
      var input = this.$refs.inputtingCheck[index];
      todo.content = afterEdit.value && afterEdit.value.trim();
      if (todo.content === "") {
        this.deleteTask(todo);
        afterEdit.readOnly = true;
      } else {
        afterEdit.readOnly = true;
      }
      if (input.checked) {
        afterEdit.classList.add("checked");
      }
      this.offstyleInput(index);
      this.show(index);
    },

    cancelEdit(index) {
      var afterEdit = this.$refs.valueEdit[index];
      afterEdit.value = this.beforeEdit;
      afterEdit.readOnly = true;
      this.offstyleInput(index);
      this.show(index);
    },

    removeCompleted() {
      this.todos = filters.active(this.todos);
    },

    hide(index) {
      var input = this.$refs.inputtingCheck[index];
      var trash = this.$refs.trashClose[index];
      input.classList.add("hide");
      trash.classList.add("hide");
    },

    show(index) {
      var input = this.$refs.inputtingCheck[index];
      var trash = this.$refs.trashClose[index];
      input.classList.remove("hide");
      trash.classList.remove("hide");
    },

    //input show style when edit todo
    onstyleInput(index) {
      var afterEdit = this.$refs.valueEdit[index];
      afterEdit.style.outline = "0.5px solid gray";
      afterEdit.style.borderRadius = "2px";
      afterEdit.style.marginLeft = "54px";
      afterEdit.style.boxShadow = "1px 1px 1px 2px gray";
    },

    //input hide style when edit todo
    offstyleInput(index) {
      var afterEdit = this.$refs.valueEdit[index];
      afterEdit.style.outline = "none";
      afterEdit.style.marginLeft = "20px";
      afterEdit.style.boxShadow = "none";
    },
  },

  computed: {
    todoss() {
      return this.todos.filter((todo) => {
        switch (this.visibility) {
          case "active":
            return !todo.status;
          case "completed":
            return todo.status;
          default:
            return this.todos;
        }
      });
    },

    remaining() {
      return filters.active(this.todos).length;
    },
  },
};
</script>
<style>
#todoapp {
  max-width: 500px;
  margin: 0 auto;
  background: #fff;
  border-radius: 7px;
}

.first-title {
  min-height: 30px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #f82052;
  text-transform: uppercase;
  font-size: 60px;
  opacity: 0.8;
}

.header {
  max-width: 100%;
  display: flex;
  justify-content: space-between;
}

.new-todo {
  position: relative;
  z-index: 0;
  width: 500px;
  height: 50px;
  border-radius: 5px;
  border: 0.5px solid #c7939f;
  padding-left: 57px;
  font-size: 20px;
  text-transform: uppercase;
  outline: none;
}

.fa-solid {
  position: absolute;
  z-index: 1;
  width: 35px;
  height: 35px;
  font-size: 35px;
  text-align: center;
  opacity: 0.5;
  top: 10rem;
  left: 42.4rem;
  cursor: pointer;
}

.main {
  max-width: 500px;
}

.main ul {
  max-width: 100%;
  margin-left: -36px;
}

.main ul li {
  list-style: none;
  max-width: 100%;
  height: 40px;
  vertical-align: middle;
  margin-bottom: 10px;
  padding-bottom: 10px;
  border-bottom: 0.5px solid #9c9c9c;
}

.main ul li .view {
  max-width: 100%;
  display: flex;
  justify-content: space-between;
  height: 100%;
}

.main ul li .view input[type="checkbox"] {
  width: 25px;
  height: 25px;
  vertical-align: middle;
  margin: auto 5px;
  cursor: pointer;
}

.main ul li .view input[type="text"] {
  width: 400px;
  height: 40px;
  margin-left: 0px;
  font-size: 20px;
  text-transform: uppercase;
  font-weight: 500;
  margin: auto 20px;
  outline: none;
  border: none;
}

.main ul li .view i {
  margin: auto 5px;
  width: 26px;
  height: 26px;
  font-size: 26px;
  transition: 0.5s all;
  opacity: 0.7;
  cursor: pointer;
}

.main ul li .view i:hover {
  transform: scale(1.1);
  opacity: 1;
}

.footer {
  max-width: 480px;
  height: 50px;
  margin-left: 10px;
}

.controls {
  display: inline-flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  height: 100%;
}

strong {
  font-size: 18px;
}

.item_left {
  font-size: 18px;
}

.filters {
  width: 200px;
}
.filters span {
  cursor: pointer;
  margin: 0 4px;
  font-size: 18px;
  height: 20px;
}

.filters span.active {
  color: #be1ddf;
  border: 1px solid #ebb7c4;
  padding: 1px 3px;
  border-radius: 3px;
  font-weight: 600;
}

.controls .clear-btn {
  outline: none;
  border: none;
  font-size: 14px;
  color: #fff;
  padding: 7px 9px;
  border-radius: 4px;
  background: #dd70cf;
  cursor: pointer;
  transition: 0.5s all;
}
.controls .clear-btn:hover {
  background: #d637c1;
}

.checked {
  text-decoration: line-through;
  opacity: 0.5;
}

.hide {
  display: none;
}

.tick_all {
  opacity: 1;
}

.showClear {
  opacity: 0;
}
</style>
