# Weather Forecast App

## Overview
Create a modern Android weather application using **Jetpack Compose**, **location services**, and **REST APIs**. This exercise tests your ability to integrate multiple APIs, handle location permissions, work with date selection, and create intuitive user interfaces.

## Duration
**Estimated Time: 3 hours**

## Objective
Build a single-screen Android application that:
1. Detects the user's current location using GPS/Network
2. Allows users to select a future date using a date picker
3. Performs reverse geocoding to get location details
4. Fetches weather forecast data from OpenWeatherMap API
5. Displays whether it will rain on the selected date
6. Demonstrates modern Android architecture and API integration

---

## Core Features

### 1. Location Detection
- **Automatic Location**: Request location permissions and get current GPS coordinates
- **Location Display**: Show the detected city/address to the user
- **Error Handling**: Handle permission denied, location disabled, or detection failures
- **Manual Refresh**: Allow users to refresh/update their location

### 2. Date Selection
- **Future Dates Only**: Date picker that only allows selection of future dates (today + next 7 days)
- **Intuitive UI**: Material Design 3 date picker component
- **Date Validation**: Prevent selection of past dates or dates beyond forecast range
- **Selected Date Display**: Clear indication of the currently selected date

### 3. Weather Forecast
- **Rain Prediction**: Clear yes/no answer about rain probability
- **Weather Details**: Temperature, humidity, weather description
- **Visual Indicators**: Icons or graphics to represent weather conditions
- **Data Freshness**: Show when the forecast data was last updated

---

## Technical Requirements

### Core Technologies
- **Language**: Kotlin
- **UI Framework**: Jetpack Compose
- **Architecture**: MVVM with Repository pattern
- **Location Services**: FusedLocationProviderClient
- **Networking**: Retrofit with Gson/Moshi
- **Permissions**: Location permissions handling
- **Minimum SDK**: API 24 (Android 7.0)
- **Target SDK**: API 34 (Android 14)

---

## API Integration

### 1. Reverse Geocoding API
**Endpoint**: `https://nominatim.openstreetmap.org/reverse`

**Required Parameters**:
```
lat={latitude}
lon={longitude}
format=json
addressdetails=1
```

**Example Request**:
```
https://nominatim.openstreetmap.org/reverse?format=json&lat=52.5487429714954&lon=-1.81602098644987&addressdetails=1
```

**Example Response**:
```json
{
  "place_id": "1620612",
  "licence": "Data © OpenStreetMap contributors, ODbL 1.0.",
  "osm_type": "node",
  "osm_id": "452010817",
  "lat": "52.5487473",
  "lon": "-1.8160821",
  "display_name": "135, Pilkington Avenue, Wylde Green, City of Birmingham, West Midlands, B72, United Kingdom",
  "address": {
    "house_number": "135",
    "road": "Pilkington Avenue",
    "village": "Wylde Green",
    "town": "Sutton Coldfield",
    "city": "City of Birmingham",
    "county": "West Midlands",
    "postcode": "B72",
    "country": "United Kingdom",
    "country_code": "gb"
  }
}
```

**Implementation Notes**:
- Use the `address.city`, `address.town`, or `address.village` for location display
- Handle cases where address components might be missing
- Add proper error handling for network failures

### 2. OpenWeatherMap Forecast API
**Endpoint**: `https://api.openweathermap.org/data/2.5/forecast`

**Required Parameters**:
```
lat={latitude}
lon={longitude}
appid={API_key}
units=metric
```

**API Key**: You'll need to sign up at [OpenWeatherMap](https://openweathermap.org/api) for a free API key

**Example Request**:
```
https://api.openweathermap.org/data/2.5/forecast?lat=52.5487&lon=-1.8161&appid=YOUR_API_KEY&units=metric
```

**Example Response Structure**:
```json
{
  "cod": "200",
  "message": 0,
  "cnt": 40,
  "list": [
    {
      "dt": 1578326400,
      "main": {
        "temp": 7.57,
        "feels_like": 3.38,
        "temp_min": 6.84,
        "temp_max": 7.57,
        "pressure": 1019,
        "sea_level": 1019,
        "grnd_level": 1015,
        "humidity": 81
      },
      "weather": [
        {
          "id": 804,
          "main": "Clouds",
          "description": "overcast clouds",
          "icon": "04d"
        }
      ],
      "clouds": {
        "all": 100
      },
      "wind": {
        "speed": 5.19,
        "deg": 211
      },
      "pop": 0.32,
      "sys": {
        "pod": "d"
      },
      "dt_txt": "2020-01-06 15:00:00"
    }
  ],
  "city": {
    "id": 2643743,
    "name": "London",
    "coord": {
      "lat": 51.5085,
      "lon": -0.1257
    },
    "country": "GB"
  }
}
```

**Key Fields for Rain Detection**:
- `pop` (Probability of Precipitation): 0.0 to 1.0 (0% to 100%)
- `weather[0].main`: "Rain", "Snow", "Drizzle" indicate precipitation
- `rain.3h`: Amount of rain in last 3 hours (if present)

**Rain Logic**:
```kotlin
fun willItRain(forecastItem: ForecastItem): Boolean {
    return forecastItem.pop > 0.3 || // 30%+ probability
           forecastItem.weather.any { it.main in listOf("Rain", "Drizzle", "Thunderstorm") }
}
```

---

## Implementation Guide

### 1. Project Structure
```
app/
├── src/main/java/com/yourcompany/weatherapp/
│   ├── data/
│   │   ├── model/           # API response models
│   │   ├── remote/          # Retrofit services
│   │   ├── repository/      # Repository implementation
│   │   └── location/        # Location services
│   ├── domain/
│   │   ├── model/           # Domain models
│   │   ├── repository/      # Repository interface
│   │   └── usecase/         # Business logic use cases
│   ├── ui/
│   │   ├── theme/           # Material Design 3 theme
│   │   ├── components/      # Reusable UI components
│   │   ├── screen/          # WeatherScreen
│   │   └── viewmodel/       # WeatherViewModel
│   └── MainActivity.kt
```

