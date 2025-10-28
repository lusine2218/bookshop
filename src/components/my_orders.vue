<script setup>
import { reactive, ref, computed } from 'vue'

const props = defineProps({
   data: Object,
   order: Array,
   userName: String,
   deleteOrder: Function,
})
const emit = defineEmits(['actionTwo', 'actionThree', 'actionFive', 'updateRezume', 'openMessages', 'updateCard', 'updateBalance', 'removeFromCart'])
const output = ref(false)
const addAmount = ref(100)
const showCardModal = ref(false)
const showOrderModal = ref(false)

const orderData = reactive({
   fullName: "",
   email: "",
   phone: "",
   deliveryPoint: "",
   cardNumber: "",
   cardHolder: "",
   expiryDate: "",
   cvv: ""
})

const deliveryPoints = [
   "Москва, ул. Тверская, д. 10",
   "Москва, пр. Мира, д. 25",
   "Москва, ул. Арбат, д. 15",
   "Санкт-Петербург, Невский пр., д. 30",
   "Санкт-Петербург, ул. Садовая, д. 12",
   "Екатеринбург, ул. Ленина, д. 45"
]

const cartTotal = computed(() => {
   return props.data.cart.reduce((total, item) => {
      const price = parseInt(item.pay) || 0
      return total + (price * item.quantity)
   }, 0)
})

function updateQuantity(itemId, newQuantity) {
   if (newQuantity < 1) {
      removeFromCart(itemId)
      return
   }

   const item = props.data.cart.find(item => item.id === itemId)
   if (item) {
      item.quantity = newQuantity
   }
}

function removeFromCart(itemId) {
   emit('removeFromCart', itemId)
}

function openOrderModal() {
   showOrderModal.value = true
}

function validateOrderForm() {
   let isValid = true

   if (!orderData.fullName.trim()) {
      alert('Пожалуйста, введите ФИО')
      isValid = false
   }

   const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
   if (!emailRegex.test(orderData.email)) {
      alert('Пожалуйста, введите корректный email')
      isValid = false
   }

   const phoneRegex = /^[\+]?[0-9\s\-\(\)]{10,15}$/
   if (!phoneRegex.test(orderData.phone.replace(/\s/g, ''))) {
      alert('Пожалуйста, введите корректный номер телефона')
      isValid = false
   }

   if (!orderData.deliveryPoint) {
      alert('Пожалуйста, выберите пункт выдачи')
      isValid = false
   }

   if (!/^\d{16}$/.test(orderData.cardNumber.replace(/\s/g, ''))) {
      alert('Номер карты должен содержать 16 цифр')
      isValid = false
   }

   if (!orderData.cardHolder.trim()) {
      alert('Введите имя владельца карты')
      isValid = false
   }

   if (!/^(0[1-9]|1[0-2])\/?([0-9]{2})$/.test(orderData.expiryDate)) {
      alert('Формат срока действия: MM/YY')
      isValid = false
   }

   if (!/^\d{3}$/.test(orderData.cvv)) {
      alert('CVV должен содержать 3 цифры')
      isValid = false
   }

   return isValid
}

function processOrder() {
   if (!validateOrderForm()) {
      return
   }

   const orderDetails = {
      items: [...props.data.cart],
      total: cartTotal.value,
      customer: {
         fullName: orderData.fullName,
         email: orderData.email,
         phone: orderData.phone
      },
      delivery: orderData.deliveryPoint,
      payment: {
         cardNumber: orderData.cardNumber.slice(-4),
         amount: cartTotal.value,
         method: 'card'
      },
      orderDate: new Date().toISOString(),
      orderId: 'ORD-' + Date.now()
   }

   console.log('Заказ оформлен:', orderDetails)
   setTimeout(() => {

      props.data.cart = []

      showOrderModal.value = false

      resetOrderForm()

      alert(`Заказ успешно оформлен! Номер заказа: ${orderDetails.orderId}\n`)
   }, 1000)
}

function resetOrderForm() {
   orderData.fullName = ""
   orderData.email = ""
   orderData.phone = ""
   orderData.deliveryPoint = ""
   orderData.cardNumber = ""
   orderData.cardHolder = ""
   orderData.expiryDate = ""
   orderData.cvv = ""
}

