<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vicinity Finder</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
        }
        #map {
            height: 500px;
            width: 100%;
            margin-top: 20px;
        }
    </style>
</head>
<body>

    <h2>🌍 Find Nearby Places</h2>
    <button onclick="getLocation()">Find My Location</button>
    <p id="status"></p>
    <div id="map"></div>

    <script>
        let map;
        let userMarker;

        function initMap() {
            map = new google.maps.Map(document.getElementById("map"), {
                center: { lat: 40.748817, lng: -73.985428 }, // Default: NYC
                zoom: 13,
            });
        }

        function getLocation() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(showPosition, showError);
                document.getElementById("status").innerText = "🔍 Finding your location...";
            } else {
                alert("Geolocation is not supported by this browser.");
            }
        }

        function showPosition(position) {
            let userLat = position.coords.latitude;
            let userLng = position.coords.longitude;
            
            let userLocation = new google.maps.LatLng(userLat, userLng);
            map.setCenter(userLocation);

            // Remove existing marker if present
            if (userMarker) {
                userMarker.setMap(null);
            }

            // Add marker for user's location
            userMarker = new google.maps.Marker({
                position: userLocation,
                map: map,
                title: "📍 You are here",
                icon: "http://maps.google.com/mapfiles/ms/icons/red-dot.png"
            });

            document.getElementById("status").innerText = "✅ Location found! Searching for nearby places...";
            
            findNearbyPlaces(userLat, userLng);
        }

        function showError(error) {
            switch(error.code) {
                case error.PERMISSION_DENIED:
                    alert("❌ Location permission denied.");
                    break;
                case error.POSITION_UNAVAILABLE:
                    alert("⚠️ Location information is unavailable.");
                    break;
                case error.TIMEOUT:
                    alert("⏳ Location request timed out.");
                    break;
                default:
                    alert("❌ Unknown error.");
            }
        }

        function findNearbyPlaces(lat, lng) {
            let location = new google.maps.LatLng(lat, lng);
            let request = {
                location: location,
                radius: '2000', // Search within 2 km
                type: ['restaurant', 'cafe', 'park', 'museum']
            };

            let service = new google.maps.places.PlacesService(map);
            service.nearbySearch(request, function(results, status) {
                if (status === google.maps.places.PlacesServiceStatus.OK) {
                    for (let i = 0; i < results.length; i++) {
                        let place = results[i];
                        let marker = new google.maps.Marker({
                            position: place.geometry.location,
                            map: map,
                            title: place.name,
                            icon: "http://maps.google.com/mapfiles/ms/icons/blue-dot.png"
                        });

                        let infowindow = new google.maps.InfoWindow();
                        google.maps.event.addListener(marker, 'click', function() {
                            infowindow.setContent(`<strong>${place.name}</strong><br>📍 ${place.vicinity}`);
                            infowindow.open(map, this);
                        });
                    }
                    document.getElementById("status").innerText = "✅ Nearby places found!";
                } else {
                    document.getElementById("status").innerText = "⚠️ No places found nearby.";
                }
            });
        }
    </script>

    <!-- Google Maps API (Replace YOUR_API_KEY with your actual API key) -->
    <script async defer src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places&callback=initMap"></script>

</body>
</html>

