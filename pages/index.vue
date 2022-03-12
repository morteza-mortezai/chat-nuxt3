<template>
  <div class="grid grid-cols-2 gap-4">
    <div>
      <!-- connection status -->
      <div class="border bg-blue-100 text-center p-5 rounded">
        connection status : {{ connected ? "on" : "off" }}
      </div>
      <!-- connection error -->
      <div
        class="border bg-red-100 text-center p-5 rounded"
        v-if="connectionError"
      >
        connection error
      </div>
      <!-- massages -->
      <div class="bg-green-100 p-5">
        <div
          v-for="(item, i) in messages"
          :key="i"
          class="bg-green-200 rounded-full my-2 p-2"
        >
          {{ item }}
        </div>
      </div>
      <!-- actions -->
      <div class="flex">
        <div class="flex flex-col flex-grow">
          <input
            v-model="text"
            name="msg"
            type="text"
            class="border-2 p-2"
            @keydown.enter="onSend"
          />
          <span>{{ msgError }}</span>
        </div>
        <button
          @click="onSend"
          type="submit"
          class="btn btn--primary"
          :disabled="connectionError"
        >
          send
        </button>
      </div>
    </div>
  </div>
</template>
<script lang="ts" setup>
import { useField, useForm } from "vee-validate";
import * as yup from "yup";
import { ref, onMounted, reactive } from "vue";

// states
var connection = null;
const messages = reactive([]);
let connected = ref(false);
let connectionError = ref(false);

// validation
const schema = yup.object({
  msg: yup.string().required().min(1).nullable(),
});
const { handleSubmit } = useForm({
  validationSchema: schema,
});
const { value: text, errorMessage: msgError } = useField("msg");

// methods
function connect() {
  connection = new WebSocket(
    "wss://demo.piesocket.com/v3/channel_1?api_key=oCdCMcMPQpbvNjUIzqtvF1d2X2okWpDQj4AwARJuAgtjhzKxVEjQU6IdCjwm&notify_self"
  );
  connection.onopen = function () {
    console.log("Server connected");
    connected.value = true;
  };
  connection.onmessage = function (e) {
    messages.push(e.data);
  };
  connection.onerror = function (error) {
    console.log("WebSocket Error: " + error);
    connectionError.value = true;
  };
  connection.onclose = function (e) {
    console.log("Server closed");
    connected.value = false;
  };
}
function close() {
  connection.close();
}
const onSend = handleSubmit(({ msg }) => {
  if (msg.includes("profanity")) {
    alert("you cant send this msg");
    return;
  } else if (msg === "restart") {
    restart();
    return;
  }
  sendMessage(msg);
});
function sendMessage(text: any) {
  connection.send(text);
}
function restart() {
  connection.close();

  setTimeout(function () {
    connect();
  }, 1000);
}

// life cylcels
onMounted(() => {
  connect();
});
</script>
<style lang="postcss ">
.btn {
  @apply text-base font-medium rounded-lg p-3;
}

.btn--primary {
  @apply bg-sky-500 text-white;
}
</style>
