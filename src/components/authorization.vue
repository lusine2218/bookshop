<script setup>
import { ref, reactive } from 'vue'
import bcrypt from 'bcryptjs'

const error = ref("")
const emit = defineEmits(['loginSuccess'])

const data = reactive({
  name: "",
  password: "",
})

function login() {
  if (data.name.trim() === "") {
    error.value = "Введите имя пользователя"
    return
  }
  if (data.password === "") {
    error.value = "Введите пароль"
    return
  }

  const users = JSON.parse(localStorage.getItem('users')) || []

  const user = users.find(u => u.name === data.name)

  if (!user) {
    error.value = "Пользователь не найден"
    return
  }

  if (!bcrypt.compareSync(data.password, user.password)) {
    error.value = "Неверный пароль"
    return
  }

  emit('loginSuccess', {
    name: user.name,
    gender: user.gender,
    age: user.age,
    role: user.role
  })

  data.name = ""
  data.password = ""
  error.value = ""
}
</script>

<template>
  <div class="cont">
    <h2>Авторизация</h2>

    <div class="form-group">
      <div class="label">Имя</div>
      <input type="text" v-model="data.name" class="width" />
    </div>

    <div class="form-group">
      <div class="label">Пароль</div>
      <input type="password" v-model="data.password" class="width" />
    </div>

    <div class="error">{{ error }}</div>

    <button @click="login" class="btn">Войти</button>
  </div>
</template>

<style scoped>
.cont {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 250px;
  margin: 0 auto;
  border: solid #411a10 1px;
  padding: 20px;
  background-color: #b5653467;
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
  width: 100%;
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

.radio-group {
  display: flex;
  justify-content: center;
}

input[type=text] {
  background-color: #f1bf7e;
  border-color: #831010;
  color: #954b1e;
}

input[type=text]:focus {
  background-color: rgb(189, 138, 102);
  color: #832510;
}

input[type=password] {
  background-color: #f1bf7e;
  border-color: #831010;
  color: #954b1e;
}

input[type=password]:focus {
  background-color: rgb(189, 121, 102);
  color: #832510;
}

.hidden-radio {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

.custom-radio {
  display: inline-block;
  padding: 10px 20px;
  background-color: #e3bebe83;
  color: #601818;
  border-radius: 5px;
  cursor: pointer;
  margin: 5px;
  transition: background-color 0.3s, color 0.3s;
}

.hidden-radio:checked+.custom-radio {
  background-color: #771b1be7;
  color: white;
}

.custom-radio:hover {
  background-color: #e7967f;
}

.btn {
  background-color: #e3bebe83;
  color: #601818;
  padding: 10px 20px;
  cursor: pointer;
  margin-top: 10px;
}

.btn:hover {
  background-color: #d27c57;
  border-color: #d27c57;
}
</style>