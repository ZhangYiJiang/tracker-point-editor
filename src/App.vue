<template>
  <div id="app">
    <h1>Track Point Editor</h1>
    <p>Add pins to transform a image by mapping points on the source image
      to a set of tracking frames</p>
    <ol>
      <li>Import the source image</li>
      <li>Add pins on important features of the source image</li>
      <li>Import the frames containing the desired transformation</li>
      <li>Offset the tracking frames so they roughly match the position of the pins</li>
      <li>Scrub the frames, adjusting the pins as you go along. Keyframes (orange pins) are
        automatically added as you adjust the pins</li>
      <li>Progress is automatically saved locally, and you can export the data when you're happy.</li>
    </ol>

    <div v-if="!source || !frames.length">
      <h3>Upload source/mask images</h3>
      <label for="source">Source frame</label>
      <input type="file" id="source" @change="sourceChanged" />

      <label for="masks">Tracking frames</label>
      <input type="file" id="masks" multiple @change="maskChanged" />
    </div>

    <div v-if="source" @click="addPoint" class="frame-container">
      <h2>Source</h2>

      <p>Click on image to add tracking points</p>
      <image-frame :frame="source"
                   :points="points"
                   @pointMove="pointMoved" />
    </div>

    <div v-show="frames.length">
      <h2>Tracking Frames</h2>

      <div>
        <h3>Alignment</h3>
        <p>Use this to realign the mask frames under the initial set of points</p>
        <label for="frame-offset-x">Horizontal</label>
        <input type="number"
               id="frame-offset-x"
               ref="offsetXInput"
               @change="updateOffset('x', $event.target.value)" />

        <label for="frame-offset-y">Vertical</label>
        <input type="number"
               id="frame-offset-y"
               ref="offsetYInput"
               @change="updateOffset('y', $event.target.value)" />
      </div>

      <br>

      <label for="scrub">Scrub: </label>
      <input id="scrub" type="range" min="0" :max="frames.length - 1"
             @input="updateFrame($event.target.value)" />

      <label>Onion</label>

      <div v-if="currentFrame" class="frame-container">
        <h3>Frame {{ current }}</h3>
        <image-frame :frame="currentFrame"
                     :points="points"
                     :offset="offset"
                     @pointMove="pointMoved" />
      </div>
    </div>

    <div v-show="Object.keys(points).length">
      <h3>Data</h3>
      <textarea rows="20" cols="50" ref="dataTextarea"></textarea>
      <button @click="importData">Import</button>
      <button @click="exportData">Export</button>
      <button @click="resetData">Clear saved data</button>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import ImageFrame from './ImageFrame';

const pointNames = 'abcdefghijklmnopqrstuvwxyz1234567890';
const storageKey = 'app-data';

export default {
  name: 'app',

  components: {
    ImageFrame,
  },

  data() {
    let points, offset;
    try {
      const data = JSON.parse(localStorage.getItem(storageKey));
      points = data.points;
      offset = data.offset;
    } catch (e) {
      console.error('Cannot read localStorage data');
    }

    return {
      source: null,
      frames: [],
      current: 0,
      points: points || {},
      offset: offset || { x: 0, y: 0 },
    };
  },

  watch: {
    points() { this.persist() },
    'offset.x'() { this.persist() },
    'offset.y'() { this.persist() },
  },

  computed: {
    currentFrame() {
      if (!this.frames.length) return null;
      return this.frames[this.current];
    },

    data() {
      return JSON.stringify({
        points: this.points,
        offset: this.offset,
      }, null, 2);
    }
  },

  methods: {
    // Data structure factories
    addPoint(evt) {
      // Find offset for new point
      const targetRect = evt.target.getBoundingClientRect();
      const x = evt.clientX - targetRect.left;
      const y = evt.clientY - targetRect.top;

      const id = pointNames[Object.keys(this.points).length];
      this.$set(this.points, id, {
        id,
        keyframes: {
          0: { x, y }
        },
      });
    },

    makeFrame(file, id) {
      return {
        id,
        file,
      };
    },

    // Data input events
    sourceChanged(evt) {
      const file = evt.target.files.item(0);
      this.source = this.makeFrame(file, 0);
    },

    maskChanged(evt) {
      this.frames = Array.from(evt.target.files)
        .map((file, index) => this.makeFrame(file, index));
    },

    pointMoved(evt) {
      const { point, frame, x, y } = evt;
      this.$set(this.points[point.id].keyframes, frame.id, {
        x: Math.round(x),
        y: Math.round(y),
      });
      this.$nextTick(() => this.persist());
    },

    // Tracking frame controls
    updateFrame(frame) {
      this.current = parseInt(frame, 10);
    },

    updateOffset(axis, value) {
      this.offset[axis] = parseInt(value, 10);
    },

    // Data storage and persistence
    persist() {
      localStorage.setItem(storageKey, this.data);
    },

    exportData() {
      this.$refs.dataTextarea.value = this.data;
    },

    importData() {
      try {
        const { points, offset } = JSON.parse(this.$refs.dataTextarea.value);
        const { x, y } = offset;
        if (typeof x !== 'number' || typeof y !== 'number' || !points) {
          throw new Error('Invalid data');
        }

        this.offset = offset;
        this.points = points;
      } catch (e) {
        alert('Invalid data / JSON');
      }
    },

    resetData() {
      if (confirm('This will clear ALL locally saved data. Are you sure you want to continue?')) {
        localStorage.setItem(storageKey, '');
        window.reload();
      }
    }
  },

  mounted() {
    // Hack to make initial form values work
    this.$refs.offsetXInput.value = this.offset.x;
    this.$refs.offsetYInput.value = this.offset.y;
  }
}
</script>

<style lang="scss">
#app {
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul, ol {
  max-width: 600px;
  margin: 0 auto;
  text-align: left;
  padding: 0;
}

.frame-container {
  user-select: none;
  overflow: hidden;
}

li {
  margin: 0 10px;
}

a {
  color: #42b983;
}

textarea {
  display: block;
  width: 600px;
  margin: 20px auto;
}

input[type='range'] {
  width: 500px;
}
</style>
