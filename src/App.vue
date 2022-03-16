<script>
import { reactive, onMounted, ref } from 'vue';
import ws from './ws';

export default {
  setup() {
    const inputUsername = ref("");
    const inputMessage = ref("");

    const state = reactive({
      username: "",
      message: [],
    });
    const Login = () => {
      if (inputUsername.value != "" || inputUsername.value != null) {
        state.username = inputUsername.value;
        ws.send(state.username);
        inputUsername.value = "";
      }
    }
    const MessageSend = () => {
      if (inputMessage.value == "" || inputMessage.value == null) {
        return;
      }
      ws.send(JSON.stringify(inputMessage.value));
      inputMessage.value = "";
    }

    onMounted(() => {
      ws.onopen = () => {
        console.log("connected");
      }
      ws.onmessage = (msg) => {
        let obj = JSON.parse(msg.data);
        state.message.push({
          rmsg: obj.msg,
          name: obj.UserName.name,
        })
      }
    })
    return {
      inputUsername,
      state,
      Login,
      inputMessage,
      MessageSend,
    }
  }
}
  
</script>

<template>
  <div v-if="state.username === '' || state.username === null">
    <form @submit.prevent="Login">
      <input type="text" v-model="inputUsername" placeholder="username">
      <input type="submit" value="Send">
    </form>
  </div>
  <div v-else>
    <section>
      <div v-for="msg in state.message" :key="msg.key">
        <div><b>{{ msg.name }}:</b>{{ msg.rmsg }}</div>
      </div>
    </section>
    <footer>
      <form @submit.prevent="MessageSend">
        <input type="text" v-model="inputMessage" placeholder="message">
        <input type="submit" value="Send">
      </form>
    </footer>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
