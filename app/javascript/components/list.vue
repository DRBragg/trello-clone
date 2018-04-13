<template>
  <div class="list">
    <h6>{{ list.name }}</h6>
    <hr />

    <draggable v-model="list.cards" :options="{group: 'cards'}" @change="cardMoved" class="dragArea">
      <div v-for="(card, index) in list.cards" class="card card-body mb-3">
        {{ card.name }}
      </div>
    </draggable>

    <div class="mb-1">
      <textarea v-model="message" class="form-control mb-1"></textarea>
      <button @click="submitMessage(list.id)" class="btn btn-primary">Add</button>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'

export default {
  props: ['list'],
  components: { draggable },
  data: function() {
    return {
      message: ""
    }
  },
  methods: {
    submitMessage () {
      var data = new FormData
      data.append("card[list_id]", this.list.id)
      data.append("card[name]", this.message)

      Rails.ajax({
        url: "/cards",
        type: "POST",
        data: data,
        dataType: "json",
        success: (data) => {
          const index = window.store.lists.findIndex(item => item.id == list_id)
          window.store.lists[index].cards.push(data)
          this.message = ""
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

.list {
  background-color: #E2E4E6;
  border-radius: 3px;
  padding: 10px;
  width: 270px;
  margin-right: 20px;
  vertical-align: top;
  display: inline-block;
}
</style>
