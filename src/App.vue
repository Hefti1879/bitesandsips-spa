<template>
  <div id="App" style="background-color: #faecdd">
    <img alt="bitesandsips_logo" src="./assets/logoV1.1.png" class="logo" />
    <h1 class="title">BitesAndSips</h1>
    <APITokenSetter :apiToken="apiToken" @setToken="setAPIToken"></APITokenSetter>
    <BitesAndSipsFilters :filters="filters" @setErrorText="setErrorText" @search_places="search_places" :coordinates="coordinates"></BitesAndSipsFilters>
    <div v-if="places.length > 0">
      <v-card :loading="loading" class="mx-auto my-12" max-width="374">
        <template v-slot:loader="{ isActive }">
          <v-progress-linear
            :active="isActive"
            color="deep-purple"
            height="4"
            indeterminate
          ></v-progress-linear>
        </template>

        <v-img cover height="250" :src="places[currentPlaceIndex].photo"></v-img>

        <div class="info-container">
          <div class="left-button">
            <v-btn class="nav-btn" @click.prevent="previousPlace">&lt;</v-btn>
          </div>
          <div class="info-content">
            <v-card-item>
              <v-card-title>{{ places[currentPlaceIndex].name }}</v-card-title>

              <v-card-subtitle>
                <span class="me-1">{{ places[currentPlaceIndex].walking_distance }}</span>
              </v-card-subtitle>
            </v-card-item>

            <v-card-text>
              <v-row align="center" class="rating-row">
                <v-rating
                  v-model="internalRatingValue"
                  :max="5"
                  color="amber"
                  density="compact"
                  half-increments
                  readonly
                  size="small"
                ></v-rating>

                <div class="text-grey ms-4">
                  Rating: {{ places[currentPlaceIndex].rating }} ({{ places[currentPlaceIndex].user_ratings_total }})
                </div>
              </v-row>

              <div class="my-4 text-subtitle-1">
                {{ "$".repeat(places[currentPlaceIndex].price_level) }} • {{ places[currentPlaceIndex].type }}
              </div>

              <div>Opened for {{ Math.floor(places[currentPlaceIndex].opened_for.hours) }} hours and {{ Math.round(places[currentPlaceIndex].opened_for.minutes) }} minutes</div>
              <div><a :href="places[currentPlaceIndex].google_maps" target="_blank">Open in Google Maps</a></div>
            </v-card-text>
          </div>
          <div class="right-button">
            <v-btn class="nav-btn" @click.prevent="nextPlace">&gt;</v-btn>
          </div>
        </div>
        <v-divider class="mx-4 mb-1"></v-divider>

        <v-card-actions class="actions">
          <v-btn :class="getButtonClass(true)" variant="text" @click.prevent="like">
            <v-icon icon="mdi-thumb-up"></v-icon> Like
          </v-btn>
          <v-btn :class="getButtonClass(false)" variant="text" @click.prevent="dislike">
            <v-icon icon="mdi-thumb-down"></v-icon> Dislike
          </v-btn>
        </v-card-actions>
      </v-card>
    </div>
    <div class="loading-container" v-else>
      <p>Search for recommendation!</p>
      <v-progress-circular
      v-if="loading"
      indeterminate
      color="primary">
    </v-progress-circular>
    </div>
    <p color="red">{{errorText}}</p>
  </div>
</template>

<script>
import BitesAndSipsFilters from "./BitesAndSipsFilters.vue";
import APITokenSetter from "./APITokenSetter.vue";
import axios from "axios";

