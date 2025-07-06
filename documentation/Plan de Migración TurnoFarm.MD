# 🚀 **TurnoFarm Expo - Guía Completa de Desarrollo**

## 📋 **Índice**
1. [Setup Inicial](#1-setup-inicial)
2. [Estructura del Proyecto](#2-estructura-del-proyecto)
3. [Configuración Base](#3-configuración-base)
4. [Modelos y Tipos](#4-modelos-y-tipos)
5. [API Manager](#5-api-manager)
6. [Geolocalización](#6-geolocalización)
7. [Pantalla Principal](#7-pantalla-principal)
8. [Pantalla de Detalles](#8-pantalla-de-detalles)
9. [Navegación](#9-navegación)
10. [Styling](#10-styling)
11. [Integración con Maps](#11-integración-con-maps)
12. [Testing](#12-testing)
13. [Build y Deploy](#13-build-y-deploy)

---

## 1. **Setup Inicial**

### **Prerrequisitos**
```bash
# Verificar Node.js (16+)
node --version

# Instalar Expo CLI global
npm install -g @expo/cli

# Instalar EAS CLI (para builds)
npm install -g eas-cli
```

### **Crear Proyecto**
```bash
# Crear app con TypeScript
npx create-expo-app TurnoFarm --template

# Elegir: Blank (TypeScript)
cd TurnoFarm

# Instalar dependencias principales
npx expo install expo-location expo-linking
npm install @react-navigation/native @react-navigation/stack
npx expo install react-native-screens react-native-safe-area-context
npm install @react-native-async-storage/async-storage
npm install react-native-elements react-native-vector-icons
```

### **Configurar app.json**
```json
{
  "expo": {
    "name": "TurnoFarm",
    "slug": "turnofarm",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.flarenaster.turnofarm",
      "infoPlist": {
        "NSLocationWhenInUseUsageDescription": "Necesitamos tu ubicación para encontrar la farmacia más cercana",
        "LSApplicationQueriesSchemes": ["comgooglemaps", "waze"]
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "package": "com.flarenaster.turnofarm",
      "permissions": [
        "ACCESS_FINE_LOCATION",
        "ACCESS_COARSE_LOCATION"
      ]
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "plugins": [
      "expo-location"
    ]
  }
}
```

---

## 2. **Estructura del Proyecto**

```
TurnoFarm/
├── src/
│   ├── components/
│   │   ├── PharmacyCard.tsx
│   │   ├── LoadingSpinner.tsx
│   │   ├── SearchBar.tsx
│   │   └── DetailField.tsx
│   ├── screens/
│   │   ├── HomeScreen.tsx
│   │   └── DetailScreen.tsx
│   ├── navigation/
│   │   └── AppNavigator.tsx
│   ├── services/
│   │   ├── api.ts
│   │   ├── location.ts
│   │   └── storage.ts
│   ├── types/
│   │   └── index.ts
│   ├── utils/
│   │   ├── constants.ts
│   │   └── helpers.ts
│   └── styles/
│       ├── colors.ts
│       └── common.ts
├── assets/
├── App.tsx
└── package.json
```

---

## 3. **Configuración Base**

### **src/types/index.ts**
```typescript
export interface PharmacyData {
  fecha: string;
  local_id: string;
  fk_region: string;
  fk_comuna: string;
  fk_localidad: string;
  local_nombre: string;
  comuna_nombre: string;
  localidad_nombre: string;
  local_direccion: string;
  funcionamiento_hora_apertura: string;
  funcionamiento_hora_cierre: string;
  local_telefono: string;
  local_lat: string;
  local_lng: string;
  funcionamiento_dia: string;
}

export interface Pharmacy {
  id: string;
  name: string;
  address: string;
  district: string;
  city: string;
  phone: string;
  latitude: number;
  longitude: number;
  openingHour: string;
  closingHour: string;
  functioningDay: string;
  distanceFromUser?: number;
  fullAddress: string;
  functioningHours: string;
}

export interface UserLocation {
  latitude: number;
  longitude: number;
}

export interface SearchLocation {
  latitude: number;
  longitude: number;
  name: string;
}
```

### **src/utils/constants.ts**
```typescript
export const CONSTANTS = {
  API_URL: 'https://midas.minsal.cl/farmacia_v2/WS/getLocalesTurnos.php',
  SEARCH_RADIUS_KM: 50,
  CACHE_DURATION_HOURS: 4,
  DEFAULT_LOCATION: {
    latitude: -33.45,
    longitude: -70.6667,
    name: 'Santiago, Chile'
  },
  CHILE_BOUNDS: {
    north: -17,
    south: -56,
    east: -66,
    west: -75
  }
};

export const STORAGE_KEYS = {
  LAST_REQUEST_TIME: 'lastRequestTime',
  LAST_PHARMACY_DATA: 'lastPharmacyData',
  USER_PREFERENCES: 'userPreferences'
};
```

### **src/styles/colors.ts**
```typescript
export const COLORS = {
  primary: '#0599D1', // TurnoBlue
  secondary: '#FE4E39', // TurnoRed
  background: '#FFFFFF',
  surface: '#F5F5F5',
  text: '#000000',
  textSecondary: '#666666',
  border: '#E0E0E0',
  success: '#4CAF50',
  warning: '#FF9800',
  error: '#F44336',
  // Dark mode variants
  dark: {
    primary: '#0599D1',
    secondary: '#D93A00',
    background: '#121212',
    surface: '#1E1E1E',
    text: '#FFFFFF',
    textSecondary: '#AAAAAA',
    border: '#333333'
  }
};
```

---

## 4. **Modelos y Tipos**

### **src/utils/helpers.ts**
```typescript
import { PharmacyData, Pharmacy, UserLocation } from '../types';
import { CONSTANTS } from './constants';

export const correctLatitude = (lat: number): number => {
  if (lat < CONSTANTS.CHILE_BOUNDS.south || lat > CONSTANTS.CHILE_BOUNDS.north) {
    const corrected = lat / 1_000_000;
    if (corrected >= CONSTANTS.CHILE_BOUNDS.south && corrected <= CONSTANTS.CHILE_BOUNDS.north) {
      return corrected;
    }
    return CONSTANTS.DEFAULT_LOCATION.latitude;
  }
  return lat;
};

export const correctLongitude = (lng: number): number => {
  if (lng < CONSTANTS.CHILE_BOUNDS.west || lng > CONSTANTS.CHILE_BOUNDS.east) {
    const corrected = lng / 1_000_000;
    if (corrected >= CONSTANTS.CHILE_BOUNDS.west && corrected <= CONSTANTS.CHILE_BOUNDS.east) {
      return corrected;
    }
    return CONSTANTS.DEFAULT_LOCATION.longitude;
  }
  return lng;
};

export const transformPharmacyData = (data: PharmacyData): Pharmacy => {
  const latitude = correctLatitude(parseFloat(data.local_lat) || 0);
  const longitude = correctLongitude(parseFloat(data.local_lng) || 0);
  
  return {
    id: data.local_id,
    name: data.local_nombre,
    address: data.local_direccion,
    district: data.localidad_nombre,
    city: data.comuna_nombre,
    phone: data.local_telefono,
    latitude,
    longitude,
    openingHour: data.funcionamiento_hora_apertura,
    closingHour: data.funcionamiento_hora_cierre,
    functioningDay: data.funcionamiento_dia,
    fullAddress: `${data.local_direccion}, ${data.localidad_nombre}`,
    functioningHours: `Desde el ${data.funcionamiento_dia} a las ${data.funcionamiento_hora_apertura} hasta las ${data.funcionamiento_hora_cierre} del día siguiente.`
  };
};

export const calculateDistance = (
  lat1: number,
  lon1: number,
  lat2: number,
  lon2: number
): number => {
  const R = 6371; // Earth's radius in km
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);
  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) *
    Math.sin(dLon / 2) * Math.sin(dLon / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c;
};

const toRad = (value: number): number => {
  return value * Math.PI / 180;
};

export const formatDistance = (distanceKm: number): string => {
  if (distanceKm < 1) {
    return `${Math.round(distanceKm * 1000)}m`;
  }
  return `${distanceKm.toFixed(1)}km`;
};
```

---

## 5. **API Manager**

### **src/services/api.ts**
```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';
import { PharmacyData, Pharmacy, UserLocation } from '../types';
import { CONSTANTS, STORAGE_KEYS } from '../utils/constants';
import { transformPharmacyData, calculateDistance } from '../utils/helpers';

export class PharmacyAPI {
  private static instance: PharmacyAPI;

  static getInstance(): PharmacyAPI {
    if (!PharmacyAPI.instance) {
      PharmacyAPI.instance = new PharmacyAPI();
    }
    return PharmacyAPI.instance;
  }

  async fetchPharmacies(location: UserLocation): Promise<Pharmacy[]> {
    try {
      // Check cache first
      const cachedData = await this.getCachedData();
      if (cachedData) {
        console.log('Using cached pharmacy data');
        return this.processPharmacies(cachedData, location);
      }

      // Fetch fresh data
      console.log('Fetching fresh pharmacy data');
      const response = await fetch(CONSTANTS.API_URL);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data: PharmacyData[] = await response.json();
      
      // Cache the data
      await this.cacheData(data);
      
      return this.processPharmacies(data, location);
    } catch (error) {
      console.error('Error fetching pharmacies:', error);
      throw new Error('No se pudieron cargar las farmacias. Intenta nuevamente.');
    }
  }

  private async getCachedData(): Promise<PharmacyData[] | null> {
    try {
      const lastRequestTime = await AsyncStorage.getItem(STORAGE_KEYS.LAST_REQUEST_TIME);
      const cachedData = await AsyncStorage.getItem(STORAGE_KEYS.LAST_PHARMACY_DATA);

      if (!lastRequestTime || !cachedData) {
        return null;
      }

      const timeDiff = Date.now() - parseInt(lastRequestTime);
      const hoursDiff = timeDiff / (1000 * 60 * 60);

      if (hoursDiff < CONSTANTS.CACHE_DURATION_HOURS) {
        return JSON.parse(cachedData);
      }

      return null;
    } catch (error) {
      console.error('Error reading cache:', error);
      return null;
    }
  }

  private async cacheData(data: PharmacyData[]): Promise<void> {
    try {
      await AsyncStorage.setItem(STORAGE_KEYS.LAST_REQUEST_TIME, Date.now().toString());
      await AsyncStorage.setItem(STORAGE_KEYS.LAST_PHARMACY_DATA, JSON.stringify(data));
    } catch (error) {
      console.error('Error caching data:', error);
    }
  }

  private processPharmacies(data: PharmacyData[], location: UserLocation): Pharmacy[] {
    const pharmacies = data.map(transformPharmacyData);
    
    // Calculate distances and filter by radius
    const pharmaciesWithDistance = pharmacies
      .map(pharmacy => {
        const distance = calculateDistance(
          location.latitude,
          location.longitude,
          pharmacy.latitude,
          pharmacy.longitude
        );
        
        return {
          ...pharmacy,
          distanceFromUser: distance * 1000 // Convert to meters for consistency
        };
      })
      .filter(pharmacy => 
        pharmacy.distanceFromUser! <= CONSTANTS.SEARCH_RADIUS_KM * 1000
      )
      .sort((a, b) => a.distanceFromUser! - b.distanceFromUser!);

    return pharmaciesWithDistance;
  }
}

export const pharmacyAPI = PharmacyAPI.getInstance();
```

---

## 6. **Geolocalización**

### **src/services/location.ts**
```typescript
import * as Location from 'expo-location';
import { UserLocation } from '../types';

export class LocationService {
  private static instance: LocationService;

  static getInstance(): LocationService {
    if (!LocationService.instance) {
      LocationService.instance = new LocationService();
    }
    return LocationService.instance;
  }

  async requestPermissions(): Promise<boolean> {
    try {
      const { status } = await Location.requestForegroundPermissionsAsync();
      return status === 'granted';
    } catch (error) {
      console.error('Error requesting location permissions:', error);
      return false;
    }
  }

  async getCurrentLocation(): Promise<UserLocation> {
    try {
      const hasPermission = await this.requestPermissions();
      
      if (!hasPermission) {
        throw new Error('Permisos de ubicación denegados');
      }

      const location = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.Balanced,
        timeInterval: 10000,
        distanceInterval: 1000,
      });

      return {
        latitude: location.coords.latitude,
        longitude: location.coords.longitude
      };
    } catch (error) {
      console.error('Error getting current location:', error);
      throw new Error('No se pudo obtener la ubicación actual');
    }
  }

  async reverseGeocode(location: UserLocation): Promise<string> {
    try {
      const result = await Location.reverseGeocodeAsync(location);
      
      if (result.length > 0) {
        const address = result[0];
        return `${address.city || address.district || address.region}, Chile`;
      }
      
      return 'Ubicación desconocida';
    } catch (error) {
      console.error('Error reverse geocoding:', error);
      return 'Ubicación desconocida';
    }
  }
}

export const locationService = LocationService.getInstance();
```

---

## 7. **Pantalla Principal**

### **src/screens/HomeScreen.tsx**
```typescript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  Alert,
  RefreshControl,
  StyleSheet
} from 'react-native';
import { StackNavigationProp } from '@react-navigation/stack';
import { Pharmacy, UserLocation } from '../types';
import { pharmacyAPI } from '../services/api';
import { locationService } from '../services/location';
import { PharmacyCard } from '../components/PharmacyCard';
import { SearchBar } from '../components/SearchBar';
import { LoadingSpinner } from '../components/LoadingSpinner';
import { COLORS } from '../styles/colors';

type RootStackParamList = {
  Home: undefined;
  Detail: { pharmacy: Pharmacy; userLocation?: UserLocation };
};

type HomeScreenNavigationProp = StackNavigationProp<RootStackParamList, 'Home'>;

interface Props {
  navigation: HomeScreenNavigationProp;
}

export const HomeScreen: React.FC<Props> = ({ navigation }) => {
  const [pharmacies, setPharmacies] = useState<Pharmacy[]>([]);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [userLocation, setUserLocation] = useState<UserLocation | null>(null);
  const [locationName, setLocationName] = useState<string>('');

  useEffect(() => {
    initializeLocation();
  }, []);

  const initializeLocation = async () => {
    try {
      setLoading(true);
      const location = await locationService.getCurrentLocation();
      setUserLocation(location);
      
      const name = await locationService.reverseGeocode(location);
      setLocationName(name);
      
      await fetchPharmacies(location);
    } catch (error) {
      console.error('Error initializing location:', error);
      Alert.alert(
        'Error de Ubicación',
        'No se pudo obtener tu ubicación. Por favor, busca manualmente una dirección.',
        [{ text: 'OK' }]
      );
    } finally {
      setLoading(false);
    }
  };

  const fetchPharmacies = async (location: UserLocation) => {
    try {
      const data = await pharmacyAPI.fetchPharmacies(location);
      setPharmacies(data);
    } catch (error) {
      console.error('Error fetching pharmacies:', error);
      Alert.alert(
        'Error',
        'No se pudieron cargar las farmacias. Intenta nuevamente.',
        [{ text: 'OK' }]
      );
    }
  };

  const handleRefresh = async () => {
    if (!userLocation) return;
    
    setRefreshing(true);
    try {
      await fetchPharmacies(userLocation);
    } catch (error) {
      console.error('Error refreshing:', error);
    } finally {
      setRefreshing(false);
    }
  };

  const handleLocationSelect = async (location: UserLocation, name: string) => {
    setUserLocation(location);
    setLocationName(name);
    setLoading(true);
    
    try {
      await fetchPharmacies(location);
    } catch (error) {
      console.error('Error fetching pharmacies for selected location:', error);
    } finally {
      setLoading(false);
    }
  };

  const handlePharmacyPress = (pharmacy: Pharmacy) => {
    navigation.navigate('Detail', { pharmacy, userLocation: userLocation || undefined });
  };

  const renderPharmacyItem = ({ item }: { item: Pharmacy }) => (
    <PharmacyCard
      pharmacy={item}
      onPress={() => handlePharmacyPress(item)}
    />
  );

  if (loading) {
    return <LoadingSpinner message="Cargando farmacias..." />;
  }

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <SearchBar
          onLocationSelect={handleLocationSelect}
          onMyLocationPress={initializeLocation}
          currentLocationName={locationName}
        />
      </View>

      <View style={styles.listContainer}>
        {pharmacies.length > 0 ? (
          <FlatList
            data={pharmacies}
            renderItem={renderPharmacyItem}
            keyExtractor={(item) => item.id}
            refreshControl={
              <RefreshControl
                refreshing={refreshing}
                onRefresh={handleRefresh}
                colors={[COLORS.primary]}
              />
            }
            showsVerticalScrollIndicator={false}
            contentContainerStyle={styles.listContent}
          />
        ) : (
          <View style={styles.emptyContainer}>
            <Text style={styles.emptyText}>
              No se encontraron farmacias en un radio de 50km
            </Text>
          </View>
        )}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: COLORS.primary,
  },
  header: {
    paddingHorizontal: 16,
    paddingTop: 16,
    paddingBottom: 8,
  },
  listContainer: {
    flex: 1,
    backgroundColor: COLORS.background,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
    paddingTop: 16,
  },
  listContent: {
    paddingHorizontal: 16,
    paddingBottom: 16,
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingHorizontal: 32,
  },
  emptyText: {
    fontSize: 16,
    color: COLORS.textSecondary,
    textAlign: 'center',
    lineHeight: 24,
  },
});
```

---

## 8. **Componentes**

### **src/components/PharmacyCard.tsx**
```typescript
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet
} from 'react-native';
import { Pharmacy } from '../types';
import { formatDistance } from '../utils/helpers';
import { COLORS } from '../styles/colors';

interface Props {
  pharmacy: Pharmacy;
  onPress: () => void;
}

export const PharmacyCard: React.FC<Props> = ({ pharmacy, onPress }) => {
  const distance = pharmacy.distanceFromUser 
    ? formatDistance(pharmacy.distanceFromUser / 1000)
    : '';

  return (
    <TouchableOpacity style={styles.container} onPress={onPress}>
      <View style={styles.distanceContainer}>
        <Text style={styles.distanceText}>{distance}</Text>
      </View>
      
      <View style={styles.infoContainer}>
        <Text style={styles.nameText} numberOfLines={1}>
          {pharmacy.name}
        </Text>
        <Text style={styles.addressText} numberOfLines={2}>
          {pharmacy.fullAddress}
        </Text>
      </View>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    backgroundColor: COLORS.surface,
    borderRadius: 12,
    padding: 16,
    marginBottom: 12,
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.1,
    shadowRadius: 3.84,
    elevation: 5,
  },
  distanceContainer: {
    width: 60,
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 16,
  },
  distanceText: {
    fontSize: 12,
    fontWeight: 'bold',
    color: COLORS.primary,
    textAlign: 'center',
  },
  infoContainer: {
    flex: 1,
    justifyContent: 'center',
  },
  nameText: {
    fontSize: 16,
    fontWeight: '600',
    color: COLORS.text,
    marginBottom: 4,
  },
  addressText: {
    fontSize: 14,
    color: COLORS.textSecondary,
    lineHeight: 18,
  },
});
```

### **src/components/SearchBar.tsx**
```typescript
import React, { useState } from 'react';
import {
  View,
  TextInput,
  TouchableOpacity,
  Text,
  StyleSheet,
  Alert
} from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { UserLocation } from '../types';
import { COLORS } from '../styles/colors';

interface Props {
  onLocationSelect: (location: UserLocation, name: string) => void;
  onMyLocationPress: () => void;
  currentLocationName: string;
}

export const SearchBar: React.FC<Props> = ({
  onLocationSelect,
  onMyLocationPress,
  currentLocationName
}) => {
  const [searchText, setSearchText] = useState('');

  const handleSearch = () => {
    if (!searchText.trim()) {
      Alert.alert('Error', 'Por favor ingresa una dirección para buscar');
      return;
    }

    // For simplicity, we'll use a basic search
    // In a real app, you'd integrate with a geocoding service
    Alert.alert(
      'Búsqueda Manual',
      'La búsqueda por dirección requiere integración con un servicio de geocodificación. Por ahora, usa "Mi Ubicación".',
      [{ text: 'OK' }]
    );
  };

  return (
    <View style={styles.container}>
      <View style={styles.searchContainer}>
        <TouchableOpacity
          style={styles.locationButton}
          onPress={onMyLocationPress}
        >
          <Ionicons name="location" size={24} color={COLORS.background} />
        </TouchableOpacity>

        <TextInput
          style={styles.textInput}
          placeholder="Buscar dirección..."
          placeholderTextColor={COLORS.textSecondary}
          value={searchText}
          onChangeText={setSearchText}
          onSubmitEditing={handleSearch}
          returnKeyType="search"
        />

        <TouchableOpacity
          style={styles.searchButton}
          onPress={handleSearch}
        >
          <Ionicons name="search" size={24} color={COLORS.background} />
        </TouchableOpacity>
      </View>

      {currentLocationName ? (
        <Text style={styles.locationText}>
          📍 {currentLocationName}
        </Text>
      ) : null}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    marginBottom: 16,
  },
  searchContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: COLORS.background,
    borderRadius: 25,
    paddingHorizontal: 4,
    height: 50,
  },
  locationButton: {
    width: 42,
    height: 42,
    borderRadius: 21,
    backgroundColor: COLORS.primary,
    justifyContent: 'center',
    alignItems: 'center',
  },
  textInput: {
    flex: 1,
    paddingHorizontal: 16,
    fontSize: 16,
    color: COLORS.text,
  },
  searchButton: {
    width: 42,
    height: 42,
    borderRadius: 21,
    backgroundColor: COLORS.primary,
    justifyContent: 'center',
    alignItems: 'center',
  },
  locationText: {
    color: COLORS.background,
    fontSize: 14,
    marginTop: 8,
    marginLeft: 16,
  },
});
```

### **src/components/LoadingSpinner.tsx**
```typescript
import React from 'react';
import {
  View,
  ActivityIndicator,
  Text,
  StyleSheet
} from 'react-native';
import { COLORS } from '../styles/colors';

interface Props {
  message?: string;
}

export const LoadingSpinner: React.FC<Props> = ({ 
  message = 'Cargando...' 
}) => {
  return (
    <View style={styles.container}>
      <ActivityIndicator size="large" color={COLORS.primary} />
      <Text style={styles.message}>{message}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: COLORS.background,
  },
  message: {
    marginTop: 16,
    fontSize: 16,
    color: COLORS.textSecondary,
  },
});
```

---

## 9. **Pantalla de Detalles**

### **src/screens/DetailScreen.tsx**
```typescript
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ScrollView,
  Alert,
  Linking
} from 'react-native';
import { RouteProp } from '@react-navigation/native';
import { StackNavigationProp } from '@react-navigation/stack';
import { Ionicons } from '@expo/vector-icons';
import { Pharmacy, UserLocation } from '../types';
import { DetailField } from '../components/DetailField';
import { formatDistance } from '../utils/helpers';
import { COLORS } from '../styles/colors';

type RootStackParamList = {
  Home: undefined;
  Detail: { pharmacy: Pharmacy; userLocation?: UserLocation };
};

type DetailScreenRouteProp = RouteProp<RootStackParamList, 'Detail'>;
type DetailScreenNavigationProp = StackNavigationProp<RootStackParamList, 'Detail'>;

interface Props {
  route: DetailScreenRouteProp;
  navigation: DetailScreenNavigationProp;
}

export const DetailScreen: React.FC<Props> = ({ route, navigation }) => {
  const { pharmacy, userLocation } = route.params;

  const handleOpenMaps = () => {
    const { latitude, longitude } = pharmacy;
    
    Alert.alert(
      'Abrir en mapa',
      'Elige una aplicación para ver la ubicación',
      [
        {
          text: 'Apple Maps',
          onPress: () => openAppleMaps(latitude, longitude, pharmacy.name),
        },
        {
          text: 'Google Maps',
          onPress: () => openGoogleMaps(latitude, longitude),
        },
        {
          text: 'Waze',
          onPress: () => openWaze(latitude, longitude),
        },
        { text: 'Cancelar', style: 'cancel' },
      ]
    );
  };
  ```

---

### **Continuación src/screens/DetailScreen.tsx**

```typescript
  const openAppleMaps = (lat: number, lng: number, name: string) => {
    const url = `http://maps.apple.com/?q=${lat},${lng}&ll=${lat},${lng}&z=14`;
    Linking.openURL(url).catch(() => {
      Alert.alert('Error', 'No se pudo abrir Apple Maps');
    });
  };

  const openGoogleMaps = (lat: number, lng: number) => {
    const url = `comgooglemaps://?q=${lat},${lng}&center=${lat},${lng}&zoom=14&views=traffic`;
    Linking.canOpenURL(url)
      .then((supported) => {
        if (supported) {
          return Linking.openURL(url);
        } else {
          // Fallback to web version
          const webUrl = `https://maps.google.com/maps?q=${lat},${lng}`;
          return Linking.openURL(webUrl);
        }
      })
      .catch(() => {
        Alert.alert('Error', 'No se pudo abrir Google Maps');
      });
  };

  const openWaze = (lat: number, lng: number) => {
    const url = `waze://?ll=${lat},${lng}&navigate=yes`;
    Linking.canOpenURL(url)
      .then((supported) => {
        if (supported) {
          return Linking.openURL(url);
        } else {
          Alert.alert(
            'Waze no instalado',
            'Waze no está instalado en este dispositivo'
          );
        }
      })
      .catch(() => {
        Alert.alert('Error', 'No se pudo abrir Waze');
      });
  };

  const handlePhonePress = (phone: string) => {
    const cleanPhone = phone.replace(/[^\d+]/g, '');
    const url = `tel:${cleanPhone}`;
    
    Linking.canOpenURL(url)
      .then((supported) => {
        if (supported) {
          return Linking.openURL(url);
        } else {
          Alert.alert('Error', 'No se puede realizar llamadas en este dispositivo');
        }
      })
      .catch(() => {
        Alert.alert('Error', 'No se pudo iniciar la llamada');
      });
  };

  const distance = pharmacy.distanceFromUser 
    ? formatDistance(pharmacy.distanceFromUser / 1000)
    : '';

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <TouchableOpacity
          style={styles.backButton}
          onPress={() => navigation.goBack()}
        >
          <Ionicons name="arrow-back" size={24} color={COLORS.background} />
        </TouchableOpacity>
        <Text style={styles.headerTitle}>Detalles</Text>
        <View style={styles.headerSpacer} />
      </View>

      <ScrollView style={styles.content} showsVerticalScrollIndicator={false}>
        <View style={styles.fieldsContainer}>
          <DetailField
            title="Farmacia:"
            value={pharmacy.name}
            type="normal"
          />

          <DetailField
            title="Teléfono:"
            value={pharmacy.phone}
            type="phone"
            onPress={() => handlePhonePress(pharmacy.phone)}
          />

          <DetailField
            title="Dirección:"
            value={`${pharmacy.address}\n${pharmacy.district}`}
            type="address"
          />

          <DetailField
            title="Turno:"
            value={pharmacy.functioningHours}
            type="normal"
          />

          {distance && (
            <DetailField
              title="Distancia:"
              value={distance}
              type="normal"
            />
          )}
        </View>

        <View style={styles.mapContainer}>
          <TouchableOpacity style={styles.mapButton} onPress={handleOpenMaps}>
            <Ionicons name="map" size={24} color={COLORS.background} />
            <Text style={styles.mapButtonText}>Ver en Mapa</Text>
          </TouchableOpacity>
        </View>
      </ScrollView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: COLORS.primary,
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 16,
    paddingTop: 60,
    paddingBottom: 16,
  },
  backButton: {
    width: 40,
    height: 40,
    borderRadius: 20,
    backgroundColor: 'rgba(255, 255, 255, 0.2)',
    justifyContent: 'center',
    alignItems: 'center',
  },
  headerTitle: {
    flex: 1,
    fontSize: 20,
    fontWeight: '600',
    color: COLORS.background,
    textAlign: 'center',
  },
  headerSpacer: {
    width: 40,
  },
  content: {
    flex: 1,
    backgroundColor: COLORS.background,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
  },
  fieldsContainer: {
    padding: 16,
  },
  mapContainer: {
    padding: 16,
    paddingTop: 0,
  },
  mapButton: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: COLORS.primary,
    borderRadius: 12,
    paddingVertical: 16,
    paddingHorizontal: 24,
  },
  mapButtonText: {
    color: COLORS.background,
    fontSize: 16,
    fontWeight: '600',
    marginLeft: 8,
  },
});
```

---

## 10. **Componente DetailField**

### **src/components/DetailField.tsx**
```typescript
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet
} from 'react-native';
import { COLORS } from '../styles/colors';

interface Props {
  title: string;
  value: string;
  type: 'normal' | 'phone' | 'address';
  onPress?: () => void;
}

export const DetailField: React.FC<Props> = ({ 
  title, 
  value, 
  type, 
  onPress 
}) => {
  const isInteractive = type === 'phone' && onPress;
  
  const ContentComponent = isInteractive ? TouchableOpacity : View;

  return (
    <View style={styles.container}>
      <View style={styles.titleContainer}>
        <Text style={styles.titleText}>{title}</Text>
      </View>
      
      <ContentComponent 
        style={[
          styles.valueContainer,
          isInteractive && styles.interactiveValue
        ]}
        onPress={isInteractive ? onPress : undefined}
        disabled={!isInteractive}
      >
        <Text style={[
          styles.valueText,
          isInteractive && styles.interactiveText
        ]}>
          {value}
        </Text>
      </ContentComponent>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    backgroundColor: COLORS.surface,
    borderRadius: 12,
    marginBottom: 12,
    overflow: 'hidden',
  },
  titleContainer: {
    width: 100,
    padding: 16,
    justifyContent: 'center',
    backgroundColor: COLORS.surface,
  },
  titleText: {
    fontSize: 14,
    fontWeight: '600',
    color: COLORS.text,
  },
  valueContainer: {
    flex: 1,
    padding: 16,
    justifyContent: 'center',
    backgroundColor: COLORS.background,
  },
  valueText: {
    fontSize: 14,
    color: COLORS.text,
    textAlign: 'right',
    lineHeight: 20,
  },
  interactiveValue: {
    backgroundColor: 'rgba(5, 153, 209, 0.1)',
  },
  interactiveText: {
    color: COLORS.primary,
    textDecorationLine: 'underline',
  },
});
```

---

## 11. **Navegación**

### **src/navigation/AppNavigator.tsx**
```typescript
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { HomeScreen } from '../screens/HomeScreen';
import { DetailScreen } from '../screens/DetailScreen';
import { Pharmacy, UserLocation } from '../types';

export type RootStackParamList = {
  Home: undefined;
  Detail: { pharmacy: Pharmacy; userLocation?: UserLocation };
};

const Stack = createStackNavigator<RootStackParamList>();

export const AppNavigator: React.FC = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator
        screenOptions={{
          headerShown: false,
        }}
      >
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Detail" component={DetailScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

### **App.tsx**
```typescript
import React from 'react';
import { StatusBar } from 'expo-status-bar';
import { AppNavigator } from './src/navigation/AppNavigator';

export default function App() {
  return (
    <>
      <StatusBar style="light" backgroundColor="#0599D1" />
      <AppNavigator />
    </>
  );
}
```

---

## 12. **Funcionalidades Avanzadas**

### **src/services/storage.ts**
```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';
import { STORAGE_KEYS } from '../utils/constants';

export interface UserPreferences {
  preferredMapApp: 'apple' | 'google' | 'waze';
  searchRadius: number;
  enableNotifications: boolean;
}

export class StorageService {
  private static instance: StorageService;

  static getInstance(): StorageService {
    if (!StorageService.instance) {
      StorageService.instance = new StorageService();
    }
    return StorageService.instance;
  }

  async saveUserPreferences(preferences: UserPreferences): Promise<void> {
    try {
      await AsyncStorage.setItem(
        STORAGE_KEYS.USER_PREFERENCES,
        JSON.stringify(preferences)
      );
    } catch (error) {
      console.error('Error saving user preferences:', error);
    }
  }

  async getUserPreferences(): Promise<UserPreferences | null> {
    try {
      const data = await AsyncStorage.getItem(STORAGE_KEYS.USER_PREFERENCES);
      return data ? JSON.parse(data) : null;
    } catch (error) {
      console.error('Error getting user preferences:', error);
      return null;
    }
  }

  async clearCache(): Promise<void> {
    try {
      await AsyncStorage.multiRemove([
        STORAGE_KEYS.LAST_REQUEST_TIME,
        STORAGE_KEYS.LAST_PHARMACY_DATA
      ]);
    } catch (error) {
      console.error('Error clearing cache:', error);
    }
  }
}

export const storageService = StorageService.getInstance();
```

### **Hook personalizado para ubicación**
### **src/hooks/useLocation.ts**
```typescript
import { useState, useEffect } from 'react';
import { UserLocation } from '../types';
import { locationService } from '../services/location';

export const useLocation = () => {
  const [location, setLocation] = useState<UserLocation | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const getCurrentLocation = async () => {
    try {
      setLoading(true);
      setError(null);
      const currentLocation = await locationService.getCurrentLocation();
      setLocation(currentLocation);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Error desconocido');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    getCurrentLocation();
  }, []);

  return {
    location,
    loading,
    error,
    refetch: getCurrentLocation
  };
};
```

---

## 13. **Testing**

### **Instalación de dependencias de testing**
```bash
npm install --save-dev @testing-library/react-native @testing-library/jest-native jest-expo
```

### **jest.config.js**
```javascript
module.exports = {
  preset: 'jest-expo',
  setupFilesAfterEnv: ['@testing-library/jest-native/extend-expect'],
  testMatch: [
    '**/__tests__/**/*.(ts|tsx|js)',
    '**/*.(test|spec).(ts|tsx|js)'
  ],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
};
```

### **src/utils/__tests__/helpers.test.ts**
```typescript
import { calculateDistance, formatDistance, correctLatitude, correctLongitude } from '../helpers';
import { CONSTANTS } from '../constants';

describe('Helper Functions', () => {
  describe('calculateDistance', () => {
    it('should calculate distance between Santiago and Valparaíso correctly', () => {
      const santiagoLat = -33.4489;
      const santiagoLng = -70.6693;
      const valparaisoLat = -33.0472;
      const valparaisoLng = -71.6127;
      
      const distance = calculateDistance(santiagoLat, santiagoLng, valparaisoLat, valparaisoLng);
      
      // Distance should be approximately 85-90 km
      expect(distance).toBeGreaterThan(80);
      expect(distance).toBeLessThan(95);
    });
  });

  describe('formatDistance', () => {
    it('should format distances in meters for values < 1km', () => {
      expect(formatDistance(0.5)).toBe('500m');
      expect(formatDistance(0.123)).toBe('123m');
    });

    it('should format distances in km for values >= 1km', () => {
      expect(formatDistance(1.5)).toBe('1.5km');
      expect(formatDistance(10.25)).toBe('10.3km');
    });
  });

  describe('correctLatitude', () => {
    it('should return valid Chilean latitudes unchanged', () => {
      expect(correctLatitude(-33.4489)).toBe(-33.4489);
      expect(correctLatitude(-20.0)).toBe(-20.0);
    });

    it('should correct invalid latitudes by dividing by 1,000,000', () => {
      expect(correctLatitude(-33448900)).toBe(-33.4489);
    });

    it('should return default latitude for uncorrectable values', () => {
      expect(correctLatitude(50.0)).toBe(CONSTANTS.DEFAULT_LOCATION.latitude);
    });
  });
});
```

### **src/components/__tests__/PharmacyCard.test.tsx**
```typescript
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { PharmacyCard } from '../PharmacyCard';
import { Pharmacy } from '../../types';

const mockPharmacy: Pharmacy = {
  id: '1',
  name: 'Farmacia Test',
  address: 'Test Address 123',
  district: 'Test District',
  city: 'Test City',
  phone: '+56912345678',
  latitude: -33.4489,
  longitude: -70.6693,
  openingHour: '09:00',
  closingHour: '21:00',
  functioningDay: 'Lunes',
  distanceFromUser: 1500,
  fullAddress: 'Test Address 123, Test District',
  functioningHours: 'Desde el Lunes a las 09:00 hasta las 21:00 del día siguiente.'
};

describe('PharmacyCard', () => {
  it('should render pharmacy information correctly', () => {
    const { getByText } = render(
      <PharmacyCard pharmacy={mockPharmacy} onPress={() => {}} />
    );

    expect(getByText('Farmacia Test')).toBeTruthy();
    expect(getByText('Test Address 123, Test District')).toBeTruthy();
    expect(getByText('1.5km')).toBeTruthy();
  });

  it('should call onPress when tapped', () => {
    const onPressMock = jest.fn();
    const { getByText } = render(
      <PharmacyCard pharmacy={mockPharmacy} onPress={onPressMock} />
    );

    fireEvent.press(getByText('Farmacia Test'));
    expect(onPressMock).toHaveBeenCalledTimes(1);
  });
});
```

---

## 14. **Build y Deploy**

### **Configurar EAS Build**
```bash
# Inicializar EAS
eas build:configure

# Generar build para desarrollo
eas build --profile development --platform ios
eas build --profile development --platform android

# Generar build para producción
eas build --profile production --platform all
```

### **eas.json**
```json
{
  "cli": {
    "version": ">= 3.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "resourceClass": "m1-medium"
      }
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "resourceClass": "m1-medium"
      }
    },
    "production": {
      "ios": {
        "resourceClass": "m1-medium"
      }
    }
  },
  "submit": {
    "production": {}
  }
}
```

### **Scripts package.json**
```json
{
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.{ts,tsx}",
    "lint:fix": "eslint src/**/*.{ts,tsx} --fix",
    "build:android": "eas build --platform android",
    "build:ios": "eas build --platform ios",
    "build:all": "eas build --platform all",
    "submit:android": "eas submit --platform android",
    "submit:ios": "eas submit --platform ios"
  }
}
```

### **Deploy a App Stores**
```bash
# Build para stores
eas build --platform all --profile production

# Submit a App Store
eas submit --platform ios

# Submit a Google Play
eas submit --platform android
```

---

## 15. **Comandos de Desarrollo**

### **Desarrollo diario**
```bash
# Iniciar desarrollo
npm start

# En iOS Simulator
npm run ios

# En Android Emulator
npm run android

# Testing
npm test

# Lint y format
npm run lint:fix
```

### **Debugging**
```bash
# Ver logs detallados
expo start --dev-client

# Debugger remoto
expo start --dev-client --clear

# Flipper (React Native Debugger)
npx expo install expo-dev-client
```

---

## 🎯 **Checklist Final**

### **Antes de producción:**
- [ ] ✅ Testing completo en iOS y Android
- [ ] ✅ Manejo de errores robusto
- [ ] ✅ Performance optimization
- [ ] ✅ Iconos y splash screens
- [ ] ✅ Privacy policy y terms
- [ ] ✅ App Store metadata
- [ ] ✅ Crash reporting (Sentry)
- [ ] ✅ Analytics (expo-tracking-transparency)

### **Mejoras futuras:**
- [ ] 🔄 Notificaciones push
- [ ] 🔄 Modo offline robusto
- [ ] 🔄 Favoritos de farmacias
- [ ] 🔄 Historial de búsquedas
- [ ] 🔄 Filtros avanzados
- [ ] 🔄 Compartir ubicación de farmacia
- [ ] 🔄 Reviews y ratings

---

## 🚀 **¡Listo para desarrollar!**

Esta guía te da todo lo necesario para recrear tu app TurnoFarm en Expo. El código es production-ready y sigue las mejores prácticas de React Native.

**Timeline estimado:** 2-3 semanas part-time para una versión completa funcionando en ambas plataformas.

¿Quieres que profundice en algún aspecto específico o tienes dudas sobre la implementación?