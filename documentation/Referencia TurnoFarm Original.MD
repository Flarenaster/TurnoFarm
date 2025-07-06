# 📋 **TurnoFarm - Documentación Completa del Proyecto**

## 🎯 **Descripción General**
Aplicación iOS nativa (Swift) para encontrar farmacias de turno en Chile. Permite localizar al usuario y mostrar farmacias cercanas ordenadas por distancia, con detalles completos e integración con apps de mapas.

---

## 🏗️ **Arquitectura del Proyecto**

### **Estructura de Archivos**
```
TurnoFarm/
├── Model/
│   ├── Pharmacy.swift          # Modelo principal de farmacia
│   ├── PharmacyData.swift      # Modelo para API response
│   └── FinderManager.swift     # Manager para API calls
├── View/
│   ├── Main.storyboard         # UI principal
│   ├── LaunchScreen.storyboard # Splash screen
│   ├── PharmacyCell.swift/.xib # Celda custom para lista
│   └── DetailFieldView/        # Componentes de detalle
│       ├── DetailFieldView.swift/.xib
│       ├── DetailFieldStrategy.swift
│       ├── NormalFieldStrategy.swift
│       ├── PhoneFieldStrategy.swift
│       └── AddressFieldStrategy.swift
├── Controller/
│   ├── FinderViewController.swift  # Pantalla principal
│   └── DetailViewController.swift  # Pantalla de detalles
├── Assets.xcassets/            # Imágenes y colores
├── Constants.swift             # Constantes de la app
├── AppDelegate.swift           # Entry point
├── SceneDelegate.swift         # Scene lifecycle
└── Info.plist                 # Configuraciones
```

---

## 📱 **Funcionalidades Principales**

### **1. Pantalla Principal (FinderViewController)**

#### **Geolocalización**
- **Permiso de ubicación**: Solicita automáticamente al abrir
- **Mi Ubicación**: Botón para re-localizar usuario
- **Manejo de errores**: Si falla GPS, permite búsqueda manual

#### **Búsqueda de Ubicaciones**
- **Autocompletado**: Usa `MKLocalSearchCompleter`
- **Filtro Chile**: Solo muestra resultados de Chile
- **Búsqueda manual**: TextField con resultados desplegables
- **Región específica**: Configurado para territorio chileno

#### **Lista de Farmacias**
- **Ordenamiento**: Por distancia (más cercana primero)
- **Información mostrada**:
  - Distancia en kilómetros
  - Nombre de la farmacia
  - Dirección completa (dirección + localidad)
- **Celda personalizada**: `PharmacyCell` con layout específico

#### **Radio de Búsqueda**
- **Límite**: 50km desde ubicación
- **Filtrado automático**: Solo muestra farmacias dentro del radio

### **2. Pantalla de Detalles (DetailViewController)**

#### **Información Completa**
- **Nombre**: Nombre de la farmacia
- **Teléfono**: Con detector de datos (clickeable)
- **Dirección**: Con detector de direcciones (clickeable)
- **Horarios**: Texto descriptivo del turno

#### **Mapa Integrado**
- **MapKit**: Mapa nativo de Apple
- **Ubicación farmacia**: Pin en ubicación exacta
- **Ruta**: Si hay ubicación del usuario, muestra ruta en auto
- **Tiempo estimado**: Calcula y muestra tiempo de viaje
- **Callout**: Información emergente en el pin

#### **Integración con Apps Externas**
Al tocar el mapa, ActionSheet con opciones:
- **Apple Maps**: Abre ubicación en Maps nativo
- **Google Maps**: Abre en Google Maps (si está instalado)
- **Waze**: Abre en Waze (si está instalado)
- **Manejo de errores**: Avisa si app no está instalada

### **3. Componentes Personalizados**

#### **DetailFieldView (Patrón Strategy)**
- **NormalFieldStrategy**: Campo de solo lectura
- **PhoneFieldStrategy**: Detecta teléfonos, permite llamar
- **AddressFieldStrategy**: Detecta direcciones, abre en mapas
- **Configuración dinámica**: Cada campo se configura según su tipo

---

## 🌐 **Integración con API**

### **Endpoint**
```
URL: https://midas.minsal.cl/farmacia_v2/WS/getLocalesTurnos.php
Método: GET
```

### **Rate Limiting Inteligente**
- **Cache local**: 4 horas entre requests
- **UserDefaults**: Guarda última respuesta y timestamp
- **Reutilización**: Si hay datos recientes, los usa sin hacer request

