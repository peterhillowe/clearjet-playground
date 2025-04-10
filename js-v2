// Use your provided Mapbox Access Token
mapboxgl.accessToken =
  'pk.eyJ1IjoicGV0ZXJoaWxsb3dlIiwiYSI6ImNtOWF2a3VhNjA4d2MyanB2d2NrN2w2d2MifQ.OjQvpqDjoukqqV_79_T_oQ';

const map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/mapbox/dark-v10',
  center: [-95, 40],
  zoom: 3
});

// Enable 3D globe projection.
map.on('style.load', () => {
  map.setProjection({ name: 'globe' });
});

// Haversine function: Computes the great‑circle distance (in miles) between two coordinates.
function haversineDistance(coord1, coord2) {
  const toRad = angle => angle * Math.PI / 180;
  const [lon1, lat1] = coord1;
  const [lon2, lat2] = coord2;
  const R = 3958.8; // Earth radius in miles
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);
  const a =
    Math.sin(dLat/2) ** 2 +
    Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) *
    Math.sin(dLon/2) ** 2;
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c;
}

// Updated mapping for multi‑leg journeys: all international origins (except Canadian ones)
// that require a multi‑leg path now map to their injection anchor.
// Hong Kong, Narita, Kansai, Fukuoka, Shanghai Pudong, Beijing Capital, Xi'an, Chengdu, Kunming all route to LAX.
const intlToDom = {
  "BER": "JFK",
  "BRU": "JFK",
  "CDG": "JFK",
  "CRL": "JFK",
  "FCO": "JFK",
  "FRA": "JFK",
  "LGW": "JFK",
  "LHR": "JFK",
  "MUC": "JFK",
  "MXP": "JFK",
  "ORY": "JFK",
  "TIJ": "SAN",
  "HKG": "LAX",
  "NRT": "LAX",
  "KIX": "LAX",
  "FUK": "LAX",
  "PVG": "LAX",
  "PEK": "LAX",
  "XIY": "LAX",
  "CTU": "LAX",
  "KMG": "LAX"
};

// Set of Canadian airport codes (non-stop routing).
const canadianCodes = ["YUL", "YVR", "YYC", "YYZ"];

