<script setup>
import { ref, reactive, computed } from 'vue'
import login from './components/login.vue'
import works from './components/works.vue'
import my_orders from './components/my_orders.vue'
import authorization from './components/authorization.vue'

const activee = ref(1)
const modal_open = ref(false)
const message_open = ref(false)
const text = ref("")
const isAuthenticated = ref(false)

const data = reactive({
  obj: {
    name: "",
    gender: "",
    age: "",
    balance: 0,
    card: null,
    cart: [],
  }
})
function active1() {
  activee.value = 1;
}

function active6() {
  activee.value = 6;
}

function active4() {
  if (data.obj.name === "") {
    text.value = "Чтобы перейти на эту страницу, нужно в начале зарегестрироваться"
    modal_open.value = true
  } else {
    activee.value = 4;
  }
}

function active5() {
  if (data.obj.name === "") {
    text.value = "Чтобы перейти на эту страницу, нужно в начале зарегестрироваться"
    modal_open.value = true
  } else {
    activee.value = 5;
  }
}

function addToCart(book) {
  const existingItem = data.obj.cart.find(item => item.title === book.title && item.author === book.author)

  if (existingItem) {
    existingItem.quantity += 1
  } else {
    data.obj.cart.push({
      ...book,
      quantity: 1,
      id: Date.now() + Math.random()
    })
  }

  text.value = `"${book.title}" добавлена в корзину!`
  modal_open.value = true
}

function removeFromCart(itemId) {
  data.obj.cart = data.obj.cart.filter(item => item.id !== itemId)
}

function close_message() {
  message_open.value = false
}

function handleLoginSuccess(userData) {
  data.obj.name = userData.name
  data.obj.gender = userData.gender
  data.obj.age = userData.age
  isAuthenticated.value = true
  activee.value = 5
}

function appData(inf) {
  data.obj.name = inf.name
  data.obj.age = inf.age
  data.obj.gender = inf.gender
  isAuthenticated.value = true
  activee.value = 5;
}

function exit(output) {
  data.obj.name = ""
  data.obj.age = ""
  data.obj.gender = ""

  isAuthenticated.value = false
  activee.value = 6
}

function del(keyToDelete) {
  deleteModal.show = true;
  deleteModal.orderKey = keyToDelete;
  deleteModal.reason = null;
  deleteModal.rating = null;
}

function confirmDelete() {
  if (deleteModal.reason === 'completed') {
    if (!copywriterName.value || !deleteModal.rating) return;

    data.obj.reviews.push({
      copywriter: copywriterName.value,
      rating: deleteModal.rating,
      date: new Date().toISOString(),
      orderKey: deleteModal.orderKey
    });

    copywriterName.value = '';
  }

  data.obj.new_orders = data.obj.new_orders.filter(
    order => order.key !== deleteModal.orderKey
  );

  deleteModal.show = false;
}

const copywriterName = ref('');
const availableCopywriters = computed(() => {
  return [...new Set(data.obj.reviews.map(r => r.copywriter))];
});

const deleteModal = reactive({
  show: false,
  orderKey: null,
  reason: null,
  rating: null
});

function updateBalance(newBalance) {
  data.obj.balance = newBalance;
}

function updateCard(cardDetails) {
  data.obj.card = cardDetails;
}
</script>

<template>
  <div class="block">
    <div class="logo-section">
      <div class="book-image-container">
        <img src="../src/assets/sova.png" class="book-image">
      </div>
      <div class="title-shop">
        bookie
      </div>
    </div>
    <div class="navigation-buttons">
      <button v-if="data.obj.name" @click="active4" :class="['btn', { active: activee === 4 }]">Книги</button>
      <button v-if="data.obj.name" @click="active5" :class="['btn', { active: activee === 5 }]">Личный кабинет</button>
      <button v-if="data.obj.name == ''" @click="active1"
        :class="['btn', { active: activee === 1 }]">Регистрация</button>
      <button v-if="data.obj.name == ''" @click="active6"
        :class="['btn', { active: activee === 6 }]">Авторизация</button>
    </div>
  </div>

  <login v-if="activee === 1" @actionOne="appData" />
  <authorization v-if="activee === 6" @loginSuccess="handleLoginSuccess" />
  <my_orders v-if="activee === 5" :data="data.obj" :order="data.obj.new_orders" :userName="data.obj.name"
    :deleteOrder="del" @actionTwo="exit" @removeFromCart="removeFromCart" @updateBalance="updateBalance"
    @updateCard="updateCard" />
  <works v-if="activee === 4" :rezumes="data.obj.new_rezumes" @actionSix="addToCart" />
  <message v-if="message_open" :interlocutor="data.obj.interlocutor"
    :messages="data.obj.messages[data.obj.interlocutor] || []" :currentUser="data.obj.name" @modulClose="close_message"
    @sendMessage="receiveMessage" />
</template>

<style>
body,
html {
  background: linear-gradient(70deg, rgb(243, 164, 100), rgb(238, 177, 97));
  background-size: cover;
}
</style>

<style scoped>
* {
  font-family: DejaVu Sans Mono, monospace;
  font-size: 1.2rem;
  color: rgb(37, 14, 11);
}

.block {
  display: flex;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 80px;
  background: linear-gradient(rgb(169, 54, 28), rgba(255, 192, 203, 0));
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  box-sizing: border-box;
  z-index: 1000;
}


.logo-section {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 1;
}

.navigation-buttons{
  display: flex;
  margin-left: -100%;
  align-items: center;
  justify-content: center; 
  flex: 1;
}

.btn {
  background-color: #e3bebe83;
  color: #603018;
  margin: 10px;
  height: 35%;

}

.btn.active {
  background-color: #772c1be7;
  color: white;
  margin: 10px;
}

.btn:hover {
  background-color: #f2b195;
  border-color: #f2b195;
}

.hidden {
  display: none;
}

.delete-reasons {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin: 20px 0;
}

.delete-reasons label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.rating-section {
  margin-top: 20px;
}

.rating-select {
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
  font-size: 16px;
}

.confirm-btn {
  background-color: #4CAF50;
  color: white;
  margin-right: 10px;
}

.cancel-btn {
  background-color: #f44336;
  color: white;
}

.delete-reasons {
  display: flex;
  gap: 15px;
  margin: 20px 0;
  justify-content: center;
}

.reason-btn {
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

.reason-btn:hover {
  background: #45a049;
}

.rating-stars {
  font-size: 24px;
  color: #ccc;
  cursor: pointer;
  margin-top: 10px;
}

.rating-stars span {
  transition: color 0.2s;
}

.rating-stars span.active {
  color: #ffc107;
}

.rating-stars span:hover {
  color: #ffc107;
}

.copywriter-input {
  padding: 8px;
  margin: 10px 0;
  width: 100%;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.confirm-rating-btn {
  padding: 10px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

.confirm-rating-btn:disabled {
  background: #cccccc;
  cursor: not-allowed;
}

.book-image-container {
  display: flex;
  align-items: center;
}

.book-image {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  object-fit: cover;
}

.title-shop {
  font-size: 1.5rem;
  font-weight: bold;
  color: rgb(37, 14, 11);
  white-space: nowrap;
}
</style>