<template>
  <div class="image-frame"
       @dragover.prevent
       @drop.prevent="pointDragEnd">
    <div class="point-cloud" @dragover.prevent>
      <div v-for="point in points"
           :key="point.id"
           draggable
           class="point"
           @dragstart="pointDragStart($event, point)"
           :style="position(point)"
      >
        <image-point :point="point"
                     :isKeyframe="lastKeyFrame(point) === String(frame.id)" />
      </div>
    </div>

    <img v-if="onionSrc" class="image onion" :src="onionSrc" :style="imgOffset" />
    <img class="image" :src="imgSrc" :style="imgOffset" />
  </div>
</template>

<script>
  import ImagePoint from './ImagePoint';

  export default {
    props: ['frame', 'points', 'offset', 'onion'],

    components: {
      ImagePoint,
    },

    computed: {
      imgSrc() {
        return window.URL.createObjectURL(this.frame.file);
      },

      onionSrc() {
        if (!this.onion) return null;
        return window.URL.createObjectURL(this.onion.file);
      },

      onionPoints() {

      },

      imgOffset() {
        if (!this.offset) return {};
        return { transform: `translate(${this.offset.x}px, ${this.offset.y}px)` };
      },
    },

    methods: {
      lastKeyFrame(point) {
        const frames = Object.keys(point.keyframes).sort((a, b) => b - a);
        // Find last keyframe
        return frames.find(frame => frame <= this.frame.id);
      },

      position(point) {
        const lastFrame = this.lastKeyFrame(point);
        const { x, y } =  point.keyframes[lastFrame];
        return { left: x + 'px', top: y + 'px' };
      },

      pointDragStart(evt, point) {
        evt.dataTransfer.setData('text/plain', JSON.stringify({
          start: {
            x: evt.pageX,
            y: evt.pageY,
          },
          point,
        }));
      },

      pointDragEnd(evt) {
        try {
          const { start, point } = JSON.parse(evt.dataTransfer.getData('text/plain'));
          const current = point.keyframes[this.lastKeyFrame(point)];
          const x = current.x + (evt.pageX - start.x);
          const y = current.y + (evt.pageY - start.y);

          console.log('New x y', x, y)
          console.log('Event', evt)

          this.$emit('pointMove', {
            point,
            x,
            y,
            frame: this.frame,
          });
        } catch (e) {
          console.error(e);
        }
      },
    }
  }
</script>

<style scoped lang="scss">
  .image-frame {
    position: relative;
    width: 1920px;
    left: calc(50vw - 960px);
    user-select: none;

    .image {
      -moz-user-select: none;
      -webkit-user-select: none;
      user-select: none;
      pointer-events: none;
    }

    .onion {
      position: absolute;
      opacity: 0.4;
      z-index: 50;
    }

    .point {
      height: 1px;
      width: 1px;

      cursor: move;
      z-index: 100;
      position: absolute;
    }
  }
</style>
