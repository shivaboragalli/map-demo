<template>
  <div>
    <input id="pac-input" class="controls" type="text" placeholder="Search Places" />
    <div id="map"></div>
  </div>
</template>

<script>
export default {
  name: "Map",

  methods: {
    initMap: async function() {
      console.log("google", window.google);
      const google = window.google;
      const input = document.getElementById("pac-input");

      var southWest = new google.maps.LatLng(12.972442, 77.580643);
      var northEast = new google.maps.LatLng(18.1124, 79.0193);
      var defaultBounds = new google.maps.LatLngBounds(southWest, northEast);

      var options = {
        bounds: defaultBounds,
        strictBounds: false,
        componentRestrictions: { country: "in" }
      };
      console.log("defaultBounds", defaultBounds);
      const autocomplete = new google.maps.places.Autocomplete(input, options);

      const map = new google.maps.Map(document.getElementById("map"), {
        zoom: 5,
        center: {
          lat: 12.972442,
          lng: 77.580643
        },
        mapTypeId: "terrain"
      });
      map.controls[google.maps.ControlPosition.TOP_LEFT].push(input);
      let marker = new google.maps.Marker({
        map: map,
        anchorPoint: new google.maps.Point(0, -29)
      });
      autocomplete.addListener("place_changed", () => {
        marker.setVisible(false);
        var place = autocomplete.getPlace();
        if (!place.geometry) {
          window.alert("No details available for input: '" + place.name + "'");
          return;
        }
        // If the place has a geometry, then present it on a map.
        if (place.geometry.viewport) {
          let lat = place.geometry.viewport.Va.i;
          let lng = place.geometry.viewport.Za.i;
          let withinRange = this.checkWithinBoundries(
            lat,
            lng,
            bermudaTriangle
          );
          if (!withinRange) {
            alert("searched place is out of range");
            return;
          }
          map.fitBounds(place.geometry.viewport);
        } else {
          let lat = place.geometry.location.lat();
          let lng = place.geometry.location.lng();
          let withinRange = this.checkWithinBoundries(
            lat,
            lng,
            bermudaTriangle
          );
          if (!withinRange) {
            alert("searched place is out of range");
            return;
          }
          map.setCenter(place.geometry.location);
          map.setZoom(17); // Why 17? Because it looks good.
        }
        marker.setPosition(place.geometry.location);
        marker.setVisible(true);
      });
      const triangleCoords = [
        {
          lat: 12.972442,
          lng: 77.580643
        },

        {
          lat: 16.705,
          lng: 74.2433
        },
        {
          lat: 18.1124,
          lng: 79.0193
        }
      ];
      const bermudaTriangle = new google.maps.Polygon({
        paths: triangleCoords,
        strokeColor: "#FF0000",
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: "#FF0000",
        fillOpacity: 0.35
      });
      bermudaTriangle.setMap(map);
    },
    checkWithinBoundries(lat, lng, bermudaTriangle) {
      let google = window.google;
      var bounds = new google.maps.LatLngBounds();
      bermudaTriangle
        .getPaths()
        .getArray()
        .forEach(function(path) {
          path.getArray().forEach(function(latLng) {
            bounds.extend(latLng);
          });
        });
      let withinRange = false;
      if (
        lat >= bounds.Va.i &&
        lat <= bounds.Va.j &&
        lng >= bounds.Za.i &&
        lng <= bounds.Za.j
      ) {
        withinRange = true;
      }
      console.log("within Range", withinRange);
      return withinRange;
    }
  },
  mounted() {
    this.initMap();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
#map {
  height: 100vh;
  width: 100%;
  top: 0;
}
.controls {
  width: 50%;
  margin-top: 10px;
  padding: 10px;
}
@media (max-width: 620px) {
  #pac-input {
    z-index: 0 !important;
    position: absolute !important;
    left: 10px !important;
    top: 70px !important;
    width: 50%;
  }
}
</style>
