<template>
  <div class="bg-slate-100 h-screen flex flex-col justify-center space-y-5">
    <div class="mx-auto flex flex-row text-4xl mb-10">
      <p><a href="https://developers.google.com/maps" target="_blank" class="hover:underline">Google Maps</a> Integration
        with <a class="font-bold text-emerald-500 hover:underline" href="https://vuejs.org/guide/introduction.html"
          target="_blank">Vue 3</a> Option
        API</p>
    </div>
    <div class="flex flex-row justify-between w-2/3 mx-auto">
      <input ref="mapInputSearch" class="p-2 w-1/2 border shadow bg-white rounded-xl focus:ring-orange-400" type="text"
        placeholder="Search place here">
      <button ref="mapLocationButton" @click="loadCurrentLocation()"
        class="p-2 bg-white rounded-xl border hover:bg-gray-100 shadow">Get My
        Location</button>
    </div>
    <div ref="mapCanvas" id="mapCanvas" class="bg-white shadow border rounded-xl w-2/3 h-2/3 mx-auto object-cover" />
    <p class="mx-auto">Made with ❤️ by <a href="https://github.com/tigadika" target="_blank"
        class="hover:underline">tigadika</a></p>
  </div>
</template>

<script>
import { Loader } from "@googlemaps/js-api-loader"

export default {
  data() {
    return {
      loader: new Loader({
        apiKey: import.meta.env.VITE_GOOGLE_API_KEY,
        libraries: ['places']
      }),
      coords: {
        lat: -6.2440488,
        lng: 106.8862587,
      },
    }
  },
  created() {
    this.fetchCurrentLocation() //fail safe if fetch is not working
  },
  mounted() {
    this.loadGMaps()
  },
  methods: {
    fetchCurrentLocation() {
      navigator.geolocation.getCurrentPosition( //geolocation's browser
        (location) => { //success callback
          this.coords = {
            lat: location.coords.latitude,
            lng: location.coords.longitude
          }
        },
        () => { //error callback
          this.handleGeoLocationError()
        }
      );
    },

    async loadGMaps() {
      await this.loader.load() // load gmaps API
      const option = {
        center: this.coords,
        zoom: 11,
      }
      const map = new google.maps.Map(this.$refs.mapCanvas, option); // render maps to mapCanvas div
      const infoWindow = new google.maps.InfoWindow() // create info window element in gmaps


      // -- searchInput section --
      const searchBox = new google.maps.places.SearchBox(this.$refs.mapInputSearch)
      map.addListener("bounds_changed", () => {
        searchBox.setBounds(map.getBounds());
      });
      let markers = [];

      // Listen for the event fired when the user selects a prediction and retrieve
      // more details for that place.
      searchBox.addListener("places_changed", () => {
        const places = searchBox.getPlaces();

        if (places.length == 0) {
          return;
        }

        // Clear out the old markers.
        markers.forEach((marker) => {
          marker.setMap(null);
        });
        markers = [];

        // For each place, get name and location.
        const bounds = new google.maps.LatLngBounds();

        places.forEach((place, index) => {
          if (!place.geometry || !place.geometry.location) {
            console.log("Returned place contains no geometry");
            return;
          }

          // Create a marker for each place.
          // window.setTimeout(() => {
          markers.push(
            new google.maps.Marker({
              map,
              title: place.name,
              position: place.geometry.location,
            })
          );
          // }, index * 100);

          if (place.geometry.viewport) {
            // Only geocodes have viewport.
            bounds.union(place.geometry.viewport);
          } else {
            bounds.extend(place.geometry.location);
          }
        });
        console.log(markers);

        markers.forEach((marker) => {
          marker.addListener("click", () => {
            console.log("test");
            infoWindow.close();
            infoWindow.setContent(marker.getTitle());
            infoWindow.open(marker.getMap(), marker);
          });
        })

        map.fitBounds(bounds);
        map.setCenter(places[0].geometry.location)
      });
      // -- end of search input section --

      return { map, infoWindow } // return map and infoWindow element to future use
    },

    async loadCurrentLocation() {
      this.fetchCurrentLocation()
      const { map, infoWindow } = await this.loadGMaps()
      // this.findPlaceFunction()
      infoWindow.setPosition(this.coords)
      infoWindow.setContent(`<strong>You are here</strong><br>LatLng: (${this.coords.lat}, ${this.coords.lng})`)
      infoWindow.open(map)
      map.setCenter(this.coords)
      map.setZoom(17)
    },

    async handleGeoLocationError() {
      const { map, infoWindow } = await this.loadGMaps()
      infoWindow.setPosition(this.coords)
      infoWindow.setContent("Geo Location Service Error. It's either blocked by your browser or your browser doesn't support it")
      infoWindow.open(map)
      map.setCenter(this.coords)
    },
  },
  watch: {
  }
}
</script>

<style lang="scss" scoped></style>