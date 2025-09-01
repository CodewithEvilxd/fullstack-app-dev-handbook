# Lesson 16: Maps & Location Services

## 🎯 **Learning Objectives**
- Integrate maps in React Native applications
- Handle location permissions and services
- Display interactive maps with markers and overlays
- Implement geolocation features
- Create location-based applications

## 📚 **Table of Contents**
1. [Introduction to Maps Integration](#introduction-to-maps-integration)
2. [Location Permissions](#location-permissions)
3. [Geolocation API](#geolocation-api)
4. [Map Integration](#map-integration)
5. [Markers and Annotations](#markers-and-annotations)
6. [Map Interactions](#map-interactions)
7. [Routing and Directions](#routing-and-directions)
8. [Geofencing](#geofencing)
9. [Offline Maps](#offline-maps)
10. [Practical Examples](#practical-examples)

---

## 🗺️ **Introduction to Maps Integration**

Maps integration is crucial for location-based applications. React Native offers several libraries for map integration and location services.

### **Popular Map Libraries**
- **react-native-maps**: Most popular map library
- **expo-location**: Location services for Expo projects
- **react-native-geolocation-service**: Enhanced geolocation service
- **Mapbox**: Alternative mapping solution

### **Use Cases**
- **Ride-sharing apps**: Real-time location tracking
- **Delivery apps**: Route optimization and tracking
- **Social apps**: Location-based content discovery
- **Travel apps**: Points of interest and navigation
- **Fitness apps**: Route tracking and analytics

---

## 🔐 **Location Permissions**

### **Requesting Location Permissions**
```javascript
import { PermissionsAndroid, Platform, Alert } from 'react-native';

const requestLocationPermission = async () => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
        {
          title: 'Location Permission',
          message: 'This app needs access to your location for better experience',
          buttonNeutral: 'Ask Me Later',
          buttonNegative: 'Cancel',
          buttonPositive: 'OK',
        }
      );

      if (granted === PermissionsAndroid.RESULTS.GRANTED) {
        console.log('Location permission granted');
        return true;
      } else {
        console.log('Location permission denied');
        return false;
      }
    } catch (err) {
      console.warn(err);
      return false;
    }
  } else {
    // iOS permissions are handled automatically
    return true;
  }
};
```

### **Background Location Permission (Android)**
```javascript
const requestBackgroundLocationPermission = async () => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.ACCESS_BACKGROUND_LOCATION,
        {
          title: 'Background Location Permission',
          message: 'This app needs background location access for continuous tracking',
          buttonNeutral: 'Ask Me Later',
          buttonNegative: 'Cancel',
          buttonPositive: 'OK',
        }
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    } catch (err) {
      console.warn(err);
      return false;
    }
  }
  return true;
};
```

---

## 📍 **Geolocation API**

### **Getting Current Location**
```javascript
import Geolocation from 'react-native-geolocation-service';

const getCurrentLocation = () => {
  return new Promise((resolve, reject) => {
    Geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude, accuracy } = position.coords;
        resolve({
          latitude,
          longitude,
          accuracy,
          timestamp: position.timestamp,
        });
      },
      (error) => {
        reject(error);
      },
      {
        enableHighAccuracy: true,
        timeout: 15000,
        maximumAge: 10000,
        distanceFilter: 10,
      }
    );
  });
};
```

### **Watching Location Changes**
```javascript
import React, { useEffect, useState } from 'react';
import { View, Text, StyleSheet } from 'react-native';
import Geolocation from 'react-native-geolocation-service';

const LocationTracker = () => {
  const [location, setLocation] = useState(null);
  const [watching, setWatching] = useState(false);

  useEffect(() => {
    let watchId = null;

    if (watching) {
      watchId = Geolocation.watchPosition(
        (position) => {
          setLocation(position.coords);
        },
        (error) => {
          console.error('Location watch error:', error);
        },
        {
          enableHighAccuracy: true,
          distanceFilter: 10,
          interval: 5000,
          fastestInterval: 2000,
        }
      );
    }

    return () => {
      if (watchId) {
        Geolocation.clearWatch(watchId);
      }
    };
  }, [watching]);

  const toggleWatching = () => {
    setWatching(!watching);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Location Tracker</Text>

      {location && (
        <View style={styles.locationInfo}>
          <Text>Latitude: {location.latitude.toFixed(6)}</Text>
          <Text>Longitude: {location.longitude.toFixed(6)}</Text>
          <Text>Accuracy: {location.accuracy.toFixed(2)} meters</Text>
          <Text>Speed: {location.speed ? location.speed.toFixed(2) : 'N/A'} m/s</Text>
        </View>
      )}

      <TouchableOpacity style={styles.button} onPress={toggleWatching}>
        <Text style={styles.buttonText}>
          {watching ? 'Stop Tracking' : 'Start Tracking'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
  },
  locationInfo: {
    backgroundColor: '#f0f0f0',
    padding: 15,
    borderRadius: 10,
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default LocationTracker;
```

---

## 🗺️ **Map Integration**

### **Basic Map Display**
```javascript
import React from 'react';
import { View, StyleSheet } from 'react-native';
import MapView from 'react-native-maps';

const BasicMap = () => {
  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
        showsUserLocation={true}
        showsMyLocationButton={true}
        zoomEnabled={true}
        scrollEnabled={true}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
});

export default BasicMap;
```

### **Custom Map Styles**
```javascript
import React, { useState } from 'react';
import { View, TouchableOpacity, Text, StyleSheet } from 'react-native';
import MapView, { PROVIDER_GOOGLE } from 'react-native-maps';

const CustomMap = () => {
  const [mapType, setMapType] = useState('standard');

  const mapStyles = {
    standard: [],
    satellite: [],
    terrain: [],
    hybrid: [],
  };

  const customStyle = [
    {
      elementType: 'geometry',
      stylers: [{ color: '#242f3e' }],
    },
    {
      elementType: 'labels.text.fill',
      stylers: [{ color: '#746855' }],
    },
    {
      elementType: 'labels.text.stroke',
      stylers: [{ color: '#242f3e' }],
    },
  ];

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        provider={PROVIDER_GOOGLE}
        mapType={mapType}
        customMapStyle={mapType === 'custom' ? customStyle : []}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
      />

      <View style={styles.controls}>
        {['standard', 'satellite', 'terrain', 'hybrid'].map((type) => (
          <TouchableOpacity
            key={type}
            style={[styles.controlButton, mapType === type && styles.activeButton]}
            onPress={() => setMapType(type)}
          >
            <Text style={[styles.controlText, mapType === type && styles.activeText]}>
              {type.charAt(0).toUpperCase() + type.slice(1)}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  controls: {
    position: 'absolute',
    top: 50,
    left: 20,
    right: 20,
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  controlButton: {
    backgroundColor: 'rgba(255,255,255,0.9)',
    padding: 10,
    borderRadius: 5,
    minWidth: 70,
    alignItems: 'center',
  },
  activeButton: {
    backgroundColor: '#007AFF',
  },
  controlText: {
    fontSize: 12,
    fontWeight: 'bold',
  },
  activeText: {
    color: 'white',
  },
});

export default CustomMap;
```

---

## 📍 **Markers and Annotations**

### **Adding Markers**
```javascript
import React, { useState } from 'react';
import { View, StyleSheet } from 'react-native';
import MapView, { Marker, Callout } from 'react-native-maps';

const MapWithMarkers = () => {
  const [markers, setMarkers] = useState([
    {
      id: 1,
      coordinate: {
        latitude: 37.78825,
        longitude: -122.4324,
      },
      title: 'San Francisco',
      description: 'Beautiful city by the bay',
    },
    {
      id: 2,
      coordinate: {
        latitude: 37.7749,
        longitude: -122.4194,
      },
      title: 'Golden Gate Park',
      description: 'Large urban park',
    },
  ]);

  const addMarker = (coordinate) => {
    const newMarker = {
      id: Date.now(),
      coordinate,
      title: `Marker ${markers.length + 1}`,
      description: `Added at ${new Date().toLocaleTimeString()}`,
    };
    setMarkers([...markers, newMarker]);
  };

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
        onPress={(event) => addMarker(event.nativeEvent.coordinate)}
      >
        {markers.map((marker) => (
          <Marker
            key={marker.id}
            coordinate={marker.coordinate}
            title={marker.title}
            description={marker.description}
            pinColor={marker.id === 1 ? 'red' : 'blue'}
          >
            <Callout>
              <View style={styles.callout}>
                <Text style={styles.calloutTitle}>{marker.title}</Text>
                <Text style={styles.calloutDescription}>{marker.description}</Text>
              </View>
            </Callout>
          </Marker>
        ))}
      </MapView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  callout: {
    width: 200,
    padding: 10,
  },
  calloutTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  calloutDescription: {
    fontSize: 14,
    color: '#666',
  },
});

export default MapWithMarkers;
```

### **Custom Markers**
```javascript
import React from 'react';
import { View, Text, Image, StyleSheet } from 'react-native';
import MapView, { Marker } from 'react-native-maps';

const CustomMarkers = () => {
  const markers = [
    {
      id: 1,
      coordinate: { latitude: 37.78825, longitude: -122.4324 },
      title: 'Restaurant',
      type: 'restaurant',
    },
    {
      id: 2,
      coordinate: { latitude: 37.7749, longitude: -122.4194 },
      title: 'Hotel',
      type: 'hotel',
    },
    {
      id: 3,
      coordinate: { latitude: 37.7849, longitude: -122.4094 },
      title: 'Gas Station',
      type: 'gas',
    },
  ];

  const getMarkerIcon = (type) => {
    switch (type) {
      case 'restaurant':
        return require('./assets/restaurant-icon.png');
      case 'hotel':
        return require('./assets/hotel-icon.png');
      case 'gas':
        return require('./assets/gas-icon.png');
      default:
        return null;
    }
  };

  const CustomMarkerView = ({ marker }) => (
    <View style={styles.customMarker}>
      <Image source={getMarkerIcon(marker.type)} style={styles.markerIcon} />
      <Text style={styles.markerText}>{marker.title}</Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
      >
        {markers.map((marker) => (
          <Marker
            key={marker.id}
            coordinate={marker.coordinate}
          >
            <CustomMarkerView marker={marker} />
          </Marker>
        ))}
      </MapView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  customMarker: {
    backgroundColor: 'white',
    borderRadius: 20,
    padding: 5,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
    elevation: 5,
    alignItems: 'center',
  },
  markerIcon: {
    width: 30,
    height: 30,
  },
  markerText: {
    fontSize: 10,
    fontWeight: 'bold',
    marginTop: 2,
  },
});

export default CustomMarkers;
```

---

## 🖱️ **Map Interactions**

### **Map Events and Gestures**
```javascript
import React, { useRef, useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import MapView, { Marker } from 'react-native-maps';

const InteractiveMap = () => {
  const mapRef = useRef(null);
  const [region, setRegion] = useState({
    latitude: 37.78825,
    longitude: -122.4324,
    latitudeDelta: 0.0922,
    longitudeDelta: 0.0421,
  });
  const [markers, setMarkers] = useState([]);

  const onRegionChange = (newRegion) => {
    setRegion(newRegion);
  };

  const onMapPress = (event) => {
    const { coordinate } = event.nativeEvent;
    const newMarker = {
      id: Date.now(),
      coordinate,
      title: `Point ${markers.length + 1}`,
    };
    setMarkers([...markers, newMarker]);
  };

  const animateToLocation = (latitude, longitude) => {
    const newRegion = {
      latitude,
      longitude,
      latitudeDelta: 0.01,
      longitudeDelta: 0.01,
    };
    mapRef.current?.animateToRegion(newRegion, 1000);
  };

  const fitToMarkers = () => {
    if (markers.length > 0) {
      mapRef.current?.fitToCoordinates(
        markers.map(marker => marker.coordinate),
        {
          edgePadding: { top: 50, right: 50, bottom: 50, left: 50 },
          animated: true,
        }
      );
    }
  };

  return (
    <View style={styles.container}>
      <MapView
        ref={mapRef}
        style={styles.map}
        region={region}
        onRegionChange={onRegionChange}
        onPress={onMapPress}
        showsUserLocation={true}
        showsMyLocationButton={true}
      >
        {markers.map((marker) => (
          <Marker
            key={marker.id}
            coordinate={marker.coordinate}
            title={marker.title}
          />
        ))}
      </MapView>

      <View style={styles.controls}>
        <TouchableOpacity
          style={styles.controlButton}
          onPress={() => animateToLocation(37.7749, -122.4194)}
        >
          <Text style={styles.controlText}>SF</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={styles.controlButton}
          onPress={() => animateToLocation(34.0522, -118.2437)}
        >
          <Text style={styles.controlText}>LA</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={styles.controlButton}
          onPress={fitToMarkers}
          disabled={markers.length === 0}
        >
          <Text style={[styles.controlText, markers.length === 0 && styles.disabledText]}>
            Fit All
          </Text>
        </TouchableOpacity>
      </View>

      <View style={styles.info}>
        <Text>Latitude: {region.latitude.toFixed(4)}</Text>
        <Text>Longitude: {region.longitude.toFixed(4)}</Text>
        <Text>Zoom: {Math.round(1 / region.latitudeDelta * 1000)}x</Text>
        <Text>Markers: {markers.length}</Text>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  controls: {
    position: 'absolute',
    top: 50,
    left: 20,
    right: 20,
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  controlButton: {
    backgroundColor: 'rgba(255,255,255,0.9)',
    padding: 10,
    borderRadius: 5,
    minWidth: 50,
    alignItems: 'center',
  },
  controlText: {
    fontSize: 14,
    fontWeight: 'bold',
  },
  disabledText: {
    opacity: 0.5,
  },
  info: {
    position: 'absolute',
    bottom: 20,
    left: 20,
    right: 20,
    backgroundColor: 'rgba(255,255,255,0.9)',
    padding: 10,
    borderRadius: 5,
  },
});

export default InteractiveMap;
```

---

## 🛣️ **Routing and Directions**

### **Basic Routing**
```javascript
import React, { useState } from 'react';
import { View, StyleSheet } from 'react-native';
import MapView, { Marker, Polyline } from 'react-native-maps';
import MapViewDirections from 'react-native-maps-directions';

const GOOGLE_MAPS_APIKEY = 'YOUR_API_KEY';

const RouteMap = () => {
  const [coordinates] = useState([
    {
      latitude: 37.78825,
      longitude: -122.4324,
    },
    {
      latitude: 37.7749,
      longitude: -122.4194,
    },
  ]);

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
      >
        {coordinates.map((coord, index) => (
          <Marker
            key={index}
            coordinate={coord}
            title={index === 0 ? 'Start' : 'End'}
            pinColor={index === 0 ? 'green' : 'red'}
          />
        ))}

        <MapViewDirections
          origin={coordinates[0]}
          destination={coordinates[coordinates.length - 1]}
          apikey={GOOGLE_MAPS_APIKEY}
          strokeWidth={3}
          strokeColor="hotpink"
          onReady={(result) => {
            console.log('Distance:', result.distance);
            console.log('Duration:', result.duration);
          }}
          onError={(errorMessage) => {
            console.log('Directions error:', errorMessage);
          }}
        />
      </MapView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
});

export default RouteMap;
```

### **Multi-stop Route**
```javascript
import React, { useState } from 'react';
import { View, StyleSheet } from 'react-native';
import MapView, { Marker, Polyline } from 'react-native-maps';

const MultiStopRoute = () => {
  const [waypoints] = useState([
    { latitude: 37.78825, longitude: -122.4324 }, // Start
    { latitude: 37.7849, longitude: -122.4094 },  // Waypoint 1
    { latitude: 37.7749, longitude: -122.4194 },  // Waypoint 2
    { latitude: 37.7649, longitude: -122.4294 },  // End
  ]);

  // Simulated route coordinates (in real app, use routing API)
  const [routeCoordinates] = useState([
    { latitude: 37.78825, longitude: -122.4324 },
    { latitude: 37.7869, longitude: -122.4214 },
    { latitude: 37.7849, longitude: -122.4094 },
    { latitude: 37.7799, longitude: -122.4144 },
    { latitude: 37.7749, longitude: -122.4194 },
    { latitude: 37.7699, longitude: -122.4244 },
    { latitude: 37.7649, longitude: -122.4294 },
  ]);

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: 37.7749,
          longitude: -122.4194,
          latitudeDelta: 0.1,
          longitudeDelta: 0.1,
        }}
      >
        {waypoints.map((point, index) => (
          <Marker
            key={index}
            coordinate={point}
            title={`Stop ${index + 1}`}
            description={index === 0 ? 'Starting Point' : index === waypoints.length - 1 ? 'Destination' : 'Waypoint'}
            pinColor={
              index === 0 ? 'green' :
              index === waypoints.length - 1 ? 'red' : 'blue'
            }
          />
        ))}

        <Polyline
          coordinates={routeCoordinates}
          strokeColor="#007AFF"
          strokeWidth={4}
          lineDashPattern={[10, 5]}
        />
      </MapView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
});

export default MultiStopRoute;
```

---

## 🏘️ **Geofencing**

### **Geofence Implementation**
```javascript
import React, { useEffect, useState } from 'react';
import { View, Text, StyleSheet, Alert } from 'react-native';
import Geolocation from 'react-native-geolocation-service';
import MapView, { Circle, Marker } from 'react-native-maps';

const GeofenceExample = () => {
  const [currentLocation, setCurrentLocation] = useState(null);
  const [geofences] = useState([
    {
      id: 1,
      center: { latitude: 37.78825, longitude: -122.4324 },
      radius: 500, // 500 meters
      name: 'Home',
    },
    {
      id: 2,
      center: { latitude: 37.7749, longitude: -122.4194 },
      radius: 300,
      name: 'Work',
    },
  ]);

  const [activeGeofence, setActiveGeofence] = useState(null);

  useEffect(() => {
    const watchId = Geolocation.watchPosition(
      (position) => {
        const location = position.coords;
        setCurrentLocation(location);
        checkGeofences(location);
      },
      (error) => console.error(error),
      { enableHighAccuracy: true, distanceFilter: 10 }
    );

    return () => Geolocation.clearWatch(watchId);
  }, []);

  const checkGeofences = (location) => {
    for (const geofence of geofences) {
      const distance = getDistance(
        location.latitude,
        location.longitude,
        geofence.center.latitude,
        geofence.center.longitude
      );

      if (distance <= geofence.radius) {
        if (activeGeofence !== geofence.id) {
          setActiveGeofence(geofence.id);
          Alert.alert(
            'Geofence Alert',
            `You entered ${geofence.name}`,
            [{ text: 'OK' }]
          );
        }
        break;
      } else if (activeGeofence === geofence.id) {
        setActiveGeofence(null);
        Alert.alert(
          'Geofence Alert',
          `You left ${geofence.name}`,
          [{ text: 'OK' }]
        );
      }
    }
  };

  const getDistance = (lat1, lon1, lat2, lon2) => {
    const R = 6371e3; // Earth's radius in meters
    const φ1 = (lat1 * Math.PI) / 180;
    const φ2 = (lat2 * Math.PI) / 180;
    const Δφ = ((lat2 - lat1) * Math.PI) / 180;
    const Δλ = ((lon2 - lon1) * Math.PI) / 180;

    const a =
      Math.sin(Δφ / 2) * Math.sin(Δφ / 2) +
      Math.cos(φ1) * Math.cos(φ2) * Math.sin(Δλ / 2) * Math.sin(Δλ / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

    return R * c;
  };

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
        showsUserLocation={true}
      >
        {geofences.map((geofence) => (
          <React.Fragment key={geofence.id}>
            <Circle
              center={geofence.center}
              radius={geofence.radius}
              strokeColor={activeGeofence === geofence.id ? '#FF0000' : '#007AFF'}
              fillColor={
                activeGeofence === geofence.id
                  ? 'rgba(255,0,0,0.2)'
                  : 'rgba(0,122,255,0.2)'
              }
              strokeWidth={2}
            />
            <Marker
              coordinate={geofence.center}
              title={geofence.name}
              description={`Radius: ${geofence.radius}m`}
            />
          </React.Fragment>
        ))}
      </MapView>

      <View style={styles.info}>
        <Text>Active Geofence: {activeGeofence ? geofences.find(g => g.id === activeGeofence)?.name : 'None'}</Text>
        {currentLocation && (
          <Text>
            Location: {currentLocation.latitude.toFixed(4)}, {currentLocation.longitude.toFixed(4)}
          </Text>
        )}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  info: {
    position: 'absolute',
    bottom: 20,
    left: 20,
    right: 20,
    backgroundColor: 'rgba(255,255,255,0.9)',
    padding: 10,
    borderRadius: 5,
  },
});

export default GeofenceExample;
```

---

## 📱 **Offline Maps**

### **Map Caching**
```javascript
import React, { useRef } from 'react';
import { View, TouchableOpacity, Text, StyleSheet } from 'react-native';
import MapView from 'react-native-maps';

const OfflineMap = () => {
  const mapRef = useRef(null);

  const cacheCurrentRegion = async () => {
    if (mapRef.current) {
      try {
        // Get current visible region
        const region = await mapRef.current.getMapBoundaries();

        // Cache tiles for offline use
        // Note: This is a simplified example
        // In production, use a proper offline map solution
        console.log('Caching region:', region);
        alert('Map region cached for offline use');
      } catch (error) {
        console.error('Cache error:', error);
      }
    }
  };

  const clearCache = () => {
    // Clear cached tiles
    console.log('Clearing map cache');
    alert('Map cache cleared');
  };

  return (
    <View style={styles.container}>
      <MapView
        ref={mapRef}
        style={styles.map}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
        mapType="satellite"
        showsUserLocation={true}
      />

      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={cacheCurrentRegion}>
          <Text style={styles.buttonText}>Cache Region</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.button, styles.clearButton]} onPress={clearCache}>
          <Text style={styles.buttonText}>Clear Cache</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  controls: {
    position: 'absolute',
    bottom: 20,
    left: 20,
    right: 20,
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    flex: 1,
    marginHorizontal: 5,
    alignItems: 'center',
  },
  clearButton: {
    backgroundColor: '#FF3B30',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default OfflineMap;
```

---

## 🎯 **Practical Examples**

### **Ride-Sharing App**
```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import MapView, { Marker, Polyline } from 'react-native-maps';
import Geolocation from 'react-native-geolocation-service';

const RideSharingApp = () => {
  const [userLocation, setUserLocation] = useState(null);
  const [driverLocation, setDriverLocation] = useState({
    latitude: 37.7749,
    longitude: -122.4194,
  });
  const [route, setRoute] = useState([]);
  const [rideStatus, setRideStatus] = useState('idle'); // idle, requested, accepted, en_route, arrived

  useEffect(() => {
    // Get user location
    Geolocation.getCurrentPosition(
      (position) => {
        setUserLocation(position.coords);
      },
      (error) => console.error(error),
      { enableHighAccuracy: true }
    );

    // Simulate driver movement
    if (rideStatus === 'en_route') {
      const interval = setInterval(() => {
        setDriverLocation(prev => ({
          ...prev,
          latitude: prev.latitude + 0.001,
          longitude: prev.longitude + 0.001,
        }));
      }, 2000);

      return () => clearInterval(interval);
    }
  }, [rideStatus]);

  const requestRide = () => {
    setRideStatus('requested');
    // Simulate ride acceptance
    setTimeout(() => {
      setRideStatus('accepted');
      setTimeout(() => {
        setRideStatus('en_route');
      }, 2000);
    }, 3000);
  };

  const getStatusColor = () => {
    switch (rideStatus) {
      case 'idle': return '#666';
      case 'requested': return '#FFA500';
      case 'accepted': return '#007AFF';
      case 'en_route': return '#28a745';
      case 'arrived': return '#6f42c1';
      default: return '#666';
    }
  };

  const getStatusText = () => {
    switch (rideStatus) {
      case 'idle': return 'Ready to ride';
      case 'requested': return 'Finding driver...';
      case 'accepted': return 'Driver found!';
      case 'en_route': return 'Driver en route';
      case 'arrived': return 'Driver arrived';
      default: return 'Ready to ride';
    }
  };

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
        showsUserLocation={true}
      >
        {userLocation && (
          <Marker
            coordinate={userLocation}
            title="Your Location"
            pinColor="blue"
          />
        )}

        <Marker
          coordinate={driverLocation}
          title="Driver"
          description={getStatusText()}
          pinColor="green"
        />

        {route.length > 0 && (
          <Polyline
            coordinates={route}
            strokeColor="#007AFF"
            strokeWidth={4}
          />
        )}
      </MapView>

      <View style={[styles.statusBar, { backgroundColor: getStatusColor() }]}>
        <Text style={styles.statusText}>{getStatusText()}</Text>
        {rideStatus === 'idle' && (
          <TouchableOpacity style={styles.rideButton} onPress={requestRide}>
            <Text style={styles.rideButtonText}>Request Ride</Text>
          </TouchableOpacity>
        )}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  statusBar: {
    padding: 20,
    alignItems: 'center',
  },
  statusText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  rideButton: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    minWidth: 150,
    alignItems: 'center',
  },
  rideButtonText: {
    color: '#007AFF',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default RideSharingApp;
```

### **Location-Based Social App**
```javascript
import React, { useState, useEffect } from 'react';
import { View, FlatList, Text, StyleSheet } from 'react-native';
import MapView, { Marker, Callout } from 'react-native-maps';
import Geolocation from 'react-native-geolocation-service';

const SocialApp = () => {
  const [userLocation, setUserLocation] = useState(null);
  const [nearbyUsers, setNearbyUsers] = useState([
    {
      id: 1,
      name: 'Alice',
      coordinate: { latitude: 37.78825, longitude: -122.4324 },
      status: 'Available',
      distance: 0.5,
    },
    {
      id: 2,
      name: 'Bob',
      coordinate: { latitude: 37.7849, longitude: -122.4094 },
      status: 'Busy',
      distance: 1.2,
    },
    {
      id: 3,
      name: 'Charlie',
      coordinate: { latitude: 37.7749, longitude: -122.4194 },
      status: 'Available',
      distance: 2.1,
    },
  ]);

  useEffect(() => {
    Geolocation.getCurrentPosition(
      (position) => {
        setUserLocation(position.coords);
      },
      (error) => console.error(error),
      { enableHighAccuracy: true }
    );
  }, []);

  const renderNearbyUser = ({ item }) => (
    <View style={styles.userItem}>
      <Text style={styles.userName}>{item.name}</Text>
      <Text style={styles.userStatus}>{item.status}</Text>
      <Text style={styles.userDistance}>{item.distance} km away</Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={{
          latitude: 37.78825,
          longitude: -122.4324,
          latitudeDelta: 0.0922,
          longitudeDelta: 0.0421,
        }}
        showsUserLocation={true}
      >
        {nearbyUsers.map((user) => (
          <Marker
            key={user.id}
            coordinate={user.coordinate}
            pinColor={user.status === 'Available' ? 'green' : 'red'}
          >
            <Callout>
              <View style={styles.callout}>
                <Text style={styles.calloutName}>{user.name}</Text>
                <Text style={styles.calloutStatus}>{user.status}</Text>
                <Text style={styles.calloutDistance}>{user.distance} km away</Text>
              </View>
            </Callout>
          </Marker>
        ))}
      </MapView>

      <View style={styles.userList}>
        <Text style={styles.listTitle}>Nearby Users</Text>
        <FlatList
          data={nearbyUsers}
          renderItem={renderNearbyUser}
          keyExtractor={(item) => item.id.toString()}
          horizontal
          showsHorizontalScrollIndicator={false}
        />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    flex: 1,
  },
  userList: {
    position: 'absolute',
    bottom: 20,
    left: 20,
    right: 20,
    backgroundColor: 'rgba(255,255,255,0.9)',
    borderRadius: 10,
    padding: 15,
  },
  listTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  userItem: {
    backgroundColor: '#f0f0f0',
    padding: 10,
    borderRadius: 8,
    marginRight: 10,
    minWidth: 120,
  },
  userName: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  userStatus: {
    fontSize: 14,
    color: '#666',
  },
  userDistance: {
    fontSize: 12,
    color: '#999',
  },
  callout: {
    width: 150,
    padding: 10,
  },
  calloutName: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  calloutStatus: {
    fontSize: 14,
    color: '#666',
  },
  calloutDistance: {
    fontSize: 12,
    color: '#999',
  },
});

export default SocialApp;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Location Permissions**: Requesting and handling location access
- ✅ **Geolocation API**: Getting current location and watching position changes
- ✅ **Map Integration**: Displaying interactive maps with react-native-maps
- ✅ **Markers & Annotations**: Adding markers, custom markers, and callouts
- ✅ **Map Interactions**: Handling map events, gestures, and animations
- ✅ **Routing & Directions**: Implementing turn-by-turn directions
- ✅ **Geofencing**: Creating location-based boundaries and alerts
- ✅ **Offline Maps**: Caching map data for offline use

### **Best Practices**
1. **Always request appropriate permissions** before accessing location
2. **Handle location errors gracefully** with user-friendly messages
3. **Optimize location updates** using distanceFilter and interval settings
4. **Cache map data** for better offline experience
5. **Use appropriate map providers** (Google Maps, Apple Maps, etc.)
6. **Consider battery impact** of location services
7. **Test on real devices** for accurate GPS behavior

### **Next Steps**
- Experiment with different map providers and styles
- Implement advanced routing algorithms
- Add real-time location sharing features
- Integrate with location-based APIs

---

## 🎯 **Assignment**

### **Task 1: Location Tracker App**
Create a location tracking app with:
- Real-time location updates
- Route recording and display
- Distance and speed calculations
- Location history with timestamps
- Export route data as GPX/KML

### **Task 2: Nearby Places Finder**
Build an app that:
- Shows nearby restaurants, hotels, gas stations
- Displays place details and ratings
- Provides directions to selected places
- Filters results by distance and category
- Shows place photos and reviews

### **Task 3: Geofence-Based Reminder**
Implement a reminder app that:
- Allows creating location-based reminders
- Triggers notifications when entering/leaving geofences
- Shows reminder details on map
- Supports multiple geofences per reminder
- Handles background location updates

---

## 📚 **Additional Resources**
- [React Native Maps Documentation](https://github.com/react-native-maps/react-native-maps)
- [React Native Geolocation Service](https://github.com/Agontuk/react-native-geolocation-service)
- [Google Maps Platform](https://developers.google.com/maps)
- [Mapbox Documentation](https://docs.mapbox.com/)
- [Expo Location Documentation](https://docs.expo.dev/versions/latest/sdk/location/)

---

**Next Lesson**: [Lesson 17: Push Notifications & Local Notifications](Lesson%2017_%20Push%20Notifications%20&%20Local%20Notifications.md)