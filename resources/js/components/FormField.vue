<template>
  <default-field :field="field" :errors="errors">
    <template slot="field">
      <el-amap-search-box
        class="search-box"
        :search-option="searchOption"
        :on-search-result="onSearchResult"
      ></el-amap-search-box>
      <div class="amap-page-container">
        <el-amap
          vid="amapDemo"
          :center="center"
          :zoom="zoom"
          :plugin="!field.value?plugin:null"
          class="amap-field"
          :events="events"
        >
          <el-amap-marker vid="component-marker" :position="position" v-if="position.length"></el-amap-marker>
        </el-amap>
      </div>
      <input
        v-if="value.address"
        :id="field.name"
        type="text"
        class="w-full form-control form-input form-input-bordered"
        :class="errorClasses"
        :placeholder="field.name"
        v-model="value.address"
      >
    </template>
  </default-field>
</template>

<script>
import { FormField, HandlesValidationErrors } from "laravel-nova";

export default {
  mixins: [FormField, HandlesValidationErrors],

  props: ["resourceName", "resourceId", "field"],
  data() {
    return {
      zoom: 12,
      center: [113.4911, 22.56153],
      position: [],
      address: "",
      searchOption: {
        city: "广东",
        citylimit: false
      },
      events: {
        click: e => {
          let { lng, lat } = e.lnglat;
          this.lng = lng;
          this.lat = lat;
          this.setMarker(lng, lat);
          this.geocoder(lng, lat);
          this.setAmapValue();
        }
      },
      plugin: [
        {
          pName: "Geolocation",
          events: {
            init: o => {
              o.getCurrentPosition((status, result) => {
                if (result && result.position) {
                  let { lng, lat } = result.position;
                  this.lng = lng;
                  this.lat = lat;
                  this.setMarker(lng, lat);
                  this.geocoder(lng, lat);
                  this.setAmapValue();
                  this.zoom = 20;
                  this.$nextTick();
                }
              });
            }
          }
        }
      ],
      lng: 0,
      lat: 0
    };
  },
  mounted() {
    let lng,
      lat,
      zoom = 12;

    if (this.field.value instanceof Object) {
      lng = this.field.value.lng;
      lat = this.field.value.lat;
      this.setMarker(lng, lat);
    } else {
      lng = this.field.lng;
      lat = this.field.lat;
      zoom = this.field.zoom;
    }
    this.lat = lat;
    this.lng = lng;
    this.zoom = zoom;
    this.center = [lng, lat];
  },
  methods: {
    /*
     * Set the initial, internal value for the field.
     */
    setInitialValue() {
      this.value = this.field.value || "";
    },

    /**
     * Fill the given FormData object with the field's internal value.
     */
    fill(formData) {
      formData.append(this.field.attribute, JSON.stringify(this.value || ""));
    },

    /**
     * Update the field's internal value.
     */
    handleChange(value) {
      this.value = value;
    },
    setAmapValue() {
      let value = {
        lng: this.lng,
        lat: this.lat,
        address: this.address
      };
      this.handleChange(value);
    },
    geocoder(lng, lat) {
      // 这里通过高德 SDK 完成。
      let geocoder = new AMap.Geocoder({
        radius: 1000,
        extensions: "all"
      });
      geocoder.getAddress([lng, lat], (status, result) => {
        if (status === "complete" && result.info === "OK") {
          if (result && result.regeocode) {
            this.address = result.regeocode.formattedAddress;
            this.setAmapValue();
            this.$nextTick();
          }
        }
      });
    },
    setMarker(lng, lat) {
      this.position = [lng, lat];
    },
    onSearchResult(pois) {
      if (pois.length > 0) {
        let { lng, lat } = pois[0];
        this.center = [lng, lat];
        this.setMarker(lng, lat);
        this.geocoder(lng, lat);
        this.lng = lng;
        this.lat = lat;
        this.setAmapValue();
      }
    }
  }
};
</script>