// Consolidated airport dataset (duplicates removed).
const airportData = {
  type: "FeatureCollection",
  features: [
    // International Origins:
    { type: "Feature", id: "BER", properties: { code: "BER", name: "Berlin Brandenburg", origin: "international" }, geometry: { type: "Point", coordinates: [13.5033, 52.3667] } },
    { type: "Feature", id: "BRU", properties: { code: "BRU", name: "Brussels", origin: "international" }, geometry: { type: "Point", coordinates: [4.4844, 50.9014] } },
    { type: "Feature", id: "CDG", properties: { code: "CDG", name: "Paris Charles de Gaulle", origin: "international" }, geometry: { type: "Point", coordinates: [2.55, 49.0128] } },
    { type: "Feature", id: "CRL", properties: { code: "CRL", name: "Brussels South Charleroi", origin: "international" }, geometry: { type: "Point", coordinates: [4.4450, 50.4608] } },
    { type: "Feature", id: "FCO", properties: { code: "FCO", name: "Rome Fiumicino", origin: "international" }, geometry: { type: "Point", coordinates: [12.2462, 41.7999] } },
    { type: "Feature", id: "FRA", properties: { code: "FRA", name: "Frankfurt", origin: "international" }, geometry: { type: "Point", coordinates: [8.5706, 50.0333] } },
    { type: "Feature", id: "HKG", properties: { code: "HKG", name: "Hong Kong", origin: "international" }, geometry: { type: "Point", coordinates: [113.9185, 22.3080] } },
    { type: "Feature", id: "LGW", properties: { code: "LGW", name: "London Gatwick", origin: "international" }, geometry: { type: "Point", coordinates: [0.1667, 51.1481] } },
    { type: "Feature", id: "LHR", properties: { code: "LHR", name: "London Heathrow", origin: "international" }, geometry: { type: "Point", coordinates: [-0.4543, 51.4700] } },
    { type: "Feature", id: "MUC", properties: { code: "MUC", name: "Munich", origin: "international" }, geometry: { type: "Point", coordinates: [11.7861, 48.3538] } },
    { type: "Feature", id: "MXP", properties: { code: "MXP", name: "Milan Malpensa", origin: "international" }, geometry: { type: "Point", coordinates: [8.7222, 45.6306] } },
    { type: "Feature", id: "ORY", properties: { code: "ORY", name: "Paris Orly", origin: "international" }, geometry: { type: "Point", coordinates: [2.3599, 48.7233] } },
    // Tijuana remains international.
    { type: "Feature", id: "TIJ", properties: { code: "TIJ", name: "Tijuana", origin: "international" }, geometry: { type: "Point", coordinates: [-117.0333, 32.5333] } },
    // Canadian Origins:
    { type: "Feature", id: "YUL", properties: { code: "YUL", name: "Montreal-Trudeau", origin: "international" }, geometry: { type: "Point", coordinates: [-73.7406, 45.4706] } },
    { type: "Feature", id: "YVR", properties: { code: "YVR", name: "Vancouver", origin: "international" }, geometry: { type: "Point", coordinates: [-123.1815, 49.1947] } },
    { type: "Feature", id: "YYC", properties: { code: "YYC", name: "Calgary", origin: "international" }, geometry: { type: "Point", coordinates: [-114.0102, 51.1139] } },
    { type: "Feature", id: "YYZ", properties: { code: "YYZ", name: "Toronto Pearson", origin: "international" }, geometry: { type: "Point", coordinates: [-79.6306, 43.6777] } },
    // Major China Airports:
    { type: "Feature", id: "PEK", properties: { code: "PEK", name: "Beijing Capital", origin: "international" }, geometry: { type: "Point", coordinates: [116.4074, 39.9042] } },
    { type: "Feature", id: "PVG", properties: { code: "PVG", name: "Shanghai Pudong", origin: "international" }, geometry: { type: "Point", coordinates: [121.7997, 31.1443] } },
    { type: "Feature", id: "CAN", properties: { code: "CAN", name: "Guangzhou Baiyun", origin: "international" }, geometry: { type: "Point", coordinates: [113.3070, 23.1200] } },
    { type: "Feature", id: "SZX", properties: { code: "SZX", name: "Shenzhen Bao'an", origin: "international" }, geometry: { type: "Point", coordinates: [114.5435, 22.6393] } },
    { type: "Feature", id: "CTU", properties: { code: "CTU", name: "Chengdu Shuangliu", origin: "international" }, geometry: { type: "Point", coordinates: [104.0665, 30.5728] } },
    { type: "Feature", id: "XIY", properties: { code: "XIY", name: "Xi'an Xianyang", origin: "international" }, geometry: { type: "Point", coordinates: [108.9531, 34.2632] } },
    { type: "Feature", id: "KMG", properties: { code: "KMG", name: "Kunming Changshui", origin: "international" }, geometry: { type: "Point", coordinates: [102.9199, 25.4663] } },
    // Major Japan Airports:
    { type: "Feature", id: "NRT", properties: { code: "NRT", name: "Narita", origin: "international" }, geometry: { type: "Point", coordinates: [140.3929, 35.7720] } },
    { type: "Feature", id: "HND", properties: { code: "HND", name: "Tokyo Haneda", origin: "international" }, geometry: { type: "Point", coordinates: [139.7798, 35.5523] } },
    { type: "Feature", id: "KIX", properties: { code: "KIX", name: "Kansai International", origin: "international" }, geometry: { type: "Point", coordinates: [135.2433, 34.4347] } },
    { type: "Feature", id: "FUK", properties: { code: "FUK", name: "Fukuoka", origin: "international" }, geometry: { type: "Point", coordinates: [130.4518, 33.5853] } },
    // Domestic US Airports:
    { type: "Feature", id: "ATL", properties: { code: "ATL", name: "Atlanta", origin: "domestic" }, geometry: { type: "Point", coordinates: [-84.4277, 33.6407] } },
    { type: "Feature", id: "AUS", properties: { code: "AUS", name: "Austin", origin: "domestic" }, geometry: { type: "Point", coordinates: [-97.6699, 30.1944] } },
    { type: "Feature", id: "BNA", properties: { code: "BNA", name: "Nashville", origin: "domestic" }, geometry: { type: "Point", coordinates: [-86.6650, 36.1247] } },
    { type: "Feature", id: "BOI", properties: { code: "BOI", name: "Boise", origin: "domestic" }, geometry: { type: "Point", coordinates: [-116.2200, 43.5650] } },
    { type: "Feature", id: "BOS", properties: { code: "BOS", name: "Boston", origin: "domestic" }, geometry: { type: "Point", coordinates: [-71.0052, 42.3646] } },
    { type: "Feature", id: "CLT", properties: { code: "CLT", name: "Charlotte", origin: "domestic" }, geometry: { type: "Point", coordinates: [-80.9441, 35.2140] } },
    { type: "Feature", id: "CMH", properties: { code: "CMH", name: "Columbus", origin: "domestic" }, geometry: { type: "Point", coordinates: [-83.0, 39.98] } },
    { type: "Feature", id: "CVG", properties: { code: "CVG", name: "Cincinnati", origin: "domestic" }, geometry: { type: "Point", coordinates: [-84.4683, 39.0489] } },
    { type: "Feature", id: "DAL", properties: { code: "DAL", name: "Dallas Love Field", origin: "domestic" }, geometry: { type: "Point", coordinates: [-96.8517, 32.8471] } },
    { type: "Feature", id: "DEN", properties: { code: "DEN", name: "Denver", origin: "domestic" }, geometry: { type: "Point", coordinates: [-104.6737, 39.8561] } },
    { type: "Feature", id: "DFW", properties: { code: "DFW", name: "Dallas-Fort Worth", origin: "domestic" }, geometry: { type: "Point", coordinates: [-97.0403, 32.8998] } },
    { type: "Feature", id: "DTW", properties: { code: "DTW", name: "Detroit", origin: "domestic" }, geometry: { type: "Point", coordinates: [-83.3554, 42.2124] } },
    { type: "Feature", id: "ELP", properties: { code: "ELP", name: "El Paso", origin: "domestic" }, geometry: { type: "Point", coordinates: [-106.3961, 31.8070] } },
    { type: "Feature", id: "EWR", properties: { code: "EWR", name: "Newark", origin: "domestic" }, geometry: { type: "Point", coordinates: [-74.1687, 40.6925] } },
    { type: "Feature", id: "FLL", properties: { code: "FLL", name: "Fort Lauderdale", origin: "domestic" }, geometry: { type: "Point", coordinates: [-80.1486, 26.0726] } },
    { type: "Feature", id: "HOU", properties: { code: "HOU", name: "Houston Hobby", origin: "domestic" }, geometry: { type: "Point", coordinates: [-95.2772, 29.6454] } },
    { type: "Feature", id: "IAH", properties: { code: "IAH", name: "Houston Intercontinental", origin: "domestic" }, geometry: { type: "Point", coordinates: [-95.3414, 29.9844] } },
    { type: "Feature", id: "IND", properties: { code: "IND", name: "Indianapolis", origin: "domestic" }, geometry: { type: "Point", coordinates: [-86.2619, 39.7173] } },
    { type: "Feature", id: "JFK", properties: { code: "JFK", name: "New York JFK", origin: "domestic" }, geometry: { type: "Point", coordinates: [-73.7781, 40.6413] } },
    { type: "Feature", id: "LAS", properties: { code: "LAS", name: "Las Vegas", origin: "domestic" }, geometry: { type: "Point", coordinates: [-115.1523, 36.0840] } },
    { type: "Feature", id: "LAX", properties: { code: "LAX", name: "Los Angeles", origin: "domestic" }, geometry: { type: "Point", coordinates: [-118.4085, 33.9416] } },
    { type: "Feature", id: "MCO", properties: { code: "MCO", name: "Orlando", origin: "domestic" }, geometry: { type: "Point", coordinates: [-81.3081, 28.4312] } },
    { type: "Feature", id: "MDW", properties: { code: "MDW", name: "Chicago Midway", origin: "domestic" }, geometry: { type: "Point", coordinates: [-87.7539, 41.7868] } },
    { type: "Feature", id: "MIA", properties: { code: "MIA", name: "Miami", origin: "domestic" }, geometry: { type: "Point", coordinates: [-80.2906, 25.7959] } },
    { type: "Feature", id: "ORD", properties: { code: "ORD", name: "Chicago O'Hare", origin: "domestic" }, geometry: { type: "Point", coordinates: [-87.9048, 41.9786] } },
    { type: "Feature", id: "PDX", properties: { code: "PDX", name: "Portland", origin: "domestic" }, geometry: { type: "Point", coordinates: [-122.5951, 45.5887] } },
    { type: "Feature", id: "PHL", properties: { code: "PHL", name: "Philadelphia", origin: "domestic" }, geometry: { type: "Point", coordinates: [-75.2444, 39.8744] } },
    { type: "Feature", id: "PHX", properties: { code: "PHX", name: "Phoenix", origin: "domestic" }, geometry: { type: "Point", coordinates: [-112.0110, 33.4342] } },
    { type: "Feature", id: "SAN", properties: { code: "SAN", name: "San Diego", origin: "domestic" }, geometry: { type: "Point", coordinates: [-117.1611, 32.7338] } },
    { type: "Feature", id: "SEA", properties: { code: "SEA", name: "Seattle", origin: "domestic" }, geometry: { type: "Point", coordinates: [-122.3321, 47.4502] } },
    { type: "Feature", id: "SFO", properties: { code: "SFO", name: "San Francisco", origin: "domestic" }, geometry: { type: "Point", coordinates: [-122.3790, 37.6213] } },
    { type: "Feature", id: "SLC", properties: { code: "SLC", name: "Salt Lake City", origin: "domestic" }, geometry: { type: "Point", coordinates: [-111.9711, 40.7884] } },
    { type: "Feature", id: "STL", properties: { code: "STL", name: "St. Louis", origin: "domestic" }, geometry: { type: "Point", coordinates: [-90.3700, 38.7487] } },
    { type: "Feature", id: "TPA", properties: { code: "TPA", name: "Tampa", origin: "domestic" }, geometry: { type: "Point", coordinates: [-82.5332, 27.9759] } }
  ]
};

