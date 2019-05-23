<template>
  <panel-item :field="field">
    <div slot="value" v-if="field.value.address">
      <div class="search-box">{{field.value.address}}</div>
      <div class="amap-page-container">
        <el-amap
          v-if="center.length"
          vid="amapDemo"
          :center="center"
          :zoom="zoom"
          class="amap-field"
        >
          <el-amap-marker v-if="position.length" vid="component-marker" :position="position"></el-amap-marker>
        </el-amap>
      </div>
    </div>
    <span v-else slot="value">&mdash;</span>
  </panel-item>
</template>

<script>
export default {
  props: ["resource", "resourceName", "resourceId", "field"],
  data() {
    return {
      zoom: 12,
      center: [],
      position: []
    };
  },
  mounted() {
    let lng,
      lat,
      zoom = 12;

    if (this.field.value instanceof Object) {
      lng = this.field.value.lng;
      lat = this.field.value.lat;
      this.center = [lng, lat];
      this.position = [lng, lat];
    }
  }
};
</script>
