<template lang="pug">
  #app
    .contain
      .top
        .title ToDo List 😖
        .inputContain
          input(v-model="todoString"
                placeholder="enter your task"
                @keyup.enter="addTodoItem")
      .options
        .btn(@click="show('all')" :class="{ active: state === 'all' }") All
        .btn(@click="show('active')" :class="{ active: state === 'active' }") Active
        .btn(@click="show('finished')" :class="{ active: state === 'finished' }") Finished
        .btn(@click="removeAllFinished" v-if="currentList.length && state === 'finished'") Clear
        span(style="color: red; margin-left: auto") {{currentList.length}}
        span Task
      .todoWrap
        p(v-if="!currentList.length") There is nothing.
        transition-group(name="list")
          TodoItem(v-for="(obj, index) in currentList"
                  :key="index"
                  :todoObject="obj"
                  :currentSourceElement="dragSourceElement"
                  @finish="completeTodo(obj)"
                  @remove="removeTodoItem(obj)"
                  @drag="getCurrentDragObj(obj, ...arguments)"
                  @drop="dropObj(obj)")
</template>

<script>

import TodoItem from './components/TodoItem'

export default {
  name: 'todo',
  components: {
    TodoItem
  },
  data () {
    return {
      todoString: '',
      todoList: [],
      state: 'all',
      dragSourceElement: null,
      currentDragIndex: null
    }
  },
  computed:{
    currentList () {
      return this.filterTodos(this.state)
    }
  },
  methods: {
    addTodoItem () {
      if (this.todoString.length) {
        let item = {
          text: this.todoString,
          isFinished: false
        }
        this.todoList.push(item)
        this.todoString = ''
      }
    },
    removeTodoItem (obj) {
      const index = this.todoList.indexOf(obj)
      this.todoList.splice(index, 1)
    },
    removeAllFinished () {
      const deleteIndex = []
      this.currentList.forEach(obj => {
        deleteIndex.push(this.todoList.indexOf(obj))
      })
      for (let i = deleteIndex.length - 1; i >= 0; i--) {
        this.todoList.splice(deleteIndex[i], 1)
      }
    },
    completeTodo (obj) {
      obj['isFinished'] = !obj['isFinished']
    },
    filterTodos (v) {
      switch (v) {
        case 'all':
          return this.todoList
          break
        case 'active':
          return this.todoList.filter(el => {
            return !el.isFinished
          })
          break
        case 'finished':
          return this.todoList.filter(el => {
            return el.isFinished
          })
          break
      }
    },
    show (filter) {
      this.state = filter
    },
    getCurrentDragObj (obj, el) {
      this.dragSourceElement = el
      this.currentDragIndex = this.todoList.indexOf(obj)
    },
    dropObj (obj) {
      const dropIndex = this.todoList.indexOf(obj)
      // console.log(`drag ${this.currentDragIndex} to ${dropIndex}`)
      this.todoList.splice(dropIndex, 0, this.todoList.splice(this.currentDragIndex, 1)[0])
      this.dragSourceElement = null
      this.currentDragIndex = null
    }
  }
}
</script>

<style lang="sass">

@import 'sass/variables'

html, body
  margin: 0
  padding: 0
  background-color: $background-color

#app
  font-family: 'Avenir', Helvetica, Arial, sans-serif
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale
  padding: 4em 0
  display: flex
  justify-content: center

.contain
  width: 80%
  max-width: 768px
  min-width: 310px
  box-shadow: 8px 14px 38px rgba(39,44,49,.06), 1px 3px 8px rgba(39,44,49,.03)
  .top
    padding: $padding-size-l
    background-color: $primary-color
    border-radius: 12px 12px 0 0
    .title
      color: $title-color
      font-size: 2em
      font-weight: bold
    input
      margin-top: 20px
      width: 100%
      border-radius: 6px
      padding: 0.8em
      box-sizing: border-box
      border: none
      &:focus
        outline: none
        color: #444
  .options
    display: flex
    align-items: flex-start
    background-color: $secondary-color
    padding: $padding-size-m
    .btn
      color: #cf8d2e
      font-weight: bold
      margin-right: 16px
      cursor: pointer
      transition-duration: .2s
    .active
      color: $title-color
    span
      padding-right: 10px
      font-weight: bold

.todoWrap p
  text-align: center
  font-style: oblique

.list-enter-active, .list-leave-active
  transition: opacity .2s !important
.list-enter, .list-leave-to
  opacity: 0

</style>