// Helper: Retrieve an airport feature by its code.
function getAirportFeature(code) {
  return airportData.features.find(f => f.properties.code === code);
}

// Define the layer ID for the route lines.
const routeLayerId = 'route-lines';

// Use click events for flight route drawing.
let selectedId = null;

// Helper: Compute and display routes for a given feature, then center the view.
function showRoutesForFeature(feature) {
  const code = feature.properties.code;
  const originCoord = feature.geometry.coordinates;
  const originType = feature.properties.origin;
  let routeFeatures = [];
  const minDistance = 600;
  
  if (originType === 'international') {
    if (canadianCodes.includes(code)) {
      // For Canadian international origins, draw nonstop routes.
      const domesticDests = airportData.features.filter(f =>
        f.properties.origin === 'domestic' &&
        haversineDistance(originCoord, f.geometry.coordinates) >= minDistance
      );
      domesticDests.forEach(dest => {
        const route = turf.greatCircle(turf.point(originCoord), turf.point(dest.geometry.coordinates), { npoints: 100 });
        routeFeatures.push(route);
      });
    } else {
      // Multi-leg journey for non-Canadian international origins.
      const injectionAnchor = intlToDom[code];
      if (injectionAnchor) {
        const anchorFeature = getAirportFeature(injectionAnchor);
        if (anchorFeature) {
          const anchorCoord = anchorFeature.geometry.coordinates;
          // Leg 1: from international origin to injection anchor.
          const leg1 = turf.greatCircle(turf.point(originCoord), turf.point(anchorCoord), { npoints: 100 });
          routeFeatures.push(leg1);
          // Leg 2: from injection anchor to all domestic airports (>=600 miles).
          const domesticDests = airportData.features.filter(f =>
            f.properties.origin === 'domestic' &&
            haversineDistance(anchorCoord, f.geometry.coordinates) >= minDistance
          );
          domesticDests.forEach(dest => {
            const leg2 = turf.greatCircle(turf.point(anchorCoord), turf.point(dest.geometry.coordinates), { npoints: 100 });
            routeFeatures.push(leg2);
          });
        }
      }
    }
  } else if (originType === 'domestic') {
    // Domestic routing: nonstop routes from origin to every other domestic airport (>=600 miles).
    const domesticDests = airportData.features.filter(f =>
      f.properties.origin === 'domestic' &&
      f.properties.code !== code &&
      haversineDistance(originCoord, f.geometry.coordinates) >= minDistance
    );
    domesticDests.forEach(dest => {
      const route = turf.greatCircle(turf.point(originCoord), turf.point(dest.geometry.coordinates), { npoints: 100 });
      routeFeatures.push(route);
    });
  }
  
  const routeGeoJSON = {
    type: "FeatureCollection",
    features: routeFeatures
  };
  
  // Remove any existing route layer and source.
  if (map.getLayer(routeLayerId)) map.removeLayer(routeLayerId);
  if (map.getSource(routeLayerId)) map.removeSource(routeLayerId);
  
  // Add new route source and layer.
  map.addSource(routeLayerId, { type: "geojson", data: routeGeoJSON });
  map.addLayer({
    id: routeLayerId,
    type: "line",
    source: routeLayerId,
    layout: {},
    paint: {
      "line-color": "#23F2F5",
      "line-width": 2,
      "line-dasharray": [2, 4],
      "line-opacity": 1
    }
  });
  
  // Center the map on the selected feature.
  map.easeTo({
    center: originCoord,
    duration: 1500
  });
}

