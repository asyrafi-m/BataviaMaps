<script>
  import { onMount } from 'svelte';
  import L from 'leaflet';
  import 'leaflet/dist/leaflet.css';

  let map;
  let currentYear = "1619";
  let baseTileLayer;
  let showBaseMap = false;

  const mapLayers = {
    "1619": ["Land", "Buildings", "ImportantBuildings", "Water", "tambalan", "BentengBatavia", "Jakarta"],
    "1622": ["Land", "Buildings", "ImportantBuildings", "tambalan", "Water", "BentengBatavia"],
    "1627": ["Land", "Buildings", "ImportantBuildings", "tambalan", "Water", "BentengBatavia"],
    "1632": ["Land", "Buildings", "ImportantBuildings", "tambalan", "Water", "BentengBatavia"]
  };

  const layerStyles = {
    Land: { pane: "LandPane", color: "#000000", weight: 0.5, fillColor: '#ffffff', fillOpacity: 1, interactive: false },
    Buildings: { pane: "BuildingsPane", color: "#dddcdf", weight: 0, fillOpacity: 1, interactive: false },
    ImportantBuildings: { pane: "importantBuildingPane", color: "#000000", fillColor: '#aaaaaa', weight: 1, fillOpacity: 1, interactive: false },
    tambalan: { pane: "tambalanPane", color: "#ffffff", weight: 1, fillOpacity: 1, interactive: false },
    Water: { pane: "waterPane", color: "#9ec9d2", weight: 0, fillOpacity: 1, interactive: false },
    BentengBatavia: { pane: "fortPane", color: "#f9882e", weight: 3, opacity: 0.3, interactive: false },
    Jakarta: { pane: "jakartaPane", color: "#f9882e", weight: 0, fillColor: '#f9882e', fillOpacity: 0.1, interactive: true }
  };

  let geoJsonLayers = [];

  function loadMapLayers(year) {
    // Clear current layers
    geoJsonLayers.forEach(layer => map.removeLayer(layer));
    geoJsonLayers = [];

    // Add new layers
    mapLayers[year].forEach(name => {
      fetch(`/maps/${year}/${name}.geojson`)
        .then(res => res.json())
        .then(data => {
          const layer = L.geoJSON(data, {
            style: layerStyles[name],
            onEachFeature: (feature, layer) => {
              if (name === "Jakarta") {
                layer.on({
                  mouseover: (e) => handleMouseOver(e),
                  mouseout: (e) => handleMouseOut(e),
                  click: (e) => handleClick(e)
                });
              }
            }
          }).addTo(map);
          geoJsonLayers.push(layer);
        });
    });
  }

  function handleMouseOver(e) {
    e.target.setStyle({
      fillOpacity: 0.4 // Change opacity on hover
    });
  }

  function handleMouseOut(e) {
    e.target.setStyle({
      fillOpacity: 0.1 // Reset opacity
    });
  }

  function handleClick(e) {
    alert("Jakarta GeoJSON clicked!");
  }

  function switchMap(year) {
    const container = document.getElementById("map-container");
    const mapElement = document.getElementById("map");

    // Apply fade-out transition
    container.classList.remove("fade-in");
    container.classList.add("fade-out");

    // Wait for fade-out to complete before changing map
    setTimeout(() => {
      currentYear = year;
      loadMapLayers(year);

      // Apply fade-in transition after map layers have been loaded
      container.classList.remove("fade-out");
      container.classList.add("fade-in");
    }, 500); // Adjust duration to match transition timing (500ms for fade-out)
  }

  function toggleBaseMap() {
    if (showBaseMap) {
      map.removeLayer(baseTileLayer);
    } else {
      baseTileLayer.addTo(map);
    }
    showBaseMap = !showBaseMap;
  }

  onMount(() => {
    map = L.map("map", {
        minZoom: 1,
        maxZoom: 20,
        zoomAnimation: true,
        zoomAnimationThreshold: 4,
        zoomSnap: 0.25,
        zoomDelta: 0.25
    }).setView([-6.1335, 106.8129], 15.5);

    // Add the base tile layer
    baseTileLayer = L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap contributors"
    });

    // ✅ Define custom panes before loading GeoJSON layers
    map.createPane("LandPane");
    map.getPane("LandPane").style.zIndex = 397;

    map.createPane("tambalanPane");
    map.getPane("tambalanPane").style.zIndex = 398;

    map.createPane("waterPane");
    map.getPane("waterPane").style.zIndex = 399;

    map.createPane("BuildingsPane");
    map.getPane("BuildingsPane").style.zIndex = 400;

    map.createPane("importantBuildingPane");
    map.getPane("importantBuildingPane").style.zIndex = 401;

    map.createPane("jakartaPane");
    map.getPane("jakartaPane").style.zIndex = 402;

    map.createPane("fortPane");
    map.getPane("fortPane").style.zIndex = 403;

    loadMapLayers(currentYear);
  });
</script>

<style>
  #map {
    height: 100vh;
    width: 100%;
    display: flex;
    position: absolute; 
    left: 0;
    bottom: 0;
    z-index: 0;
    background-color:#fff;
    transition: opacity 0.5s ease-in-out;
  }
  label {
    color: #000;
  }
  .map-toggle {
    position: absolute;
    display: flex;
    flex-direction: column;
    top: 1rem;
    right: 1rem;
    background: white;
    padding: 0.5rem;
    z-index: 1000;
    border-radius: 8px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
    width: auto;
  }

  .map-toggle input[type="radio"] {
    margin-right: 5px;
  }

  #map-container.fade-out #map {
    opacity: 0;
  }

  #map-container.fade-in #map {
    opacity: 1;
  }
</style>

<div class="map-toggle">
  <label for="map-select" style="color: #000;">Select Map:</label>
  
  <!-- Radio Buttons for Year Selection -->
  <label>
    <input type="radio" name="year" value="1619" bind:group={currentYear} on:change={() => switchMap("1619")}> 1619
  </label>
  <label>
    <input type="radio" name="year" value="1622" bind:group={currentYear} on:change={() => switchMap("1622")}> 1622
  </label>
  <label>
    <input type="radio" name="year" value="1627" bind:group={currentYear} on:change={() => switchMap("1627")}> 1627
  </label>
  <label>
    <input type="radio" name="year" value="1632" bind:group={currentYear} on:change={() => switchMap("1632")}> 1632
  </label>

  <button on:click={toggleBaseMap} style="margin-top: 0.5rem;">
    {showBaseMap ? "Hide Base Map" : "Show Base Map"}
  </button>
</div>

<div id="map-container" class="fade-in">
  <div id="map"></div>
</div>