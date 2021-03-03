<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Blank</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Blank</ion-title>
        </ion-toolbar>
      </ion-header>

      <div id="container">
        <video muted autoplay></video>
        <div id="webcamcontrols">
          <button id="mic" @click="startRecording">MIC</button>
          <button id="record" @click="startRecording">RECORD</button>
          <div id="recordings"></div>
          <!-- <button class="recordbutton" @click="stopRecording">STOP</button> -->
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
interface State {
  webcamstream: any;
  streamRecorder: any;
  list: any;
}
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
} from "@ionic/vue";
import { defineComponent } from "vue";
export default defineComponent({
  name: "Home",
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
  },
  data() {
    return {
      webcamstream: null,
      streamRecorder: {},
      list: [],
    } as State;
  },
  mounted() {
    const constraints = {
      video: { width: { exact: 640 }, height: { exact: 480 } },
      audio: true,
    };
    const video: any = document.querySelector("video");
    this.list = document.getElementById("recordings");
    navigator.mediaDevices.getUserMedia(constraints).then((stream) => {
      video.srcObject = stream;
      this.webcamstream = stream;
      console.log(this.webcamstream);
    });
  },
  methods: {
    startRecording() {
      const mimeType = "audio/webm";
      const w: any = window;
      const getMic: any = document.getElementById("mic");
      const recordButton: any = document.getElementById("record");
      const list: any = document.getElementById("recordings");
      console.log(this.webcamstream);
      const mediaRecord: any = new w.MediaRecorder(this.webcamstream, {
        type: mimeType,
      });
      let chunks: any = [];
      console.log(this.webcamstream);
      mediaRecord.addEventListener("dataavailable", (event: any) => {
        if (typeof event.data === "undefined") return;
        if (event.data.size === 0) return;
        chunks.push(event.data);
      });
      mediaRecord.addEventListener("stop", () => {
        const recording = new Blob(chunks, {
          type: mimeType,
        });
        console.log(this.webcamstream);
        this.renderRecording(recording, this.list);
        chunks = [];
      });
      recordButton.removeAttribute("hidden");
      recordButton.addEventListener("click", () => {
        if (mediaRecord.state === "inactive") {
          mediaRecord.start();
          recordButton.innerText = "Stop";
        } else {
          mediaRecord.stop();
          recordButton.innerText = "Record";
        }
      });
    },
    renderRecording(blob: any, list: any) {
      const blobUrl = URL.createObjectURL(blob);
      const li = document.createElement("li");
      const audio = document.createElement("audio");
      const anchor = document.createElement("a");
      anchor.setAttribute("href", blobUrl);
      const now = new Date();
      anchor.setAttribute("download", `recording.webm`);
      anchor.innerText = "Download";
      audio.setAttribute("src", blobUrl);
      audio.setAttribute("controls", "controls");
      li.appendChild(audio);
      li.appendChild(anchor);
      this.list.appendChild(li);
    },
  },
});
</script>

<style scoped>
#container {
  text-align: center;

  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;

  color: #8c8c8c;

  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
