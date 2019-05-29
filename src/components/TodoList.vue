<template>
    <div class="todoList">
        <input type="text" class="todo-input" 
        placeholder="What needs to be done??"
        v-model="newTodo"
        @keyup.enter="addTodo">
        <div class="bar">
            <div class="progress" v-if="progressPercent > 0"
            v-bind:style="{'background':'#ec407a',
            'width':progressPercent+'%'}">
                <!-- <span class="percent">{{progressPercent}}%</span> -->
            </div>            
        </div>
        <draggable v-model="todos">
        <transition-group name="fade" enter-active-class="animated fadeInUp"
        leave-active-class="animated fadeOutDown">
            <div v-for="(todo, index) in todosFiltered" 
            :key="todo.id" class="todo-item">
                <div class="todo-item-left">
                    <input type="checkbox" 
                    v-model="todo.completed">
                    <div v-if="!todo.editing" 
                    @dblclick="editTodo(todo)" 
                    class="todo-item-label" :class="{ completed : todo.completed }">
                        {{todo.title}}
                    </div>
                    <input v-else @blur="doneEdit(todo)"
                    @keyup.enter="doneEdit(todo)"
                    @keyup.esc="cancelEdit(todo)"
                    class="todo-item-edit" 
                    type="text" 
                    v-model="todo.title" v-focus>
                </div>
                <div class="remove-item" @click="removeTodo(index)">
                    &times;
                </div>                
            </div>    
        </transition-group>
        </draggable>
        <div class="extra-container">
            <div>
                <label>
                    <input type="checkbox"
                    :checked="!anyRemaining"
                    @change="checkAllTodos">
                    Check All
                </label>
            </div>
            <div>{{ remaining }} item(s) left</div>
        </div>
        <div class="extra-container">
            <div>
                <button :class="{ active: filter == 'all' }" 
                @click="filter= 'all'">All</button>
                <button :class="{ active: filter == 'active' }" 
                @click="filter= 'active'">Active</button>
                <button :class="{ active: filter == 'completed' }" 
                @click="filter= 'completed'">Completed</button>
            </div>
            <div>
                <transition name="fade">
                    <button v-if="showClearCompletedButton"
                    @click="clearCompleted">Clear completed</button>
                </transition>                
            </div>
        </div>
    </div>
</template>

<script>
import draggable from 'vuedraggable'

