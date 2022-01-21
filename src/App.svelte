<script>
  import * as L from "leaflet";
  import markerIcon2x from "leaflet/dist/images/marker-icon-2x.png";
  import markerIcon from "leaflet/dist/images/marker-icon.png";
  import markerShadow from "leaflet/dist/images/marker-shadow.png";
  //fix for markers not showing up
  delete L.Icon.Default.prototype._getIconUrl;
  L.Icon.Default.mergeOptions({
    iconRetinaUrl: markerIcon2x,
    iconUrl: markerIcon,
    shadowUrl: markerShadow,
  });

  let map;
  let text;
  let FALLBACK_LATLNG = [0, 0];

  async function getAddress(text) {
    const data = await fetch(
      `https://geosearch.planninglabs.nyc/v1/search?text=${text}&size=1`
    )
      .then((res) => res.json())
      .catch((error) => ({ features: [] }, error));
    if (data.features)
      return [...data.features[0].geometry.coordinates].reverse();

    return FALLBACK_LATLNG;
  }

  async function _map(container) {
    const search = new URLSearchParams(window.location.search);
    text = search.get("q");
    let latlng = FALLBACK_LATLNG;

    //add case if cornell tech campus

    if (text) latlng = await getAddress(text);
    map = initMap(container, latlng);
    return {
      destroy: () => {
        map.remove();
      },
    };
  }

  function initMap(container, latlng) {
    const m = L.map(container).setView(latlng, 16);

    const settings = {
      maxZoom: 20,
      minZoom: 9,
      bounds: L.latLngBounds(
        [40.496133, -74.2555913],
        [40.9155327, -73.70000906]
      ),
    };

    L.control
      .layers(
        {
          osm: L.tileLayer(
            "https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png",
            {
              ...settings,
              attribution: `&copy;<a href="https://www.openstreetmap.org/copyright" target="_blank">OSM</a>,
		&copy;<a href="https://carto.com/attributions" target="_blank">CARTO</a>;
		<a href="https://geosearch.planninglabs.nyc/" target="_blank">GeoSearch</a>`,
              subdomains: "abcd",
            }
          ).addTo(m),
          "google satellite": L.tileLayer(
            "http://www.google.com/maps/vt?lyrs=s@189&gl=cn&x={x}&y={y}&z={z}",
            {
              ...settings,
              attribution: "&copy; Google",
            }
          ),
          "google road": L.tileLayer(
            "https://mt1.google.com/vt/lyrs=m&x={x}&y={y}&z={z}",
            {
              ...settings,
              attribution: "&copy; Google",
            }
          ),
        },
        {},
        { position: "bottomright", collapsed: true }
      )
      .addTo(m);

    m.zoomControl.setPosition("bottomleft");

    L.marker(latlng).addTo(m);

    return m;
  }
</script>

<main>
  <div id="details">
    <p><strong>{text}</strong></p>
    <p>
      View on <a
        href="https://maps.google.com/maps?f=q&source=s_q&hl=en&geocode=&q={text}"
        target="_blank"
        rel="noopener noreferrer">Google</a
      >;
    </p>
  </div>
  <div id="map" use:_map />
</main>

<svelte:head>
  <link
    type="text/css"
    rel="stylesheet"
    href="https://unpkg.com/leaflet/dist/leaflet.css"
  />
</svelte:head>

<style>
  #map {
    height: 100vh;
    width: 100vw;
  }

  #details {
    position: absolute;
    left: 0;
    top: 0;
    z-index: 999;
    margin: 0.2rem;
    padding: 0.5rem;
    background: white;
    border-radius: 2px;
    box-shadow: 1px 1px rgba(0, 0, 0, 0.5);
  }

  #details p{
	  margin: 0;
    font-size: 0.55rem;
  }
</style>