map.on('load', function () {
  // Add the airport data as a GeoJSON source.
  map.addSource('airports', {
    type: 'geojson',
    data: airportData
  });
  
  // Add a symbol layer for airport markers.
  map.addLayer({
    id: 'airport-layer',
    type: 'symbol',
    source: 'airports',
    layout: {
      // For TIJ, use "truck-15" from Mapbox's sprite library; otherwise use "airport-15".
      "icon-image": [
        "case",
        ["==", ["get", "code"], "TIJ"],
          "truck-15",
        "airport-15"
      ],
      "icon-size": 1.2,
      "text-field": [
        "concat",
        ["get", "name"],
        " (",
        ["get", "code"],
        ")"
      ],
      "text-font": ["Open Sans Bold", "Arial Unicode MS Bold"],
      "text-size": 12,
      "text-offset": [0, 1.2],
      "text-anchor": "top"
    },
    paint: {
      "text-color": "#ffffff",
      "text-halo-color": "#0038E3",
      "text-halo-width": 1
    }
  });
  
  // Fit the map to display all airport markers.
  const coords = airportData.features.map(f => f.geometry.coordinates);
  const bounds = coords.reduce((b, coord) => b.extend(coord),
    new mapboxgl.LngLatBounds(coords[0], coords[0])
  );
  map.fitBounds(bounds, { padding: 50 });
  
  // Set up click behavior to draw flight routes.
  map.on('click', 'airport-layer', function(e) {
    // Prevent the click from propagating to the map background.
    e.originalEvent.stopPropagation();
    const feature = e.features[0];
    // If a different airport was previously selected, clear its state.
    if (selectedId && selectedId !== feature.id) {
      map.setFeatureState({ source: 'airports', id: selectedId }, { hover: false });
    }
    selectedId = feature.id;
    map.setFeatureState({ source: 'airports', id: feature.id }, { hover: true });
    
    // Draw routes for the selected airport.
    showRoutesForFeature(feature);
  });
  
  // Clicking on the map (non-marker) clears the selection and routes.
  map.on('click', function(e) {
    const features = map.queryRenderedFeatures(e.point, { layers: ['airport-layer'] });
    if (!features.length && selectedId) {
      map.setFeatureState({ source: 'airports', id: selectedId }, { hover: false });
      selectedId = null;
      if (map.getLayer(routeLayerId)) map.removeLayer(routeLayerId);
      if (map.getSource(routeLayerId)) map.removeSource(routeLayerId);
    }
  });
  
  // Pre-select LAX and center the map on it.
  const laxFeature = getAirportFeature("LAX");
  if (laxFeature) {
    selectedId = laxFeature.id;
    map.setFeatureState({ source: 'airports', id: laxFeature.id }, { hover: true });
    showRoutesForFeature(laxFeature);
  }
});
