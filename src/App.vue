<script setup>
import { Peer } from "peerjs";
import { ref } from "vue";

const messList = ref([]);
const message = ref("");
const peer = new Peer();
const otherPeerId = ref("");
const peerId = ref("");
let conn; //переменная, хранящая соединение

peer.on("open", function (p) {
  peerId.value = p;
});

peer.on("connection", function (c) {
  //входящее соединение...
  conn = c;
  initConn();
});

function addMess(mess) {
  messList.value.push(mess);
}

function connectToNode() {
  //исходящее соединение...
  conn = peer.connect(otherPeerId.value);
  initConn();
}

function initConn() {
  conn.on("open", function () {
    //открыто соединение
    addMess("<div><h4>Соединение установлено</h4></div>");
    conn.on("data", function (data) {
      //прилетело сообщение
      addMess("<div><b>Партнер: </b>" + data + "</div>");
    });
  });
  conn.on("close", function () {
    addMess("-----------Соединение разорвано-------------");
  });
}

function sendMess() {
  addMess("<div><b>Я: </b>" + message.value + "</div>");
  conn.send(message.value);
  message.value = "";
}
</script>

<template>
  <div>
    <h3>
      Мой ID: <span>{{ peerId }}</span>
    </h3>
    <input v-model="otherPeerId" type="text" />
    <button @click="connectToNode">Соединиться</button>
    <div>
      {{ messList }}
    </div>
    <br />
    <textarea v-model="message" />
    <br />
    <button @click="sendMess">Отправить</button>
  </div>
</template>