export default {
  name: "App",
  components: {
    BitesAndSipsFilters,
    APITokenSetter
  },
  data() {
    return {
      apiToken: '',
      coordinates: {
        latitude: '',
        longitude: ''
      },
      places: [],
      currentPlaceIndex: 0,
      filters: {
        maxDistance: 3,
        minTime: 1,
        placeType: 'restaurant'
      },
      likes: [],
      dislikes: [],
      loading: false,
      internalRatingValue: 0,
      errorText: ''
    }
  },
  created: function () {
    this.set_coordinates();
  },
  methods: {
    setAPIToken(token){
      this.apiToken = token;
      //localStorage.setItem("APIToken", token);
    },
    search_places(maxDistance, minTime, type, address) {
      this.loading = true;
      let params = {
        radius: maxDistance,
        min_time: minTime,
        type: type
      }
      let route = '';
      if(this.coordinates.latitude > 0){
        params.latitude = this.coordinates.latitude;
        params.longitude = this.coordinates.longitude;
        route = 'nearbysearch';
      }else {
        params.street = address.street;
        params.city = address.city;
        params.country = address.country;
        params.postal = address.zip;
        route = 'addresssearch';
      }
      axios
          .get("https://localhost/api/" + route, {params: params })
          .then((response) => {
            this.places = JSON.parse(response.data)
            this.sortPlacesByRating();
            this.currentPlaceIndex = 0;
            this.loading = false;
            if(this.places.length < 1){
              this.setErrorText('No results found');
              return;
            }
            
            this.internalRatingValue = this.places[this.currentPlaceIndex]?.rating ?? 0;
            // Hier kannst du die Daten aus der Antwort in deine Komponente einfügen
          })
          .catch((error) => {
            console.log(error);
            this.setErrorText('Error: No results found');
            this.loading = false;
          });      
    },
    sortPlacesByRating(){
      if(this.places == undefined ||this.places.length < 1) return;
      let sortedPlaces = this.places;
      sortedPlaces = sortedPlaces.sort((a, b) => (+(this.likes.includes(b.place_id))) - (+(this.likes.includes(a.place_id))));
      sortedPlaces = sortedPlaces.sort((a, b) => (+(this.dislikes.includes(a.place_id))) - (+(this.dislikes.includes(b.place_id))));
      this.places = sortedPlaces;
    },
    set_coordinates() {
      navigator.geolocation.getCurrentPosition(pos => {
        this.coordinates.longitude = pos.coords.longitude;
        this.coordinates.latitude = pos.coords.latitude;
      });
    },
    like() {
      this.loading = true;
      setTimeout(() => (this.loading = false), 2000);
      if(this.likes.includes(this.places[this.currentPlaceIndex].place_id)){
        this.removeRating();
      } else {
        this.rate('like');
      }   
    },
    dislike() {
      if(this.dislikes.includes(this.places[this.currentPlaceIndex].place_id)){
        this.removeRating;
      } else {
        this.rate('dislike');
      }  
    },
    previousPlace(){
      if (this.currentPlaceIndex > 0) {
        this.currentPlaceIndex--;
        this.internalRatingValue = this.places[this.currentPlaceIndex].rating;
      }
    },
    nextPlace(){
      if (this.currentPlaceIndex < this.places.length - 1) {
        this.currentPlaceIndex++;
        this.internalRatingValue = this.places[this.currentPlaceIndex].rating;
      }
    },
    rate(route){
      let currentId = this.places[this.currentPlaceIndex].place_id;
      axios.post('https://localhost/api/' + route, {'placeid': currentId})
          .then(() => {
            if(route == 'like'){
              this.likes.push(currentId);
              this.dislikes = this.dislikes.filter(id => id !== currentId);
            }else{
              this.likes = this.likes.filter(id => id !== currentId);
              this.dislikes.push(currentId);
            }
          })
          .catch(() => {
            this.setErrorText('An error has occurred')
          })
    },
    removeRating(){
      let currentId = this.places[this.currentPlaceIndex].place_id;
      axios.post('https://localhost/api/removerating', {'placeid': currentId})
          .then(() => {
            this.likes = this.likes.filter(id => id !== currentId);
            this.dislikes = this.dislikes.filter(id => id !== currentId);
          })
          .catch(() => {
            this.setErrorText('An error has occurred')
          })
    },
    getButtonClass(rating){
      if((rating && this.likes.includes(this.places[this.currentPlaceIndex].place_id)) || (!rating && this.dislikes.includes(this.places[this.currentPlaceIndex].place_id))){
        return 'active';
      } else {
        return 'inactive';
      }
    },
    setErrorText(text){
      this.errorText = text;
    }
  }
};
</script>

<style>
#App {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  color: #333333;
  min-height: 100vh;
}

.title {
  font-size: 40px;
  margin-top: 0;
}

.logo {
  width: 20%;
}

.auth-container {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.component {
  margin: 0 10px;
}

.info-container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}

.loading-container {
  display: flex;
  align-items: center;
  flex-direction: column;  
}

.left-button,
.right-button {
  flex: 0 0 auto;
  width: 36px;
}

.left-button > button, .right-button > button {
  min-width: 0;
}

.info-content {
  flex: 0 0 80%;
  text-align: center;
}

.actions {
  justify-content: space-evenly;
}

.nav-btn {
  padding: 0;
  background-color: #faecdd;
  border: none;
  cursor: pointer;
}

.rating-row {
  justify-content: center;
}

.active {
  background-color: #9575cd !important;
  color: white !important;
}

.inactive{
  color: #9575cd !important;
}

@media (max-width: 600px) {
  .half-size {
    width: 100%;
  }
}
</style>
