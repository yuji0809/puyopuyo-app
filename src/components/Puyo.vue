<template>
  <div :class="['puyo', type]"></div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  props: {
    type: String as () => string | null,
  },
});
</script>

<style scoped>
.puyo {
  width: 40px;
  height: 40px;
  border: 1px solid #ccc;
}

.puyo.red {
  background-color: red;
}

.puyo.green {
  background-color: green;
}

.puyo.blue {
  background-color: blue;
}

.puyo.yellow {
  background-color: yellow;
}

.puyo.purple {
  background-color: purple;
}
</style>
  