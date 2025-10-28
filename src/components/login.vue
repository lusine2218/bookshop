<script setup>
import { ref, reactive } from 'vue'
import bcrypt from 'bcryptjs'

const error1 = ref("")
const error2 = ref("")
const error3 = ref("")
const error4 = ref("")
const error5 = ref("")
const emit = defineEmits(['actionOne'])

const data = reactive({
  name: "",
  gender: "",
  age: null,
  password: "",
})

function record() {
  let confirm = true
  error1.value = ""
  error2.value = ""
  error3.value = ""
  error4.value = ""
  error5.value = ""
  if (data.name == "") {
    error1.value = "Вы не ввели имя или сделали это не в правильном формате"
    confirm = false
  }
  if (data.gender == "") {
    error2.value = "Вы не определились с полом"
    confirm = false
  }
  if (data.age === null || data.age < 10 || data.age > 164) {
    error3.value = "Вы не ввели возраст или сделали это не в правильном формате"
    confirm = false
  }
  if (data.password == "") {
    error5.value = "Придумайте пароль"
    confirm = false
  }
  if (confirm) {
    const salt = bcrypt.genSaltSync(10)
    const hashedPassword = bcrypt.hashSync(data.password, salt)
    const users = JSON.parse(localStorage.getItem('users') || '[]')
    if (users.some(u => u.name === data.name)) {
      error1.value = "Пользователь с таким именем уже существует"
      return
    }
    users.push({
      name: data.name,
      gender: data.gender,
      age: data.age,
      role: data.role,
      password: hashedPassword 
    })
    localStorage.setItem('users', JSON.stringify(users))
    emit('actionOne', {
      name: data.name,
      gender: data.gender,
      age: data.age,
      role: data.role,
      password: hashedPassword
    })
    data.name = ""
    data.gender = ""
    data.age = null
    data.role = ""
    data.password = ""
  }
}
</script>

<template>
  <div class="cont">
    <h2>Регистрация</h2>

    <div class="form-group">
      <div class="label">Имя</div>
      <input type="text" v-model="data.name" class="width" />
      <div class="error">{{ error1 }}</div>
    </div>

    <div class="form-group">
      <div class="label">Пол</div>
      <div class="radio-group">
        <input type="radio" value="мужской" v-model="data.gender" id="one" class="hidden-radio">
        <label for="one" class="custom-radio">мужской</label>
        <input type="radio" value="женский" v-model="data.gender" id="two" class="hidden-radio">
        <label for="two" class="custom-radio">женский</label>
      </div>
      <div class="error">{{ error2 }}</div>
    </div>

    <div class="form-group">
      <div class="label">Возраст</div>
      <input type="number" v-model="data.age" class="width" min="10" max="164" />
      <div class="error">{{ error3 }}</div>
    </div>

    <div class="form-group">
      <div class="label">Пароль</div>
      <input type="password" v-model="data.password" class="width" />
      <div class="error">{{ error5 }}</div>
    </div>

    <button @click="record" class="btn">Подтвердить</button>
  </div>
</template>

<style scoped>
.cont {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 250px;
  margin: 0 auto;
  margin-top: 50px;
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
  background-color: rgb(189, 128, 102);
  color: #832510;
}

input[type=number] {
  background-color: #f1bf7e;
  border-color: #831010;
  color: #954b1e;
}

  

input[type=number]:focus {
  background-color: rgb(189, 131, 102);
  color: #832510;
}

input[type=password] {
  background-color: #f1bf7e;
  border-color: #831010;
  color: #954b1e;
}

input[type=password]:focus {
  background-color: rgb(189, 141, 102);
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
  background-color: #d89566;
}

.btn {
  background-color: #e3bebe83;
  color: #601818;
  padding: 10px 20px;
  cursor: pointer;
  margin-top: 10px;
}

.btn:hover {
  background-color: #d9815f;
  border-color: #d9815f;
}
</style>