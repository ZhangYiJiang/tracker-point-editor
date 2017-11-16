<template>
<div>
  <div :class="{ crosshair: true, keyframe: isKeyframe }">
    <p>{{ point.id }}</p>
  </div>
</div>
</template>

<script>
export default {
  props: ['point', 'isKeyframe'],


}
</script>

<style scoped lang="scss">
.crosshair {
  width: 1px;
  height: 1px;
  background: rgb(225, 40, 30);
  position: absolute;

  opacity: 0.6;
  color: #fff;

  &::before {
    content: '';
    position: absolute;
    left: -7px;
    top: -7px;
    width: 11px;
    height: 11px;
    border: 2px solid rgb(225, 40, 30);
  }

  &.keyframe {
    color: orange;
    opacity: 1;
  }

  p {
    margin: 0.5rem 0 0 0;
    padding: 0;
    font-weight: bold;
    text-shadow: 0 0 1px #000, 0 0 1px #000;
  }
}
</style>
