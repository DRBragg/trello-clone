<template>
  <div class="list">
    <h6>{{ list.name }}</h6>
    <hr />

    <draggable v-model="list.cards" :options="{group: 'cards'}" @change="cardMoved" class="dragArea">
      <card v-for="(card, index) in list.cards" :card="card" :list="list"></card>
    </draggable>

    <a v-if="!editing" @click="startEditing">Add a Card</a>
    <div v-else class="mb-1">
      <textarea v-model="message" ref="message" class="form-control mb-1"></textarea>
      <button @click="createCard(list.id)" class="btn btn-primary">Add</button>
      <a @click="editing=false">Cancel</a>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import card from 'components/card'

export default {
  props: ['list'],
  components: { draggable, card },
  data: function() {
    return {
      editing: false,
      message: ""
    }
  },
  methods: {
    startEditing () {
      this.editing = true
      this.$nextTick(() => { this.$refs.message.focus() })
    },
    createCard () {
      var data = new FormData
      data.append("card[list_id]", this.list.id)
      data.append("card[name]", this.message)

      Rails.ajax({
        url: "/cards",
        type: "POST",
        data: data,
        dataType: "json",
        success: (data) => {
          this.message = ""
          this.$nextTick(() => { this.$refs.message.focus() })
        }
      })
    },
    cardMoved (event) {
      const evt = event.added || event.moved

      if (evt == undefined) { return }

      const element = evt.element
      const list_index = window.store.lists.findIndex((list) => {
        return list.cards.find((card) => {
          return card.id === element.id
        })
      })

      var data = new FormData
      data.append("card[list_id]", window.store.lists[list_index].id)
      data.append("card[position]", evt.newIndex + 1)

      Rails.ajax({
        url: `/cards/${element.id}/move`,
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
</style>