export default {
  name: 'todo-list',
  data () {
    return {
      newTodo: "",
      beforeEditCache: '',
      filter: 'all',
      idForTodo: 3,
      todos: [
        {
            'id': 1,
            'title': 'Finish Vue',            
            'completed': false,
            'editing': false
        },
        {
            'id': 2,
            'title': 'tratata',
            'completed': false,
            'editing': false
        }
      ]
    }
  },
  computed: {
      remaining() {
          return this.todos.filter(todo => !todo.completed).length
      },
      anyRemaining() {
          return this.remaining != 0
      },
      todosFiltered() {
          if (this.filter == 'all') {
              return this.todos
          } else if (this.filter == 'active') {
              return this.todos.filter(todo => !todo.completed)
          } else if (this.filter == 'completed') {
              return this.todos.filter(todo => todo.completed)
          }
          return this.todos;
      },
      showClearCompletedButton() {
          return this.todos.filter(todo => todo.completed).length > 0
      },
      progressPercent() {
          return Math.round(this.todos.filter(todo => todo.completed).length
          / this.todos.length * 100)
      }
  },
  directives: {
      focus: {
        inserted: function (el) {
            el.focus()
        }
      }
  },
  methods: {
      addTodo() {
        if(this.newTodo.trim().length == 0){
            return
        }
        this.todos.push({
            id: this.idForTodo,
            title: this.newTodo,
            completed: false
        })

        this.newTodo = ''
        this.idForTodo++
      },
      removeTodo(index){
        this.todos.splice(index, 1)
      },
      editTodo(todo) {
        this.beforeEditCache = todo.title
        todo.editing = true
      },
      doneEdit(todo) {
        if(this.title.trim() == ''){
            todo.title = this.beforeEditCache
        }
        todo.editing = false
      },
      cancelEdit(todo){
        todo.title = this.beforeEditCache
        todo.editing = false
      },
      checkAllTodos() {
          this.todos.forEach((todo) => todo.completed = event.target.checked)
      },
      clearCompleted() {
          this.todos = this.todos.filter(todo => !todo.completed)
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
    @import 'animate.css';
    // @import 'node_modules/bootstrap/scss/bootstrap';
    // @import 'node_modules/bootstrap-vue/src/index.scss';

    .todoList {
        width: 100%;
        align-items: center;
    }

    .todo-input {
        width: 100%;
        padding: 10px 18px;
        font-size: 18px;
        margin-bottom: 16px;

        &:focus{
            outline: 0;
        }
    }

    .todo-item {
        margin-bottom: 12px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        animation-duration: 0.3s;
    }

    .remove-item {
        cursor: pointer;
        margin-left: 14px;
        &:hover{
            color: black;
        }
    }

    .todo-item-left {
        display: flex;
        align-items: center;
    }

    .todo-item-label {
        padding: 10px;
        border: 1px solid white;
        margin-left: 12px;
        word-break: break-all; 
        word-wrap: break-word;
    }

    .todo-item-edit {
        font-size: 24px;
        color: #2c3e50;
        margin-left: 12px;
        width: 100%;
        padding: 10px;
        border: 1px solid #ccc;
        font-family: 'Avenir', Helvetica, Arial, sans-serif;

        &:focus {
            outline: none;
        }
    }

    .completed {
        text-decoration: line-through;
        color: gray;
    }

    .extra-container {
        display: flex;
        align-items: center;
        justify-content: space-between;
        font-size: 16px;
        border-top: 1px solid lightgray;
        padding-top: 14px;
        margin-bottom: 14px;
    }

    button {
        font-size: 14px;
        background-color: white;
        appearance: none;

        &:hover {
            background: lightgreen;
        }

        &:focus {
            outline: none;
        }
    }

    .active {
        background: lightgreen;
    }
    
    .fade-enter-active, .fade-leave-active {
        transition: opacity 0.2s;
    }

    .fade-enter, .fade-leave {
        opacity: 0;
    }

    .bar{
	width: 100%;
    background:#dfdfdf;
    height: 15px;
	overflow: hidden;
	padding:5px;
    }

    .progress{
        // align-items: center;
        float:left;
        padding:2px;
    }

    .percent{
        float: right;
        // align-items: center;
        font-weight: 600;
        height: 30px;
        line-height: 30px;
    }

//     @-webkit-keyframes fadeInUp {
//   from {
//     opacity: 0;
//     -webkit-transform: translate3d(0, 100%, 0);
//     transform: translate3d(0, 100%, 0);
//   }

//   to {
//     opacity: 1;
//     -webkit-transform: translate3d(0, 0, 0);
//     transform: translate3d(0, 0, 0);
//   }
// }

// @keyframes fadeInUp {
//   from {
//     opacity: 0;
//     -webkit-transform: translate3d(0, 100%, 0);
//     transform: translate3d(0, 100%, 0);
//   }

//   to {
//     opacity: 1;
//     -webkit-transform: translate3d(0, 0, 0);
//     transform: translate3d(0, 0, 0);
//   }
// }

// .fadeInUp {
//   -webkit-animation-name: fadeInUp;
//   animation-name: fadeInUp;
// }

// @-webkit-keyframes fadeOutDown {
//   from {
//     opacity: 1;
//   }

//   to {
//     opacity: 0;
//     -webkit-transform: translate3d(0, 100%, 0);
//     transform: translate3d(0, 100%, 0);
//   }
// }

// @keyframes fadeOutDown {
//   from {
//     opacity: 1;
//   }

//   to {
//     opacity: 0;
//     -webkit-transform: translate3d(0, 100%, 0);
//     transform: translate3d(0, 100%, 0);
//   }
// }

// .fadeOutDown {
//   -webkit-animation-name: fadeOutDown;
//   animation-name: fadeOutDown;
// }
</style>
