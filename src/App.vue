<template>
  <div id="app">
    <div v-if="!source || !frames.length">
      <label for="source">Source frame</label>
      <input type="file" id="source" @change="sourceChanged" />

      <label for="masks">Mask frames</label>
      <input type="file" id="masks" multiple @change="maskChanged" />
    </div>

    <div @click="addPoint">
      <h2>Source</h2>

      <p>Click on image to add tracking points</p>
      <image-frame v-if="source"
                   :frame="source"
                   :points="points"
                   @pointMove="pointMoved" />
    </div>

    <h2>Frames</h2>

    <p>Use this to realign the frames under the initial set of points</p>
    <label for="frame-offset-x">Horizontal offset</label>
    <input type="number"
           id="frame-offset-x"
           ref="offsetXInput"
           @change="updateOffset('x', $event.target.value)" />

    <label for="frame-offset-y">Vertical offset</label>
    <input type="number"
           id="frame-offset-y"
           ref="offsetYInput"
           @change="updateOffset('y', $event.target.value)" />

    <br>

    <label>Scrub: </label>
    <input type="range" min="0" :max="frames.length - 1"
           @input="updateFrame($event.target.value)" />

    <label>Onion</label>

    <div v-if="currentFrame">
      <h3>Frame {{ current }}</h3>
      <image-frame :frame="currentFrame"
                   :points="points"
                   :offset="offset"
                   @pointMove="pointMoved" />
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
  },

  methods: {
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

    sourceChanged(evt) {
      const file = evt.target.files.item(0);
      this.source = this.makeFrame(file, 0);
    },

    maskChanged(evt) {
      this.frames = Array.from(evt.target.files).map((file, index) => this.makeFrame(file, index));
    },

    pointMoved(evt) {
      const { point, frame, x, y } = evt;
      if (x < 500) {
        console.log('Out of bounds - x - ignoring', x);
        return;
      }

      this.$set(this.points[point.id].keyframes, frame.id, {x, y});
      this.$nextTick(() => this.persist());
    },

    updateFrame(frame) {
      this.current = parseInt(frame, 10);
    },

    updateOffset(axis, value) {
      this.offset[axis] = parseInt(value, 10);
    },

    persist() {
      localStorage.setItem(storageKey, JSON.stringify({
        points: this.points,
        offset: this.offset,
      }));
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
  user-select: none;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
