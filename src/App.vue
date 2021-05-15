<script>
/* eslint-disable no-undef */
/* eslint-disable no-unused-vars */

import { ref, onMounted} from 'vue'
import {Loader} from '@googlemaps/js-api-loader'
import axios from 'axios'

const GOOGLE_MAPS_API_KEY = 'AIzaSyB41DRUbKWJHPxaFjMAwdrzWzbVKartNGg'

export default {
  name: 'App',
  components: {
  },
  data: function() {
    return {
      selectedUser: '',
      users: [],
      geo: [],
      points: [],
      map: null
    }
    
  },

  mounted() {
    axios.get('http://127.0.0.1:8000/api/users/')
      .then(response => {
        this.users = response.data
      })
      .catch(e => {
        console.log(e);
      });

    const loader = new Loader({ apiKey: GOOGLE_MAPS_API_KEY});
    loader
      .load()
      .then(() => { this.initMap(); })
      .catch(e => { console.log(e) });

  },

  watch: {
    selectedUser: function(user) {
      this.setMarkers(user);
    }

  },

  methods: {
    initMap() {
      console.log("initMap");
      this.map = new google.maps.Map(document.getElementById('map'), {
        center: new google.maps.LatLng(0, 0),
        zoom: 15,
        mapTypeId: google.maps.MapTypeId.ROADMAP
      })
    },

    setMarkers() {
      axios.get('http://127.0.0.1:8000/api/geo/')
      .then((response) => {
        this.geo = response.data;
        this.map.setCenter(new google.maps.LatLng(this.geo[0].lat, this.geo[0].lng));
        let points = this.setMarkerOnMap();
        this.showMarkers(points);
        
        // this.setWaypoints(points);
      })
      .catch(e => {
        console.log(e);
      });      
    },

    setMarkerOnMap() {
      console.log("setMarkerOnMap");
      let points = [];
      let geoPoints = this.geo;
      for(let i = 0; i < geoPoints.length; i++) {
        let point = new google.maps.Marker({
          position: { lat: geoPoints[i].lat, lng: geoPoints[i].lng },
          map: this.map
        });
        points.push(point);
        console.log(point);
      } 
      return points;
    },

    setMapOnAll(map, points) {
      for (let i = 0; i < points.length; i++) {
        console.log(points[i]);
        points[i].setMap(map);
      }
    },

    // Shows any markers currently in the array.
    showMarkers(points) {
      this.setMapOnAll(this.map, points);
    },

    // Removes the markers from the map, but keeps them in the array.
    clearMarkers() {
      this.setMapOnAll(null, points);
    },

    // Deletes all markers in the array by removing references to them.
    deleteMarkers() {
      this.clearMarkers();
      this.points = []
      directionsDisplay = null;
    },

    setWaypoints(points) {
      var directionsService = new google.maps.DirectionsService();
      var directionsDisplay = new google.maps.DirectionsRenderer({suppressMarkers:true});

      directionsDisplay.setMap(this.map);
      let start = points[0].getPosition();
      let end = points[points.length - 1].getPosition();

      let waypts = [];
      for (let i = 1; i < points.length - 1; i++) {
          waypts.push({
              location: points[i].getPosition(),
              stopover: true
          });
      }

      var request = {
        origin: start,
        destination: end,
        waypoints: waypts,
        optimizeWaypoints: true,
        travelMode: google.maps.TravelMode.DRIVING
      };
      
      directionsService.route(request, function (response, status) {
          if (status == google.maps.DirectionsStatus.OK) {
              if(directionsDisplay)
              {
                directionsDisplay.setDirections(response);
              }
              waypts_response = response
          }
        });
    }

  }

}
</script>

<template>
  <div>
    <div id="map" style="width: 100%; height: 80vh"></div>
    <div class="d-flex text-center" style="height: 20vh">
      <select name="userSelect" v-model="selectedUser">
        <option v-for="user in users" v-bind:key="user.id" v-bind:value="user.imei" v-on:change="setMarkers"> {{ user.imei }}</option>
      </select>
      <!-- <div class="m-auto">

      </div> -->
    </div>
  </div>
  
</template>



