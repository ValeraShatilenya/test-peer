<template>
  <div class="h-screen flex flex-col py-2 px-3">
    <div
      v-if="peerId"
      class="grid grid-cols-[auto_1fr] gap-2 items-center mb-2"
    >
      <BaseButton @click="copyPeerId">{{ peerId }} </BaseButton>
      <TextInput
        v-model="otherPeerId"
        placeholder="ID другого узла..."
        button-text="Соединиться"
        @button-click="connectToNode"
      />
    </div>
    <div class="bg-gray-200 space-y-2 flex-1 overflow-y-auto px-4 py-2">
      <div
        v-for="{ text, from } in messages"
        class="rounded-lg p-2 shadow-sm w-fit"
        :class="{
          'bg-white': from === 'other',
          'ml-auto bg-blue-500 text-white': from === 'me',
          'mx-auto': !from,
        }"
      >
        {{ text }}
      </div>
    </div>
    <div class="flex items-center gap-4 mt-2">
      <TextInput
        v-model="message"
        placeholder="Напишите сообщение..."
        button-text="Отправить"
        :disabled="!conn"
        @button-click="sendMess"
      />
    </div>
  </div>
</template>

<script setup>
import { Peer } from "peerjs";
import { ref, shallowRef } from "vue";
import TextInput from "./components/TextInput.vue";
import BaseButton from "./components/BaseButton.vue";

const messages = ref([]);
const message = ref("");
const peer = new Peer();
const otherPeerId = ref("");
const peerId = ref("");
const conn = shallowRef(null); //переменная, хранящая соединение

peer.on("open", function (p) {
  peerId.value = p;
});

peer.on("connection", function (c) {
  //входящее соединение...
  conn.value = c;
  initConn();
});

function addMess(mess) {
  messages.value.push(mess);
}

function connectToNode() {
  //исходящее соединение...
  conn.value = peer.connect(otherPeerId.value);
  initConn();
}

function initConn() {
  conn.value.on("open", function () {
    //открыто соединение
    addMess({ text: "Соединение установлено" });
    conn.value.on("data", function (text) {
      //прилетело сообщение
      addMess({ text, from: "other" });
    });
  });
  conn.value.on("close", function () {
    addMess({ text: "Соединение разорвано" });
  });
}

function sendMess() {
  addMess({ text: message.value, from: "me" });
  conn.value.send(message.value);
  message.value = "";
}

const copyPeerId = () => {
  navigator.clipboard.writeText(peerId.value);
};
</script>
