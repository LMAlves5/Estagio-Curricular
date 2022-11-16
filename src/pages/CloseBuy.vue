<template>
  <div class="ui grid">
    <div class="six wide column">
      <form class="ui segment large form">
        <div class="ui segment">
          <div class="field">
            <div class="ui right icon input large">
              <input
                type="text"
                placeholder="Enter your address"
                v-model="coordinates"
              />
              <i class="dot circle link icon" @click="locatorButtonPressed"></i>
            </div>
          </div>
          <div class="field">
            <div class="two fields">
              <div class="field">
                <select v-model="type">
                  <option value="restaurant">Restaurant</option>
                  <option value="supermarket">Supermarket</option>
                  <option value="gas_station">Gas Station</option>
                </select>
              </div>
              <div class="field">
                <select v-model="radius">
                  <option value="5">5 KM</option>
                  <option value="10">10 KM</option>
                  <option value="15">15 KM</option>
                  <option value="20">20 KM</option>
                </select>
              </div>
            </div>
          </div>
          <button class="ui button pink" @click="findCloseBuyButtonPressed">
            Find CloseBuy
          </button>
        </div>
      </form>

      <div class="ui segment" style="max-height: 500px; overflow: scroll">
        <div class="ui divided items">
          <div class="item" v-for="place in places" :key="place.id">
            <div class="content">
              <div class="header">{{ place.name }}</div>
              <div class="meta">{{ place.vicinity }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="ten wide column segment ui" ref="map"></div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      lat: 0,
      lng: 0,
      type: "",
      radius: "",
      places: [],
    };
  },
  computed: {
    coordinates() {
      return `${this.lat}, ${this.lng}`;
    },
  },
  mounted() {
    let map = document.createElement("script");
    map.setAttribute(
      "src",
      "https://maps.googleapis.com/maps/api/js?key=AIzaSyDkbf5MsXZ7eUqxFouSc0yylfOAjOHfgi4"
    );
    document.head.appendChild(map);
  },
  methods: {
    locatorButtonPressed() {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          this.lat = position.coords.latitude;
          this.lng = position.coords.longitude;
        },
        (error) => {
          console.log("Error getting location");
        }
      );
    },
    findCloseBuyButtonPressed() {
      const URL = `https://cors-anywhere.herokuapp.com/https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=${
        this.lat
      },${this.lng}
&type=${this.type}
&radius=${this.radius * 1000}
&key=AIzaSyDkbf5MsXZ7eUqxFouSc0yylfOAjOHfgi4`;
      axios
        .get(URL)
        .then((response) => {
          this.places = response.data.results;
          this.addLocationsToGoogleMaps();
        })
        .catch((error) => {
          console.log(error.message);
        });
    },
    addLocationsToGoogleMaps() {
      var map = new google.maps.Map(this.$refs["map"], {
        zoom: 15,
        center: new google.maps.LatLng(this.lat, this.lng),
        mapTypeId: google.maps.MapTypeId.ROADMAP,
      });

      var infowindow = new google.maps.InfoWindow();

      this.places.forEach((place) => {
        const lat = place.geometry.location.lat;
        const lng = place.geometry.location.lng;
        let marker = new google.maps.Marker({
          position: new google.maps.LatLng(lat, lng),
          map: map,
        });

        google.maps.event.addListener(marker, "click", () => {
          infowindow.setContent(
            `<div class="ui header">${place.name}</div><p>${place.vicinity}</p>`
          );
          infowindow.open(map, marker);
        });
      });
    },
  },
};
</script>