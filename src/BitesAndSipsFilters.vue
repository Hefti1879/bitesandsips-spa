<template>
  <div class="filters-container">
    <div class="filters-header">
      <h2>Filters:</h2>
    </div>
    <div class="filters-body">
      <div class="filter-group">
        <label>
          <input type="radio" v-model="placeType" value="restaurant" />
          Restaurant
        </label>
        <label>
          <input type="radio" v-model="placeType" value="bar" />
          Bar
        </label>
      </div>
      <div class="filter-group" v-if="coordinates.longitude.length < 1">
        <div class="address-group">
          <div class="address-row">
            <label> Country </label>
            <input type="text" v-model="address.country" />
            <label> Street </label>
            <input type="text" v-model="address.street" />
          </div>
          <div class="address-row">
            <label> Zip </label>
            <input type="text" v-model="address.zipcode" />
            <label> City </label>
            <input type="text" v-model="address.city" />
          </div>
        </div>
      </div>
      <div class="slider-group">
        <label>
          At least open for: {{ minTime }} hours
          <input type="range" v-model="minTime" min="1" max="5" />
        </label>
        <label>
          Maximum Distance: {{ maxDistance }} km
          <input type="range" v-model="maxDistance" min="1" max="5" />
        </label>
      </div>
    </div>
    <button
      class="search-button"
      @click="search()">
      Search
    </button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "BitesAndSipsFilters",
  props: {
    coordinates: {
      latitude: String,
      longitude: String,
    },
    token: String,
    filters: {
      placeType: String,
      minTime: Number,
      maxDistance: Number,
    },
  },
  data() {
    return {
      placeType: "restaurant",
      minTime: 1,
      maxDistance: 3,
      address: {
        street: "",
        city: "",
        country: "",
        zipcode: "",
      },
      isLoading: false,
    };
  },
  watch: {
    filters: {
      immediate: true,
      handler(newVal) {
        if (newVal == undefined || newVal.length < 1) return;
        this.placeType = newVal.placeType;
        this.maxDistance = newVal.maxDistance;
        this.minTime = newVal.minTime;
      },
    },
  },
  methods: {
    search() {
      this.save_filters();
      this.$emit(
        "search_places",
        this.maxDistance * 1000,
        this.minTime,
        this.placeType,
        this.address
      );
    },
    save_filters() {
      let data = {
        type: this.placeType,
        radius: this.maxDistance,
        minTime: this.minTime,
      };
      if (this.token != undefined && this.token.length > 0) {
        axios
          .post("https://localhost/api/setfilters", data)
          .then(() => {
            // Hier kannst du die Daten aus der Antwort in deine Komponente einfügen
          })
          .catch((error) => {
            this.$emit('setErrorText','Error: Filters not set');
            if (error.response.status == 401) {
              this.$emit("set-token", "");
            }
          });
      }
    },
  },
};
</script>

<style scoped>
.filters-container {
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  margin: 0 auto;
  max-width: 600px;
}

.filters-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}

.search-button {
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 8px 16px;
  border-radius: 5px;
  cursor: pointer;
}

.filter-group {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  margin-bottom: 10px;
}

.slider-group {
  display: flex;
  justify-content: space-evenly;
}

label {
  margin-right: 10px;
}

input[type="text"] {
  padding: 5px;
  border-radius: 5px;
  border: 1px solid #ccc;
  margin-right: 10px;
  flex: 1;
}

.address-group {
  display: flex;
  flex-direction: column;
  margin-top: 10px;
  align-items: flex-start;
}

.address-row {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  width: 100%;
}

.address-group label {
  width: 60px;
  text-align: left;
}
</style>
