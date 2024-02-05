<script setup lang="ts">
import { ref } from "vue";

const restResponse = ref("");
async function restRequest() {
  restResponse.value = "";
  try {
    // const response = await fetch(`${import.meta.env.VITE_BACKEND_URL}/health`);
    const response = await fetch(`https://quiz.impact6.de/health`);
    if (response.ok) {
      restResponse.value = await response.json();
    } else {
      restResponse.value = `Failed the request with status ${response.status}`;
    }
  } catch (e) {
    restResponse.value = "A network error occured";
  }
}

const websocketMessages = ref<string[]>([]);
const wsStatus = ref("CLOSED");
const wsConnection = ref<WebSocket>();
function toggleConnection() {
  if (wsConnection.value === undefined) {
    try {
      wsStatus.value = "CONNECTING";
      wsConnection.value = new WebSocket(
        `wss://quiz.impact6.de/echo`,
      );
      wsConnection.value.onopen = () => {
        wsStatus.value = "OPEN";
      };
      wsConnection.value.onmessage = (msg: MessageEvent<string>) => {
        websocketMessages.value.push(msg.data);
      };
      wsConnection.value.onclose = () => {
        wsStatus.value = "CLOSED";
        wsConnection.value = undefined;
      };
    } catch (e) {
      wsStatus.value = "ERROR";
      console.log("Could not espablish Websocket connection", e);
    }
  } else {
    wsConnection.value.close();
  }
}

const wsMessage = ref<string>("");
function sendMessage() {
  if (wsStatus.value === "OPEN") {
    const msg = wsMessage.value;
    wsConnection.value?.send(msg);
    wsMessage.value = "";
  }
}
</script>

<template>
  <button @click="restRequest">REST</button>
  <p>{{ restResponse }}</p>
  <button @click="toggleConnection">Websocket</button>
  <p>Connection {{ wsStatus }}</p>
  <form @submit.prevent="sendMessage">
    <input v-model="wsMessage" />
    <button type="submit">Send</button>
  </form>
  <div>
    <p v-for="msg of websocketMessages" :key="msg">{{ msg }}</p>
  </div>
</template>
