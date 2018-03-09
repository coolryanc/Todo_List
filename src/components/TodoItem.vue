<template lang="pug">
  .item(draggable="true")
    .check(:class="{ complete: todoObject.isFinished }" @click="emitParent('finish')") ✔
    label(v-show = "!isEdit" @dblclick="isEdit = true") {{todoObject.text}}
    input(v-show = "isEdit"
          v-model = "todoObject.text"
          @keyup.enter = "isEdit = false"
          @keyup.esc = "isEdit = false")
    span(@click="emitParent('remove')") X
</template>

<script>
export default {
  name: 'TodoItem',
  props: {
    todoObject: Object,
    currentSourceElement: HTMLDivElement
  },
  data () {
    return {
      isEdit: false
    }
  },
  computed: {
    element () {
      return this.$el
    }
  },
  mounted () {
    const element = this.element
    this.addDragDropHandlers(element)
  },
  methods: {
    emitParent (action) {
      switch (action) {
        case 'remove':
          this.$emit('remove')
          break
        case 'finish':
          this.$emit('finish')
          break
        case 'drag':
          this.$emit('drag', this.element)
          break
        case 'drop':
          this.$emit('drop')
          break
      }
    },
    addDragDropHandlers (elem) {
      elem.addEventListener('dragstart', this.handleStart, false)
      elem.addEventListener('dragover', this.handleOver, false)
      elem.addEventListener('dragleave', this.handleLeave, false)
      elem.addEventListener('drop', this.handleDrop, false)
      elem.addEventListener('dragend', this.handleEnd, false)
    },
    handleStart (e) {
      this.emitParent('drag')
      e.dataTransfer.effectAllowed = 'move'
      this.element.classList.add('drag')
    },
    handleOver (e) {
      if (e.preventDefault) {
        e.preventDefault()
      }
      this.element.classList.add('over')
      e.dataTransfer.dropEffect = 'move'
      return false
    },
    handleLeave (e) {
      //pass
      this.element.classList.remove('over')
    },
    handleDrop (e) {
      if (e.stopPropagation) {
        e.stopPropagation()
      }
      if (this.currentSourceElement != this.element) {
        console.log('drop')
        this.emitParent('drop')
      }
      this.element.classList.remove('over')
      return false
    },
    handleEnd (e) {
      //pass
      this.element.classList.remove('drag')
      this.element.classList.remove('over')
    }
  }
}
</script>

<style lang="sass">

@import '../sass/variables'

.item
  display: flex
  align-items: center
  height: 80px
  width: 100%
  background-color: $third-color
  border-bottom: solid 1px rgba(black, 0.1)
  transition: .2s
  z-index: 999
  position: relative
  cursor: move
  &:hover
    height: 100px
    width: calc(100% + 20px)
    transform: translate(-10px, -1px)
    box-shadow: 0 0 1px rgba(39,44,49,.1), 0 3px 16px rgba(39,44,49,.07)
    z-index: 999
    span
      opacity: 1
      line-height: 100px
  .check
    color: $text-color
    padding: 0 32px
    font-size: 1.5em
    cursor: pointer
    transition: .2s
  .complete
    color: $finished-color !important
  label
    margin-left: 20px
    cursor: pointer
    &:hover
      &::after
        opacity: 1
    &::after
      opacity: 0
      content: 'double click to edit'
      display: inline-block
      margin-left: 10px
      padding: 2px 5px
      color: $title-color
      background-color: $text-color
      border-radius: 5px
  input
    margin-left: 20px
    font-size: 1em
    width: 50%
    border: none
    border-bottom: 1px solid $primary-color
    z-index: 999
    &:focus
      outline: none
      color: #444
  span
    position: absolute
    cursor: pointer
    font-size: 1em
    text-align: center
    background-color: red
    color: $title-color
    line-height: 80px
    padding: 0 12px
    height: 100%
    right: 0
    opacity: 0.2
    transition: .2s

.drag
  background-color: #eeefef
.over
  border-top: 4px solid rgb(135, 170, 230)

</style>
