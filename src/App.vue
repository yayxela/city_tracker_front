<script>
/* eslint-disable no-undef */
/* eslint-disable no-unused-vars */

import { ref, onMounted} from 'vue'
import {Loader} from '@googlemaps/js-api-loader'

const GOOGLE_MAPS_API_KEY = 'AIzaSyBIwzALxUPNbatRBj3Xi1Uhp0fFzwWNBkE'


export default {
  name: 'App',
  components: {

  },
  setup() {

    const locations = [
        {id: 1, lat: 52.2360592, lng: 21.002903599999968, name_point: 'A', title: 'text on hover'},
        {id: 2, lat: 52.2179967, lng: 21.222655600000053, name_point: 'B', title: 'text on hover'},
        {id: 3, lat: 52.2166692, lng: 20.993677599999955, name_point: 'C', title: 'text on hover'},
      ]

    const mapDiv = ref(null)
    const loader = new Loader({ apiKey: GOOGLE_MAPS_API_KEY})
    let markers = [];

    var map = ref(null);
    var directionsService;
    var directionsDisplay;

    onMounted(async () => {
      await loader.load()
      map = new google.maps.Map(mapDiv.value, {
        center: new google.maps.LatLng(0, 0),
        zoom: 7,
        mapTypeId: google.maps.MapTypeId.ROADMAP
      })
      directionsService = new google.maps.DirectionsService();
      

      setMarker(map, locations);
      showMarkers()
      if(markers !== []) {
        setWaypoints(map);
      }
      console.log(map);
      // map.value.setCenter(50,50);
      // deleteMarkers();

    })

    function setMarker(map, locations) {
      for(let i = 0; i < locations.length; i++) {
        let point = new google.maps.Marker({
          position: locations[i],
          map: map
        });
        markers.push(point);
      }
      
    }

    function setMapOnAll(map) {
      for (let i = 0; i < markers.length; i++) {
        markers[i].setMap(map);
      }
    }

    // Shows any markers currently in the array.
    function showMarkers() {
      setMapOnAll(map);
    }

    // Removes the markers from the map, but keeps them in the array.
    function clearMarkers() {
      setMapOnAll(null);
    }

    // Deletes all markers in the array by removing references to them.
    function deleteMarkers() {
      clearMarkers();
      markers = [];
      directionsDisplay = null;
    }

    function setWaypoints(map) {
      directionsDisplay = new google.maps.DirectionsRenderer({suppressMarkers:true});
      directionsDisplay.setMap(map);
      var start = markers[0].getPosition();
      var end = markers[markers.length - 1].getPosition();
      var waypts = [];
      for (var i = 1; i < markers.length - 1; i++) {
          waypts.push({
              location: markers[i].getPosition(),
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
            console.log(response)
              if(directionsDisplay)
              {
                directionsDisplay.setDirections(response);
              }
              
              // var route = response.routes[0];
              // var summaryPanel = document.getElementById('directions_panel');
              // summaryPanel.innerHTML = '';
              // // For each route, display summary information.
              // for (var i = 0; i < route.legs.length; i++) {
              //     var routeSegment = i + 1;
              //     summaryPanel.innerHTML += '<b>Route Segment: ' + routeSegment + '</b><br>';
              //     summaryPanel.innerHTML += route.legs[i].start_address + ' to ';
              //     summaryPanel.innerHTML += route.legs[i].end_address + '<br>';
              //     summaryPanel.innerHTML += route.legs[i].distance.text + '<br><br>';
              // }
          }
        });
    }
    
    return {mapDiv}

  },

}
</script>

<template>
  
  <div ref="mapDiv" style="width: 100%; height: 80vh"></div>
  <div class="d-flex text-center" style="height: 20vh">
    <!-- <div class="m-auto">

    </div> -->
  </div>
</template>



