<template>
  <div v-if="field.asHtml" v-html="field.value"></div>
  <span v-else-if="field.value.address">{{ field.value.address }}</span>
  <span v-else>&mdash;</span>
</template>

<script>
export default {
  props: ["resourceName", "field"]
};
</script>
