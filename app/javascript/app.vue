<template>
  <draggable v-model="lists" :options="{group: 'lists'}" @end="listMoved" class="board dragArea">
    <list v-for="(list, index) in lists" :list="list"></list>
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
      lists: this.original_lists
    }
  },
  methods: {
    listMoved (event) {
      var data = new FormData
      data.append("list[position]", event.newIndex + 1)

      Rails.ajax({
        url: `lists/${this.lists[event.newIndex].id}/move`,
        type: "PATCH",
        data: data,
        dataType: "json"
      })
    }
  }
}
</script>

<style scoped>
.dragArea {
  min-height: 20px;
}

.board {
  overflow-x: scroll;
  white-space: nowrap;
}
</style>
