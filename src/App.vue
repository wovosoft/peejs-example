<template>
  <Container>
    <Row>
      <Col :md="6">
        <Input :value="myId"/>
      </Col>
      <Col :md="6" :sm="12">
        <InputGroup>
          <Input v-model="theId"/>
          <Button variant="primary" @click="callUser">Connect</Button>
        </InputGroup>
      </Col>
      <Col :md="6" :sm="12">
        <video ref="myVideo" muted autoplay class="w-100 border"></video>
      </Col>
      <Col :md="6" :sm="12">
        <video ref="otherVideo" class="w-100 border"></video>
      </Col>
    </Row>
  </Container>
</template>
<script setup lang="ts">
import {Peer} from "peerjs";
import {Container, Row, Col, Input, InputGroup, Button} from "@wovosoft/wovoui";
import {ref} from "vue";

const theId = ref<string>();
const myId = ref<string>();
const myVideo = ref<HTMLVideoElement>();
const otherVideo = ref<HTMLVideoElement>();

const peer = new Peer();
peer.on("open", (id: string) => {
  myId.value = id;
});

const currentCall = ref<any>();

async function callUser() {
  const stream = await navigator.mediaDevices?.getUserMedia({
    video: true,
    audio: true,
  });

  myVideo.value.srcObject = stream;
  myVideo.value?.play();

  const call = peer.call(theId.value, stream);
  call.on("stream", (stream) => {
    otherVideo.value.srcObject = stream;
    otherVideo.value.play();
  });
  call.on("data", (stream) => {
    otherVideo.value.srcObject = stream;
  });
  call.on("error", (err) => {
    console.log(err);
  });
  call.on('close', () => {
    endCall()
  });

  currentCall.value = call;
}

peer.on("call", (call) => {
  // grab the camera and mic
  navigator.mediaDevices
      ?.getUserMedia({video: true, audio: true})
      .then((stream) => {
        // play the local preview
        myVideo.value.srcObject = stream;
        myVideo.value.play();
        call.answer(stream);
        currentCall.value = call;
        call.on("stream", (remoteStream) => {
          // when we receive the remote stream, play it
          otherVideo.value.srcObject = remoteStream;
          otherVideo.value.play();
        });
      })
      .catch((err) => {
        console.log("Failed to get local stream:", err);
      });
});

function endCall() {
  if (!currentCall) return;

  try {
    currentCall.value?.close();
  } catch {
  }
  currentCall.value = undefined;
}
</script>