### 2. Data Models
```kotlin
// Domain Models
data class Location(
    val latitude: Double,
    val longitude: Double,
    val address: String,
    val city: String,
    val country: String
)

data class WeatherForecast(
    val date: LocalDate,
    val temperature: Double,
    val description: String,
    val willRain: Boolean,
    val rainProbability: Double,
    val humidity: Int,
    val icon: String
)

data class WeatherState(
    val isLoadingLocation: Boolean = false,
    val isLoadingWeather: Boolean = false,
    val location: Location? = null,
    val selectedDate: LocalDate = LocalDate.now().plusDays(1),
    val forecast: WeatherForecast? = null,
    val error: String? = null
)
```

### 3. Location Service
```kotlin
class LocationService @Inject constructor(
    private val fusedLocationClient: FusedLocationProviderClient,
    private val geocodingApi: GeocodingApi
) {
    
    @RequiresPermission(Manifest.permission.ACCESS_FINE_LOCATION)
    suspend fun getCurrentLocation(): Location {
        // Get GPS coordinates
        // Call reverse geocoding API
        // Return Location object
    }
}
```

### 4. Weather Repository
```kotlin
interface WeatherRepository {
    suspend fun getCurrentLocation(): Result<Location>
    suspend fun getWeatherForecast(
        latitude: Double, 
        longitude: Double, 
        date: LocalDate
    ): Result<WeatherForecast>
}
```

---

## User Interface Requirements

### Screen Layout
1. **Header Section**:
   - App title "Weather Forecast"
   - Location display with refresh button
   - Current location loading indicator

2. **Date Selection**:
   - Date picker button showing selected date
   - Clear indication that only future dates are allowed
   - Material Design 3 date picker dialog

3. **Weather Display**:
   - Large, clear "Will it rain?" answer (YES/NO)
   - Weather icon and description
   - Temperature and additional details
   - Rain probability percentage
   - Loading states and error handling

4. **Footer**:
   - Last updated timestamp
   - Data source attribution

### UI States to Handle
- **Loading location**: Show spinner with "Detecting location..."
- **Loading weather**: Show spinner with "Fetching forecast..."
- **Location permission denied**: Show explanation and settings button
- **No network**: Show retry button and offline message
- **API errors**: Show user-friendly error messages
- **Success**: Show complete weather information

---

## Permissions

### Required Permissions
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

### Runtime Permission Handling
- Request location permissions at startup
- Show rationale dialog explaining why location is needed
- Handle permission denied gracefully
- Provide option to manually enter location (bonus feature)

---

## Error Handling

### Location Errors
- **Permission denied**: Show explanation and request again
- **Location disabled**: Guide user to enable location services
- **Location timeout**: Show retry option
- **Geocoding failure**: Fall back to coordinates display

### Weather API Errors
- **Invalid API key**: Show setup instructions
- **Network timeout**: Show retry button
- **Rate limit exceeded**: Show appropriate message
- **Invalid coordinates**: Validate input data

### User Experience
- **Offline mode**: Cache last successful forecast
- **Graceful degradation**: Show partial data if available
- **Clear error messages**: Avoid technical jargon
- **Easy recovery**: Always provide way to retry or refresh

---

## Evaluation Criteria

### Technical Implementation (40%)
- **API Integration**: Correct implementation of both APIs with proper error handling
- **Architecture**: Clean MVVM implementation with proper separation of concerns
- **Location Services**: Proper permission handling and location detection
- **Code Quality**: Readable, maintainable Kotlin code with appropriate patterns

### UI/UX Design (30%)
- **Material Design 3**: Correct implementation of Material theming and components
- **User Experience**: Intuitive flow and clear information hierarchy
- **Loading States**: Proper handling of async operations and user feedback
- **Error Handling**: Graceful error states with recovery options

### Modern Android Practices (20%)
- **Jetpack Compose**: Effective use of modern declarative UI
- **State Management**: Proper state hoisting and lifecycle awareness
- **Permission Handling**: Correct runtime permission implementation
- **Network Security**: Proper API key handling and network configuration

### Feature Completeness (10%)
- **Core Functionality**: All required features working as expected
- **Edge Cases**: Proper handling of unusual scenarios
- **Data Validation**: Input validation and boundary checking
- **Performance**: Responsive UI and efficient API usage

---

## Deliverables

### 1. Complete Android Project
- Fully functional weather application
- All source code with proper git history
- Working APK file

### 2. Documentation
- **README.md** with:
  - Setup and run instructions
  - Architecture overview
  - Known limitations or assumptions

### 3. Demo Video
  - 30 seconds screen recording showing:
  - Location detection and permission flow
  - Date selection functionality
  - Weather forecast display for different dates
  - Error handling (if possible to demonstrate)

---

## API Setup Instructions

### OpenWeatherMap API Key
1. Sign up at [OpenWeatherMap](https://openweathermap.org/api)
2. Navigate to API Keys section
3. Generate a new API key (free tier allows 1000 calls/day)
4. Add the key to your app (preferably in BuildConfig or local.properties)

### Rate Limiting
- OpenWeatherMap free tier: 60 calls/minute, 1000 calls/day
- Nominatim: 1 request/second for bulk requests
- Implement proper caching to minimize API calls
- Handle rate limit errors gracefully

### Network Security
- Use HTTPS for all API calls
- Don't hardcode API keys in source code
- Consider using Android Keystore for sensitive data
- Implement certificate pinning for production apps
