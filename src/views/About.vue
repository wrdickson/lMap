<template>
  <div class="about">
    <l-map class="map1" ref="map" 
      :zoom=13 
      :center="[38.5733, -109.5498]"
    >
      
    </l-map>
    
  </div>
</template>

<script>
  /* eslint-disable */
  import { L, LMap, LTileLayer, LMarker  } from 'vue2-leaflet'
  import 'leaflet/dist/leaflet.css'
  import 'leaflet-draw'
  import 'leaflet-draw/dist/leaflet.draw.css'
  import 'font-awesome/css/font-awesome.css'
  import 'leaflet.awesome-markers'
  import 'leaflet.awesome-markers/dist/leaflet.awesome-markers.css'
  import Vue from 'vue'
  
  Vue.component('l-map', LMap);
  Vue.component('l-tile-layer', LTileLayer);
  Vue.component('l-marker', LMarker); 
  
  // this part resolve an issue where the markers would not appear
  delete L.Icon.Default.prototype._getIconUrl;

  L.Icon.Default.mergeOptions({
    iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
    iconUrl: require('leaflet/dist/images/marker-icon.png'),
    shadowUrl: require('leaflet/dist/images/marker-shadow.png')
  });
  export default{
    components: {
        LMap,
        LTileLayer,
        LMarker
    },
    data: () => ({
      map: null,
      pLayer: {}
    }),
    mounted () {
      
      this.$nextTick( () => {
        this.collection = {
          type: "FeatureCollection",
          features: []
        }
        this.map = this.$refs.map.mapObject; // work as expected
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(this.map);
        
        
        // Creates a red marker with the coffee icon
        var blackMarker = L.AwesomeMarkers.icon({
          icon: 'bullseye',
          iconColor: 'white',
          markerColor: 'black',
          prefix:'fa'
        }); 
        var redMarker = L.AwesomeMarkers.icon({
          icon: 'bullseye',
          iconColor: 'black',
          markerColor: 'red',
          prefix:'fa'
        });
            
        L.marker([38.5733, -109.5497], {icon: redMarker}).addTo(this.map)
        
        
        var editableLayers = new L.FeatureGroup();
        this.map.addLayer(editableLayers);
        
        var options = {
            position: 'topright',
            draw: {
                polyline: {
                    shapeOptions: {
                        color: '#000',
                        weight: 6
                    }
                },
                polygon: {
                    allowIntersection: false, // Restricts shapes to simple polygons
                    drawError: {
                        color: '#000', // Color the shape will turn when intersects
                        message: '<strong>Oh snap!<strong> you can\'t draw that!' // Message that will show when intersect
                    },
                    shapeOptions: {
                        color: '#000',
                        weight: 6
                    }
                },
                circle: false, // Turns off this drawing tool
                circlemarker: false,
                rectangle: false,
                
                marker: {
                    icon: blackMarker
                }
            },
            edit: {
                featureGroup: editableLayers, //REQUIRED!!
                remove: true
            }
        }; 
        
        
        var drawControl = new L.Control.Draw(options);
        this.map.addControl(drawControl);
        
        this.map.on(L.Draw.Event.CREATED, (e)=> {
            console.log("created",e);
           var type = e.layerType,
               layer = e.layer;
           if (type === 'marker') {
               // Do marker specific actions
           }
           // Do whatever else you need to. (save to db; add to map etc)
           this.map.addLayer(layer);
           editableLayers.addLayer(layer);
        });
        
       this.map.on('draw:edited', function (e) {
           console.log("edited",e);
           console.log( e.layers.toGeoJSON() );
           var layers = e.layers;
           layers.eachLayer(function (layer) {
               //do whatever you want; most likely save back to db
           });
       });
        
        
        
      })
    },
    name: "someMap"
  }


</script>

<style>
.map1{
  
  height: 600px !important;
}

</style>
