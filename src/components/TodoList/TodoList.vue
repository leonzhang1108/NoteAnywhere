<style lang="stylus">
  @import 'todo-list'
</style>

<template>
  <div ref="todoListPage">
    <h1 style="text-align: center; width: 100%;">NoteAnywhere</h1>
    <div class="weui-cell weui-cell_vcode">
      <div class="weui-cell__bd">
        <input id="newContent"
               @input="filterInput"
               @keydown="addKeyDown"
               class="weui-input"
               type="text"
               placeholder="add & filter">
      </div>
      <div class="weui-cell__ft">
        <button @click="addToContent"
                class="weui-vcode-btn">add</button>
      </div>
    </div>
  
    <div v-show="isInEdit"
         @click="clickToSee"
         class="mask"
         :class="isInEdit?'in-edit':''"></div>
    <div class="todo-form">
      <transition-group name="item-change"
                        tag="div">
        <div v-for="(item, index) in todoList"
             class="todo-menu-item"
             :key="item"
             :index="index"
             :class="[{done: item.checked}, {editable: item.editable}, {invisible: !item.seen}]"
             @click="clickToEdit"
             @touchmove="touchmove"
             @touchstart="touchstart"
             @touchend="touchend">
          <div v-if="item.checked"
               class="item-check item-side"><img class="side-img"
                 :src="undo"></div>
          <div v-else
               class="item-check item-side"><img class="side-img"
                 :src="check"></div>
          <div class="item-content">
            <span class="content-show"
                  v-show="!item.editable">{{item.content}}</span>
            <textarea class="content-input"
                      style="resize:none"
                      @keydown="editKeyDown"
                      @click="clickToStop"
                      @touchmove="textAreaPrevent"
                      @touchstart="textAreaPrevent"
                      @touchend="textAreaPrevent"
                      v-show="item.editable"
                      v-model="item.content"
                      rows="4"></textarea>
          </div>
          <div class="item-delete item-side"><img class="side-img"
                 :src="cross"></div>
        </div>
      </transition-group>
    </div>
  </div>
