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

    <BaseButton @click="callToNode">Вызов</BaseButton>
    <video ref="myVideo" muted="muted" width="400px" height="auto" />
    <div v-if="peerCall">
      Входящий звонок
      <BaseButton @click="callAnswer">Принять</BaseButton>
      <!-- <button @click="callcancel">Отклонить</button> -->
    </div>
    <div v-if="peerCall && remVideo">
      Звонок начат...
      <!-- <button @click="callclose">Завершить звонок</button> -->
    </div>
    <video ref="remVideo" width="400px" height="auto" />
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
const peerCall = shallowRef(null);
const conn = shallowRef(null); //переменная, хранящая соединение

const myVideo = ref(null);
const remVideo = ref(null);

peer.on("open", (p) => (peerId.value = p));

peer.on("connection", function (c) {
  //входящее соединение...
  conn.value = c;
  initConn();
});

peer.on("call", (call) => (peerCall.value = call));

async function callAnswer() {
  const mediaStream = await navigator.mediaDevices.getUserMedia({
    audio: true,
    // video: true,
  });

  const video = myVideo.value;

  peerCall.value.answer(mediaStream); // отвечаем на звонок и передаем свой медиапоток собеседнику
  //peercall.on ('close', onCallClose); //можно обработать закрытие-обрыв звонка
  video.srcObject = mediaStream; //помещаем собственный медиапоток в объект видео (чтоб видеть себя)
  video.onloadedmetadata = function (e) {
    //запускаем воспроизведение, когда объект загружен
    video.play();
  };
  setTimeout(function () {
    //входящий стрим помещаем в объект видео для отображения
    remVideo.value.srcObject = peerCall.value.remoteStream;
    remVideo.value.onloadedmetadata = function (e) {
      // и запускаем воспроизведение когда объект загружен
      remVideo.value.play();
    };
  }, 1500);
}

async function callToNode() {
  try {
    //вызов
    const mediaStream = await navigator.mediaDevices.getUserMedia({
      audio: true,
      // video: true,
    });

    const video = myVideo.value;

    peerCall.value = peer.call(otherPeerId.value, mediaStream);

    peerCall.value.on("stream", function (stream) {
      //нам ответили, получим стрим
      setTimeout(function () {
        remVideo.value.srcObject = peerCall.value.remoteStream;
        remVideo.value.onloadedmetadata = function (e) {
          remVideo.value.play();
        };
      }, 1500);
    });
    //  peercall.on('close', onCallClose);
    video.srcObject = mediaStream;
    video.onloadedmetadata = function (e) {
      video.play();
    };
  } catch (error) {
    console.error(error);
  }
}

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
