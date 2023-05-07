<template>
  <v-row>
    <v-col cols="8" md="6">
      <v-btn @click="startCamera">Start Camera</v-btn>
      <v-btn @click="stopCamera">Stop Camera</v-btn>
      <div style="position: relative; width: 500px; height: 500px;">
        <video ref="video" style="position: absolute; width: 100%; height: 440px;" autoplay></video>
        <canvas ref="canvas" style="position: absolute; top:30px; width: 100%; height: 380px;"></canvas>
      </div>
    </v-col>
    <v-col cols="4" md="6">
      <div>
        <h2>Tracked Items:</h2>
        <ul>

          <li v-for="(item, index) in trackedItems" :key="index">
            {{ item.name }} => {{ item.duration }}
          </li>


        </ul>

      </div>
    </v-col>
  </v-row>
</template>
<script>
import * as tf from '@tensorflow/tfjs';
import * as cocoSsd from '@tensorflow-models/coco-ssd';
export default {
  data() {
    return {
      stream: null,
      trackedItems: [],

      lastTrackedItems: [],
      longestDurationItem: { name: '', duration: '0s' },

    };
  },

  computed: {

  },
  watch: {

  },


  async mounted() {
    const millisToTime = (millis) => {
      const padZero = (num) => (num < 10 ? '0' + num : num);
      const seconds = padZero(Math.floor((millis / 1000) % 60));
      const minutes = padZero(Math.floor((millis / (1000 * 60)) % 60));
      const hours = padZero(Math.floor((millis / (1000 * 60 * 60)) % 24));
      return `${hours}:${minutes}:${seconds}`;
    };

    const model = await cocoSsd.load();
    const canvas = this.$refs.canvas;
    const context = canvas.getContext('2d');
    const trackedItems = {};
    let longestDurationItem = { name: '', duration: '00:00:00' };
    let time = 0;

    setInterval(async () => {
      if (!this.stream) return;
      const video = this.$refs.video;
      const start = Date.now();
      const predictions = await model.detect(video);
      const elapsed = Date.now() - start;
      const remaining = Math.max(0, 1000 - elapsed);
      await new Promise(resolve => setTimeout(resolve, remaining));

      context.clearRect(0, 0, canvas.width, canvas.height);
      context.font = '16px Arial';
      context.fillStyle = '#ffffff';
      context.strokeStyle = '#00ffff';
      context.lineWidth = 3;

      predictions.forEach(prediction => {
        context.beginPath();
        context.rect(...prediction.bbox);
        context.stroke();
        context.fillText(prediction.class, prediction.bbox[0], prediction.bbox[1] - 5);
        context.strokeText(prediction.class, prediction.bbox[0], prediction.bbox[1] - 5);

        const now = Date.now();
        if (!trackedItems[prediction.class]) {
          trackedItems[prediction.class] = { startTime: now, endTime: null, duration: 0 };
        } else if (trackedItems[prediction.class].endTime === null) {
          trackedItems[prediction.class].duration += now - trackedItems[prediction.class].startTime;
          trackedItems[prediction.class].startTime = now;
        }
      });

      this.trackedItems = Object.entries(trackedItems).map(([name, item]) => ({
        name,
        duration: millisToTime(item.duration),
      }));

      longestDurationItem = this.trackedItems.reduce((acc, cur) =>
        cur.duration > acc.duration ? cur : acc, longestDurationItem);

      this.longestDurationItem = `${longestDurationItem.name}: ${longestDurationItem.duration}`;

      sessionStorage.setItem('trackedItems', JSON.stringify(this.trackedItems));
      sessionStorage.setItem('longestDurationItem', this.longestDurationItem);

      Object.entries(trackedItems).forEach(([name, item]) => {
        const prediction = predictions.find(p => p.class === name);
        if (prediction === undefined) {
          item.endTime = Date.now();
        } else {
          item.startTime = Date.now();
          item.endTime = null;
        }
      });

      time++;
      this.time = millisToTime(time * 1000);
    }, 1000);
  },




  methods: {


    formatDuration(duration) {
          const minutes = Math.floor(duration / 60000);
          const seconds = Math.floor((duration % 60000) / 1000);
          return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
        },
        async startCamera() {
          try {
            const video = this.$refs.video;
            this.stream = await navigator.mediaDevices.getUserMedia({ video: true });
            video.srcObject = this.stream;
            await video.play();
            const canvas = this.$refs.canvas;
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
          } catch (error) {
            console.error('Failed to start camera:', error);
          }
        },
        stopCamera() {
          if (this.stream) {
            this.stream.getTracks().forEach(track => track.stop());
            this.stream = null;
          }
        },
      },
  created() {


  },
};
</script>