function formatPhone(value) {
   const numbers = value.replace(/\D/g, '')
   if (numbers.length <= 1) return numbers
   if (numbers.length <= 4) return `+7 (${numbers.slice(1, 4)}`
   if (numbers.length <= 7) return `+7 (${numbers.slice(1, 4)}) ${numbers.slice(4, 7)}`
   if (numbers.length <= 9) return `+7 (${numbers.slice(1, 4)}) ${numbers.slice(4, 7)}-${numbers.slice(7, 9)}`
   return `+7 (${numbers.slice(1, 4)}) ${numbers.slice(4, 7)}-${numbers.slice(7, 9)}-${numbers.slice(9, 11)}`
}

function handlePhoneInput(event) {
   orderData.phone = formatPhone(event.target.value)
}

function addFunds() {
   if (addAmount.value < 100) {
      alert('Минимальная сумма пополнения - 100 ₽')
      return
   }

   emit('updateBalance', props.data.balance + addAmount.value)
   addAmount.value = 100
   showCardModal.value = false
}

function exit() {
   output.value = true
   emit('actionTwo', output)
   console.log('Выход из системы')
}
</script>

<template>
   <div class="gridd">
      <div class="square">
         <h3>Мой профиль</h3>
         <div class="profile-info">
            <p><strong>Имя:</strong> {{ data.name || 'Не указано' }}</p>
            <p><strong>Пол:</strong> {{ data.gender || 'Не указан' }}</p>
            <p><strong>Возраст:</strong> {{ data.age || 'Не указан' }}</p>
         </div>
         <button @click="exit" class="btn">Выйти</button>
      </div>

      <div class="square cart-section">
         <h3>Корзина</h3>
         <div v-if="data.cart && data.cart.length > 0" class="cart-items">
            <div v-for="item in data.cart" :key="item.id" class="cart-item">
               <div class="cart-item-info">
                  <p class="cart-item-title"><strong>{{ item.title }}</strong></p>
                  <p class="cart-item-author">Автор: {{ item.author }}</p>
                  <p class="cart-item-price">{{ item.pay }}</p>
               </div>
               <div class="cart-item-controls">
                  <div class="quantity-controls">
                     <button @click="updateQuantity(item.id, item.quantity - 1)" class="quantity-btn">-</button>
                     <span class="quantity">{{ item.quantity }}</span>
                     <button @click="updateQuantity(item.id, item.quantity + 1)" class="quantity-btn">+</button>
                  </div>
                  <button @click="removeFromCart(item.id)" class="btn small remove-btn">Удалить</button>
               </div>
            </div>
            <div class="cart-total">
               <p><strong>Итого: {{ cartTotal }} ₽</strong></p>
               <button @click="openOrderModal" class="btn checkout-btn">Оформить заказ</button>
            </div>
         </div>
         <div v-else class="empty-cart">
            <p>Корзина пуста</p>
         </div>
      </div>

      <div class="square none">
         <h3>Мой баланс</h3>
         <div class="balance-display">
            {{ data.balance || 0 }} ₽
         </div>
         <button @click="showCardModal = true" class="btn">Пополнить баланс</button>
      </div>

      <div v-if="showOrderModal" class="modal-overlay">
         <div class="modal-content order-modal">
            <h3>Оформление заказа</h3>

            <div class="order-summary">
               <h4>Ваш заказ:</h4>
               <div v-for="item in data.cart" :key="item.id" class="order-item">
                  <span>{{ item.title }} (x{{ item.quantity }})</span>
                  <span>{{ (parseInt(item.pay) * item.quantity) }} ₽</span>
               </div>
               <div class="order-total">
                  <strong>Итого: {{ cartTotal }} ₽</strong>
               </div>
            </div>

            <div class="form-section">
               <h4>Данные получателя</h4>
               <div class="form-group">
                  <label>ФИО *</label>
                  <input v-model="orderData.fullName" type="text" placeholder="Иванов Иван Иванович" required>
               </div>

               <div class="form-group">
                  <label>Email для чека *</label>
                  <input v-model="orderData.email" type="email" placeholder="example@mail.ru" required>
               </div>

               <div class="form-group">
                  <label>Телефон *</label>
                  <input v-model="orderData.phone" type="tel" placeholder="+7 (999) 999-99-99" @input="handlePhoneInput"
                     required>
               </div>
            </div>

            <div class="form-section">
               <h4>Пункт выдачи</h4>
               <div class="form-group">
                  <label>Выберите пункт выдачи *</label>
                  <select v-model="orderData.deliveryPoint" required>
                     <option value="">Выберите пункт выдачи</option>
                     <option v-for="point in deliveryPoints" :key="point" :value="point">
                        {{ point }}
                     </option>
                  </select>
               </div>
            </div>

            <div class="form-section">
               <h4>Данные карты</h4>
               <div class="form-group">
                  <label>Номер карты *</label>
                  <input v-model="orderData.cardNumber" type="text" placeholder="1234 5678 9012 3456" maxlength="19"
                     @input="orderData.cardNumber = orderData.cardNumber.replace(/\D/g, '').replace(/(\d{4})/g, '$1 ').trim()"
                     required>
               </div>

               <div class="form-group">
                  <label>Имя владельца *</label>
                  <input v-model="orderData.cardHolder" type="text" placeholder="IVAN IVANOV" required>
               </div>

               <div class="form-row">
                  <div class="form-group">
                     <label>Срок действия (MM/YY) *</label>
                     <input v-model="orderData.expiryDate" type="text" placeholder="MM/YY" maxlength="5"
                        @input="orderData.expiryDate = orderData.expiryDate.replace(/\D/g, '').replace(/(\d{2})(\d{0,2})/, '$1/$2')"
                        required>
                  </div>

                  <div class="form-group">
                     <label>CVV *</label>
                     <input v-model="orderData.cvv" type="password" placeholder="123" maxlength="3"
                        @input="orderData.cvv = orderData.cvv.replace(/\D/g, '')" required>
                  </div>
               </div>
            </div>

            <div class="modal-actions">
               <button @click="showOrderModal = false" class="btn cancel">Отмена</button>
               <button @click="processOrder" class="btn confirm-btn">Оплатить {{ cartTotal }} ₽</button>
            </div>
         </div>
      </div>

      <div v-if="showCardModal" class="modal-overlay">
         <div class="modal-content">
            <h3>Пополнение баланса</h3>
            <div class="form-group">
               <label>Сумма пополнения</label>
               <input v-model.number="addAmount" type="number" min="100" class="width">
            </div>
            <div class="modal-actions">
               <button @click="showCardModal = false" class="btn cancel">Отмена</button>
               <button @click="addFunds" class="btn">Пополнить</button>
            </div>
         </div>
      </div>
   </div>
