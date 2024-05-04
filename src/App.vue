<script setup>
import { ref, reactive } from 'vue'
import { VueDraggable } from 'vue-draggable-plus'

// Data for the board
const data = ref([
  {
    'title': 'Todo',
    'cards': [
      { name: 'Task 1', tags: ['requirement-changes', 'halfway-done', 'test', 're-test', 'waiting-for-test', 'done']},
      { name: 'Task 2', tags: ['done', 'waiting-for-test'] },
      { name: 'Task 3', tags: [] }
    ]
  },
  {
    'title': 'Doing',
    'cards': [
      { name: 'Task 4', tags: ['test', 'halfway-done'] },
      { name: 'Task 5', tags: ['done', 'test'] }
    ]
  },
  {
    'title': 'Done',
    'cards': [
      { name: 'Task 6', tags: ['requirement-changes', 're-test'] }
    ]
  }
])

// list functions
function addList() {
  data.value.push({
    'title': 'New List',
    'cards': []
  })
}
function removeList(index) {
  data.value.splice(index, 1);
}

// card functions
function addCard(index) {
  data.value[index]['cards'].push({
    name: 'New Task',
    tags: []
  })
}
function removeCard(listIndex, cardIndex) {
  data.value[listIndex].cards.splice(cardIndex, 1);
}

// tags color
function getTagColor(name, tag) {
  if (name == 'test') {
    return '#065535';
  }
  if (name == 'requirement-changes') {
      return '#003366';
  }
  if (name == 're-test') {
      return '#b29600';
  }
  if (name == 'done') {
      return '#800000';
  }
  if (name == 'halfway-done') {
      return '#800080';
  }
  if (name == 'waiting-for-test') {
      return '#088da5';
  }
  return '#e1e1e1';
}

// init data
const editStates = ref(data.value.map(() => false))
const showDialog = ref(false)
const currentCard = reactive({ name: '', tags: [] })
let currentListIndex = ref(-1)
let currentItemIndex = ref(-1)

// dialog functions
function openEditDialog(item, listIndex, itemIndex) {
  currentCard.name = item.name
  currentCard.tags = item.tags.join(',');
  currentListIndex.value = listIndex
  currentItemIndex.value = itemIndex
  showDialog.value = true
}
function saveChanges() {
  if (currentListIndex.value !== -1 && currentItemIndex.value !== -1) {
    const item = data.value[currentListIndex.value].cards[currentItemIndex.value]
    item.name = currentCard.name
    item.tags = currentCard.tags != '' ? currentCard.tags.split(',').map(tag => tag.trim()) : [];
    closeDialog()
  }
}
function closeDialog() {
  showDialog.value = false
}
</script>

<template>
  <button @click="addList" class="add-list-button">+ Add List</button>
  <VueDraggable v-model="data" class="board">
    <div v-for="(list, index) in data" :key="index" class="list">
      <div class="list-header-container">
        <div 
          v-if="!editStates[index]"
          @click="editStates[index] = true"
          class="list-header"
        >
          {{ list.title }}
        </div>
        <input 
          v-else
          v-model="list.title"
          class="list-header-input"
          @blur="editStates[index] = false"
          @keyup.enter="editStates[index] = false"
        />
        <span>
          <button @click="addCard(index)" class="add-card-button">+ Add Card</button>
          <button @click="removeList(index)" class="delete-list-button">Delete List</button>
        </span>
      </div>
      <VueDraggable
        v-model="data[index]['cards']"
        class="cards"
        group="people"
      >
        <div
          v-for="(item, itemIndex) in data[index]['cards']"
          :key="item.id"
          class="card"
          @click="openEditDialog(item, index, itemIndex)"
        >
          <button @click.stop="removeCard(index, itemIndex)" class="delete-card-button">X</button>
          {{ item.name }}
          <div class="card-tags">
            <span v-for="tag in item.tags" :style="{ backgroundColor: getTagColor(tag) }" :key="tag" class="tag">{{ tag }}</span>
          </div>
        </div>
      </VueDraggable>
    </div>
  </VueDraggable>
  <pre>
    {{ data }}
  </pre>
  <div v-if="showDialog" class="dialog-overlay" @click.self="closeDialog">
    <div class="dialog" @click.stop>
      <h3>Edit Card</h3>
      <div class="input-label">
        <label for="cardName">Name:</label>
        <input type="text" id="cardName" v-model="currentCard.name"/>
      </div>
      <div class="input-label">
        <label for="cardTags">Tags (comma-separated):</label>
        <input type="text" id="cardTags" v-model="currentCard.tags"/>
      </div>
      <button @click="saveChanges" class="button">Save</button>
      <button @click="closeDialog" class="button">Close</button>
    </div>
  </div>
</template>

<style scoped>
.list-header-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.list-header {
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
}

.list-header-input {
  width:100px;
  font-size: 16px;
  font-weight: bold;
  flex-grow: 1;
  margin-right: 10px; /* Ensure some space between input and button */
}

.add-card-button {
  padding: 5px 10px;
  background-color: #5aac44;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}
.delete-list-button {
  padding: 5px 10px;
  background-color: red;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  margin-left: 10px;
}

.board {
  display: flex;
  flex-wrap: nowrap;
  overflow-x: auto;
  padding: 20px;
  background-color: #0079bf; /* Trello background color */
  height: 100vh;
}

.list {
  flex: 0 0 300px;
  background-color: #ebecf0;
  margin-right: 10px;
  border-radius: 3px;
  padding: 10px;
  height: 90%; /* slightly less than the viewport height for aesthetics */
  overflow-y: auto;
}

.cards {
  min-height: 50px; /* ensures that dragging is easier even if the list is empty */
}

.card {
  position: relative;
  background-color: #fff;
  border-radius: 3px;
  padding: 10px;
  margin-bottom: 8px;
  box-shadow: 0 1px 0 rgba(9, 30, 66, 0.25);
  cursor: pointer;
  user-select: none;
}
.card-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 10px;
}
.tag {
  background-color: #e1e1e1;
  border-radius: 3px;
  padding: 2px 5px;
  font-size: 12px;
}
.add-list-button {
  height: 40px;
  background-color: #ffffff66;
  border: none;
  color: #333;
  font-size: 16px;
  cursor: pointer;
  border-radius: 3px;
  padding: 0 10px;
}
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.dialog {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  z-index: 10;
}
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.dialog {
  background: white;
  padding: 20px;
  width: 300px; /* 控制对话框宽度 */
  border-radius: 10px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.25);
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.dialog h3 {
  margin-top: 0;
  color: #333;
}

.input-label {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.input-label input {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 14px;
}

.button {
  padding: 10px 20px;
  background-color: #007BFF;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.button:hover {
  background-color: #0056b3;
}
.delete-card-button {
  position: absolute;
  top: 5px;
  right: 5px;
  padding: 2px 5px;
  background-color: #ff6347; /* Tomato red */
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 12px;
}
</style>