### **Modelo de Datos**
```swift
struct PharmacyData: Codable {
    let fecha: String
    let local_id: String
    let local_nombre: String
    let comuna_nombre: String
    let localidad_nombre: String
    let local_direccion: String
    let funcionamiento_hora_apertura: String
    let funcionamiento_hora_cierre: String
    let local_telefono: String
    let local_lat: String
    let local_lng: String
    let funcionamiento_dia: String
}

struct Pharmacy: Codable {
    let id: String
    let name: String
    let address: String
    let district: String
    let city: String
    let phone: String
    let latitude: Double
    let longitude: Double
    let openingHour: String
    let closingHour: String
    let functioningDay: String
    var distanceFromUser: Double?
    var fullAddress: String
}
```

### **Corrección de Coordenadas**
- **Validación**: Verifica coordenadas están dentro de Chile
- **Normalización**: Divide por 1,000,000 si están en formato incorrecto
- **Fallback**: Santiago como coordenadas por defecto (-33.45, -70.6667)

---

## 🎨 **Diseño y UI**

### **Colores Personalizados**
- **TurnoBlue**: Color principal de la app
- **TurnoRed**: Color de acento
- **Soporte Dark Mode**: Variantes para modo oscuro

### **Layout**
- **Responsive**: Funciona en iPhone y iPad
- **Auto Layout**: Constraints para diferentes tamaños
- **Stack Views**: Organización limpia de componentes

### **Iconografía**
- **SF Symbols**: Iconos nativos de Apple
- **location.circle.fill**: Botón de ubicación
- **magnifyingglass**: Botón de búsqueda

---

## 🔧 **Configuraciones Técnicas**

### **Permisos (Info.plist)**
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>Necesitamos tu ubicación para encontrar la farmacia más cercana</string>

<key>LSApplicationQueriesSchemes</key>
<array>
    <string>comgooglemaps</string>
    <string>waze</string>
</array>
```

### **Deployment Target**
- **iOS 13.0+**: Versión mínima soportada
- **Swift 5.0**: Lenguaje de programación
- **Xcode 15.3**: Versión de desarrollo

### **Bundle Identifier**
`com.flarenaster.TurnoFarm`

---

## 🔄 **Flujo de Usuario**

### **Flujo Principal**
1. **App Launch** → Solicita permisos de ubicación
2. **Geolocalización** → Obtiene coordenadas del usuario
3. **API Call** → Consulta farmacias (con rate limiting)
4. **Filtrado** → Muestra solo farmacias dentro de 50km
5. **Lista** → Ordena por distancia y muestra
6. **Selección** → Usuario toca farmacia
7. **Detalles** → Muestra información completa + mapa
8. **Navegación** → Usuario puede abrir en apps externas

### **Flujo de Búsqueda Manual**
1. **Usuario escribe** → Autocompletado se activa
2. **Filtro Chile** → Solo resultados chilenos
3. **Selección lugar** → Usa coordenadas del lugar
4. **API Call** → Busca farmacias cerca del lugar
5. **Lista actualizada** → Muestra farmacias cercanas al lugar

---

## 🚀 **Features Técnicos Avanzados**

### **Manejo de Estados**
- **Delegate Pattern**: FinderManagerDelegate para callbacks
- **Error Handling**: Manejo robusto de errores de red/GPS
- **Loading States**: Indicadores de carga apropiados

### **Performance**
- **Lazy Loading**: Celdas reutilizables en tabla
- **Memory Management**: Weak references para evitar retain cycles
- **Efficient Rendering**: Solo renderiza celdas visibles

### **Experiencia de Usuario**
- **Smooth Animations**: Transiciones nativas
- **Haptic Feedback**: (Implementable)
- **Accessibility**: Labels apropiados para VoiceOver
- **Offline Tolerance**: Cache de última búsqueda

---

## 🎯 **Casos de Uso Específicos**

### **Usuario en Santiago**
1. Abre app → GPS automático → Ve farmacias cercanas
2. Selecciona una → Ve detalles + tiempo de viaje
3. Toca mapa → Abre en Waze para navegar

### **Usuario viajando**
1. Abre app → No reconoce ubicación
2. Busca "Valparaíso" → Selecciona de autocompletado
3. Ve farmacias de Valparaíso → Selecciona una
4. Llama directamente desde el teléfono mostrado

### **Usuario sin GPS**
1. App falla GPS → Permite búsqueda manual
2. Escribe ubicación → Encuentra farmacias
3. Funcionalidad completa sin ubicación propia

---

## 📊 **Métricas y Limitaciones**

### **Limitaciones Actuales**
- **iOS Only**: No hay versión Android
- **Chile Only**: API y filtros específicos para Chile
- **Rate Limit**: 1 request cada 4 horas
- **Radius**: Máximo 50km de búsqueda

### **Performance Targets**
- **App Launch**: < 3 segundos hasta lista
- **GPS Fix**: < 10 segundos en condiciones normales
- **API Response**: Típicamente < 5 segundos
- **Map Rendering**: Instantáneo con datos cached

---

Esta documentación cubre todas las funcionalidades, arquitectura y detalles técnicos del proyecto original TurnoFarm en Swift/iOS.