</template>
<script>
import 'weui'
import check from 'assets/check.png'
import cross from 'assets/cross.png'
import undo from 'assets/undo.png'
export default {
  data() {
    var list = (localStorage.getItem('datas') && JSON.parse(localStorage.getItem('datas')).length > 0)
      ? JSON.parse(localStorage.getItem('datas'))
      : [
        {
          content: 'how it works:',
          checked: false,
          editable: false
        }, {
          content: 'click item to edit, completed item not allowed',
          checked: false,
          editable: false
        }, {
          content: 'edit box to add item or filter list',
          checked: false,
          editable: false
        }, {
          content: 'slide item from left to right, complete or undo it',
          checked: false,
          editable: false
        }, {
          content: 'slide item from right to left, delete it',
          checked: false,
          editable: false
        }
      ]

    list.forEach(item => {
      item.seen = true
    })
    // this.sortList(list)

    return {
      todoList: list,
      check: check,
      cross: cross,
      undo: undo
    }
  },
  computed: {
    isInEdit: function () {
      return this.todoList.some(item => {
        return item.editable
      })
    }
  },
  created() {
  },
  methods: {
    textAreaPrevent(e) {
      e.stopPropagation()
    },
    addKeyDown(e) {
      if (e.keyCode == 13) this.addToContent()
    },
    editKeyDown(e) {
      if (e.keyCode == 13) this.clickToSee()
    },
    showIcon(v, e) {
      var offset = Number(v)
      var $parent = $(e.target).parents('.todo-menu-item')
      var $check = $parent.find('.item-check')
      var $delete = $parent.find('.item-delete')
      if (offset > 80) {
        $check.css({ "opacity": 1, left: -offset + 'px' })
        $delete.css({ opacity: 0 })
      } else if (offset < -80) {
        $delete.css({ "opacity": 1, right: offset + 'px' })
        $check.css({ opacity: 0 })
      } else {
        if (offset > 0) {
          $check.css({ "opacity": Math.abs(offset) / 80 })
        } else {
          $delete.css({ "opacity": Math.abs(offset) / 80 })
        }
        $check.css({ left: -offset + 'px' })
        $delete.css({ right: offset + 'px' })
      }
    },
    allCanSeen() {
      this.todoList.forEach((item, index) => {
        item.seen = true
      })
    },
    filterInput() {
      var value = $('#newContent').val().trim()
      this.todoList.forEach((item, index) => {
        item.seen = true
        if (item.content.indexOf(value) < 0) item.seen = false
      })
    },
    setLocalstorage() {
      localStorage.setItem('datas', JSON.stringify(this.todoList))
    },
    addToContent() {
      $('.weui-cell.weui-cell_vcode').removeClass('invalid')
      if ($('#newContent').val().trim()) {
        this.todoList.unshift({
          content: $('#newContent').val(),
          checked: false,
          editable: false,
          seen: true
        })
        $('#newContent').val('')
      } else {
        $('.weui-cell.weui-cell_vcode').addClass('invalid')
      }
      this.allCanSeen()
      this.setLocalstorage()
    },
    clickToEdit(e) {
      var $this = $(e.target)
      var $parent = $this.parents('.todo-menu-item')
      if ($parent.hasClass('done')) return
      $('body').scrollTop(0)
      this.clickToSee()
      var index = $parent.attr('index')
      this.todoList[index].editable = true
      $this.siblings('input').show().focus()
      var margin = 100 - Number($this.offset().top)
      $parent.css({ 'margin-top': margin + 'px' })
      $parent.next().css({ 'margin-top': -margin + 'px' })
      e.preventDefault()
    },
    clickToSee() {
      if (this.todoList.some(item => {
        return !item.content
      })) return

      this.todoList.forEach(item => {
        item.editable = false
      })
      $('.todo-menu-item').css({ 'margin-top': 0 })
      this.setLocalstorage()
    },
    clickToStop(e) {
      e.stopPropagation()
    },
    findTarget(e) {
      return $(e.target).hasClass('item-content') ? $(e.target) : $(e.target).parents('.item-content')
    },
    touchstart(e) {
      if ($('.clicked').length) return
      this.timeOutEvent = setTimeout(() => {
        this.longPress(e)
      }, 500)
      var $this = this.findTarget(e)
      var $parent = $this.parent()
      $this.data('currentLocation', e.targetTouches[0].pageX)
      $parent.addClass('clicked')
      // e.preventDefault()
    },
    touchmove(e) {
      if (this.timeOutEvent) {
        clearTimeout(this.timeOutEvent)
        this.timeOutEvent = 0
      }
      var $this = this.findTarget(e)
      if ($this.parent().hasClass('sorting')) {
        console.log('sorting')
      } else if ($this.parent().hasClass('clicked')) {
        var originPageX = Number($this.data('currentLocation'))
        var currentPageX = e.targetTouches[0].pageX
        var translateX = currentPageX - originPageX
        if (translateX < -80) translateX = (-80 + translateX) / 2
        if (translateX > 80) translateX = (80 + translateX) / 2
        if ($this[0]) $this[0].parentNode.style.WebkitTransform = "translateX(" + translateX + "px)"
        $this.data('movingTranslateX', translateX)
        this.showIcon(translateX, e)
      }
    },
    touchend(e) {
      if (this.timeOutEvent) {
        clearTimeout(this.timeOutEvent)
        this.timeOutEvent = 0
      }
      var $this = this.findTarget(e)
      var $parent = $this.parent()
      $parent.removeClass('clicked').removeClass('sorting')
      var tlx = Number($this.data('movingTranslateX'))
      if (tlx <= -80) {
        this.removeItem($this, $parent)
        return
      } else if (tlx >= 80) {
        this.completeUndoItem($this, $parent)
      }

      $parent.find('.item-check').css({ opacity: 0 })
      $parent.find('.item-delete').css({ opacity: 0 })
      if ($this[0]) $this[0].parentNode.style.WebkitTransform = "translateX(0)"
    },
    longPress(e) {
      this.timeOutEvent = 0
      console.log('long press')
      $(e.target).parents('.todo-menu-item').addClass('sorting')
    },
    removeItem($this, $parent) {
      $this.data('movingTranslateX', 0)
      var index = $parent.attr('index')
      $parent.addClass('removing')
      this.todoList.splice(index, 1)
      if ($this[0]) $this[0].parentNode.style.WebkitTransform = "translateX(-" + (Number($('body').width()) + 180) + "px)"
      $parent.find('.item-delete').css({ opacity: 1 })
      setTimeout(() => {
        $parent.removeClass('removing')
        this.setLocalstorage()
      }, 900)
    },
    sortList(list) {
      list.sort((a, b) => {
        if (a.checked - b.checked) return (a.checked - b.checked)
        return -1
      })
    },
    completeUndoItem($this, $parent) {
      $this.data('movingTranslateX', 0)
      var index = $parent.attr('index')
      if (this.todoList[index] && this.todoList[index].content) {
        this.todoList[index].checked = !this.todoList[index].checked
        setTimeout(() => {

          // this.sortList(this.todoList)

          if (this.todoList[index].checked) {
            this.todoList.push(this.todoList.splice(index, 1)[0])
          } else {
            this.todoList.unshift(this.todoList.splice(index, 1)[0])
          }
          this.setLocalstorage()
        }, 500)
      }
    }
  }
}
</script>