</template>

<style scoped>
.gridd {
   display: grid;
   grid-template-columns: repeat(2, 1fr);
   gap: 50px;
}

.square {
   width: 270px;
   height: auto;
   color: black;
   background: linear-gradient(to bottom left, rgba(183, 84, 45, 0.5), rgba(250, 222, 167, 0.5));
   border-radius: 5%;
   padding: 20px;
   margin-top: 50px;
}

.create {
   width: 270px;
   height: auto;
   color: black;
   background: linear-gradient(to bottom left, rgb(183, 45, 45, 0.5), rgb(250, 167, 181, 0.5));
   border-radius: 25%;
   padding: 20px;
   margin-top: 50px;
}

.form-group {
   width: 100%;
   margin-bottom: 10px;
}

.label {
   text-align: center;
   margin-bottom: 5px;
}

.width {
   width: 60%;
   height: 25px;
   padding: 8px;
   box-sizing: border-box;
}

.error {
   text-align: center;
   color: red;
   font-size: 1em;
   margin-top: 3px;
}

input[type=text] {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
}

input[type=text]:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

input[type=number] {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
}

input[type=number]:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

input[type=date] {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
}

input[type=date]:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

input[type="password"] {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
}

input[type="password"]:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

.card-preview {
   background: linear-gradient(135deg, #3a4a6b, #1e2b4d);
   color: white;
   padding: 15px;
   border-radius: 10px;
   margin-bottom: 15px;
   font-family: 'Courier New', monospace;
}

textarea {
   background-color: #f1bf7e;
   border-radius: 20px 20px 20px 20px;
   border-color: #831010;
   color: #954b1e;
   padding: 5%;
}

textarea:focus {
   background-color: rgb(189, 114, 102);
   color: #832510;
}

.balance-display {
   font-size: 2rem;
   font-weight: bold;
   text-align: center;
   margin: 20px 0;
   color: #2c3e50;
}

.add-funds {
   margin-top: 20px;
   padding: 15px;
   background: rgba(255, 255, 255, 0.2);
   border-radius: 10px;
}

.funds-input {
   width: 100%;
   padding: 8px;
   margin: 10px 0;
   border-radius: 5px;
   border: 1px solid #ccc;
}

.modal-overlay {
   position: fixed;
   top: 0;
   left: 0;
   right: 0;
   bottom: 0;
   background-color: rgba(0, 0, 0, 0.5);
   display: flex;
   justify-content: center;
   align-items: center;
   z-index: 1000;
}

.modal-content {
   background-color: white;
   padding: 20px;
   border-radius: 10px;
   width: 400px;
   max-width: 90%;
}

.modal-actions {
   display: flex;
   justify-content: space-between;
   margin-top: 20px;
}

.btn.cancel {
   background-color: #f44336;
}

.balance-display {
   font-size: 2rem;
   font-weight: bold;
   text-align: center;
   margin: 20px 0;
   color: #503c2c;
}

.btn {
   width: 150px;
   height: auto;
   background-color: #e3bebe83;
   color: #601818;
}

.btn:hover {
   background-color: #771b1be7;
   color: white;
}

.cart-section {
   min-height: 400px;
}

.cart-items {
   max-height: 300px;
   overflow-y: auto;
}

.cart-item {
   display: flex;
   justify-content: space-between;
   align-items: center;
   padding: 10px;
   border-bottom: 1px solid #ddd;
   margin-bottom: 10px;
}

.cart-item-info {
   flex: 1;
}

.cart-item-title {
   margin: 0;
   font-size: 1em;
}

.cart-item-author {
   margin: 2px 0;
   font-size: 0.9em;
   color: #666;
}

.cart-item-price {
   margin: 2px 0;
   font-weight: bold;
   color: #a34821;
}

.cart-item-controls {
   display: flex;
   flex-direction: column;
   align-items: center;
   gap: 5px;
}

.quantity-controls {
   display: flex;
   align-items: center;
   justify-content: center;
   gap: 5px;
}

.quantity-btn {
   width: 45px;
   height: 25px;
   border: 1px solid #a34821;
   background: white;
   border-radius: 4px;
   cursor: pointer;
   text-align: center;
   padding: 0 10px;
}

.quantity {
   padding: 0 10px;
   font-weight: bold;
   min-width: 20px;
   text-align: center;
}

.remove-btn {
   padding: 2px 10px;
   font-size: 0.8em;
}

.cart-total {
   margin-top: 15px;
   padding-top: 15px;
   border-top: 2px solid #a34821;
   text-align: center;
}

.checkout-btn {
   margin-top: 10px;
   background-color: #a04f2c;
}

.checkout-btn:hover {
   background-color: #6f311e;
}

.empty-cart {
   text-align: center;
   color: #666;
   padding: 20px;
}

.order-modal {
   max-width: 600px;
   max-height: 90vh;
   overflow-y: auto;
}

.order-summary {
   background: #f9f9f9;
   padding: 15px;
   border-radius: 8px;
   margin-bottom: 20px;
   border-left: 4px solid #a34821;
}

.order-item {
   display: flex;
   justify-content: space-between;
   padding: 5px 0;
   border-bottom: 1px solid #eee;
}

.order-total {
   display: flex;
   justify-content: space-between;
   margin-top: 10px;
   padding-top: 10px;
   border-top: 2px solid #a34821;
   font-size: 1.1em;
}

.form-section {
   margin-bottom: 25px;
   padding-bottom: 15px;
   border-bottom: 1px solid #eee;
}

.form-section h4 {
   color: #a34821;
   margin-bottom: 15px;
   font-size: 1.1em;
}

.form-row {
   display: grid;
   grid-template-columns: 1fr 1fr;
   gap: 15px;
}

.form-group {
   margin-bottom: 15px;
}

.form-group label {
   display: block;
   margin-bottom: 5px;
   font-weight: bold;
   color: #333;
}

.form-group input,
.form-group select {
   width: 100%;
   padding: 10px;
   border: 1px solid #ddd;
   border-radius: 5px;
   font-size: 14px;
   box-sizing: border-box;
}

.form-group input:focus,
.form-group select:focus {
   outline: none;
   border-color: #a34821;
   box-shadow: 0 0 0 2px rgba(163, 72, 33, 0.1);
}

.confirm-btn {
   background-color: #2c5aa0;
}

.confirm-btn:hover {
   background-color: #1e3d6f;
}

.none {
   display: none;
}
</style>
