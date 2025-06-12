<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Delivery Los Helechos</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
      background-color: #f2f2f2;
    }
    img {
      width: 250px;
      margin-bottom: 20px;
    }
    #map {
      height: 400px;
      width: 100%;
      margin-top: 20px;
    }
    input, button {
      padding: 10px;
      margin: 10px;
      font-size: 16px;
    }
    #resultado {
      margin-top: 20px;
      font-size: 18px;
    }
  </style>
</head>
<body>
  <img src="logo.png" alt="Logotipo Los Helechos" />
  <h1>Calculadora de Costo de Domicilio</h1>
  <p><strong>Ejemplo:</strong> Carrera 64 #81B-42, Barranquilla, Atlántico, Colombia</p>

  <input type="text" id="address" placeholder="Escribe tu dirección completa" />
  <button onclick="calculateDistance()">Calcular</button>

  <div id="map"></div>
  <div id="resultado"></div>

  <script>
    let map;
    let userMarker;
    let directionsRenderer;
    const restaurantLatLng = { lat: 11.02395, lng: -74.80899 };

    function initMap() {
      map = new google.maps.Map(document.getElementById("map"), {
        center: restaurantLatLng,
        zoom: 12,
      });

      new google.maps.Marker({
        position: restaurantLatLng,
        map: map,
        title: "Los Helechos",
      });
    }

    function calculateDistance() {
      const address = document.getElementById("address").value.trim();
      if (!address) {
        alert("Por favor, escribe una dirección.");
        return;
      }

      const geocoder = new google.maps.Geocoder();
      geocoder.geocode({ address }, (results, status) => {
        if (status === "OK") {
          const userLocation = results[0].geometry.location;
          const userLatLng = {
            lat: userLocation.lat(),
            lng: userLocation.lng()
          };

          if (userMarker) userMarker.setMap(null);

          userMarker = new google.maps.Marker({
            position: userLatLng,
            map: map,
            title: "Tu ubicación",
          });

          map.setCenter(userLatLng);

          const distanceService = new google.maps.DistanceMatrixService();
          distanceService.getDistanceMatrix({
            origins: [userLatLng],
            destinations: [restaurantLatLng],
            travelMode: google.maps.TravelMode.DRIVING,
          }, (response, status) => {
            if (status === "OK") {
              const distance = response.rows[0].elements[0].distance.value / 1000;
              const duration = response.rows[0].elements[0].duration.text;

              if (distance > 5) {
                alert("Fuera de cobertura. Máximo 5 km.");
                return;
              }

              let cost;
              if (distance > 3.5) cost = 6000;
              else if (distance > 2.5) cost = 4500;
              else if (distance > 1.5) cost = 3000;
              else cost = 2000;

              document.getElementById("resultado").innerHTML =
                "<p><strong>Distancia:</strong> " + distance.toFixed(2) + " km</p>" +
                "<p><strong>Tiempo estimado:</strong> " + duration + "</p>" +
                "<p><strong>Costo:</strong> $" + cost + "</p>";

              if (!directionsRenderer) {
                directionsRenderer = new google.maps.DirectionsRenderer({ map: map });
              }

              const directionsService = new google.maps.DirectionsService();
              directionsService.route({
                origin: userLatLng,
                destination: restaurantLatLng,
                travelMode: google.maps.TravelMode.DRIVING,
              }, (result, status) => {
                if (status === "OK") {
                  directionsRenderer.setDirections(result);
                }
              });
            } else {
              alert("Error al calcular distancia.");
            }
          });
        } else {
          alert("No se pudo encontrar la dirección. Intenta con otra.");
        }
      });
    }
  </script>

  <!-- ✅ API Key funcional para GitHub Pages -->
  <script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyA4pEN-vgTK5p7gYYovqMJROtw7n3BL4HI&callback=initMap&libraries=places" async defer></script>
</body>
</html>
