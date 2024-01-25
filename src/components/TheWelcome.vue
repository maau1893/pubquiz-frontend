<script setup lang="ts">
import { ref } from "vue";

const restResponse = ref("");
async function restRequest() {
  restResponse.value = "";
  try {
    const response = await fetch(`${import.meta.env.VITE_BACKEND_URL}/health`);
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
        `${import.meta.env.VITE_BACKEND_URL}/echo`,
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
      console.log("Could not espablish Websocket connection");
    }
  } else {
    wsConnection.value.close();
  }
}
</script>

<template>
  <button @click="restRequest">REST</button>
  <p>{{ restResponse }}</p>
  <button @click="toggleConnection">Websocket</button>
  <p>Connection {{ wsStatus }}</p>
  <div>
    <p v-for="msg of websocketMessages" :key="msg">{{ msg }}</p>
  </div>
</template>
