<template>
  <draggable v-model="lists" :options="{group: 'lists'}" @end="listMoved" class="board dragArea">
    <list v-for="(list, index) in lists" :list="list"></list>
    <a v-if="!editing" @click="startEditing" class="list">Add a List</a>
    <div v-else class="list">
      <textarea v-model="message" ref="message" class="form-control mb-1"></textarea>
      <button @click="submitMessage()" class="btn btn-primary">Add</button>
      <a @click="editing=false">Cancel</a>
    </div>
  </draggable>
</template>

<script>
import draggable from 'vuedraggable'
import list from 'components/list'

export default {
  props: ["original_lists"],
  components: { draggable, list },
  data: function () {
    return {
      editing: false,
      message: "",
      lists: this.original_lists
    }
  },
  methods: {
    startEditing () {
      this.editing = true
      this.$nextTick(() => { this.$refs.message.focus() })
    },
    listMoved (event) {
      var data = new FormData
      data.append("list[position]", event.newIndex + 1)

      Rails.ajax({
        url: `lists/${this.lists[event.newIndex].id}/move`,
        type: "PATCH",
        data: data,
        dataType: "json"
      })
    },
    submitMessage () {
      var data = new FormData
      data.append("list[name]", this.message)

      Rails.ajax({
        url: "/lists",
        type: "POST",
        data: data,
        dataType: "json",
        success: (data) => {
          window.store.lists.push(data)
          this.message = ""
          this.editing = false
        }
      })
    }
  }
}
</script>

<style scoped>
.dragArea {
  min-height: 10px;
}
.board {
  overflow-x: auto;
  white-space: nowrap;
}
.list {
  background: #E2E4E6;
  border-radius: 3px;
  display: inline-block;
  margin-right: 20px;
  padding: 10px;
  vertical-align: top;
  width: 270px;
}
</style>
