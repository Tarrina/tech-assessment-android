# Rain Check App - Weather Forecast Application

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

**UI Reference**: Use `rain-check.html` in the `ui/` folder as inspiration for the visual design and user experience flow.

---

## Core Features

### 1. Location Detection
- **Automatic Location**: Request location permissions and get current GPS coordinates
- **Location Display**: Show the detected city/address to the user
- **Error Handling**: Handle permission denied, location disabled, or detection failures
- **Manual Refresh**: Allow users to refresh/update their location

### 2. Date Selection
- **Future Dates Only**: Date picker that only allows selection of future dates (today + next 7 days)
- **Intuitive UI**: Material Design date picker component
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
https://nominatim.openstreetmap.org/reverse?format=json&lat=23.0225&lon=72.5714&addressdetails=1
```

**Example Response**:
```json
{
  "place_id": "2825870",
  "licence": "Data © OpenStreetMap contributors, ODbL 1.0.",
  "osm_type": "way",
  "osm_id": "234567890",
  "lat": "23.0225",
  "lon": "72.5714",
  "display_name": "42, Ellis Bridge, Paldi, Ahmedabad, Gujarat, 380007, India",
  "address": {
    "house_number": "42",
    "road": "Ellis Bridge",
    "neighbourhood": "Paldi",
    "city": "Ahmedabad",
    "state": "Gujarat",
    "postcode": "380007",
    "country": "India",
    "country_code": "in"
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
https://api.openweathermap.org/data/2.5/forecast?lat=23.0225&lon=72.5714&appid=YOUR_API_KEY&units=metric
```

**Example Response Structure**:
```json
{
  "cod": "200",
  "message": 0,
  "cnt": 40,
  "list": [
    {
      "dt": 1747526400,
      "main": {
        "temp": 28.5,
        "feels_like": 31.2,
        "temp_min": 26.8,
        "temp_max": 30.1,
        "pressure": 1012,
        "sea_level": 1012,
        "grnd_level": 1009,
        "humidity": 65
      },
      "weather": [
        {
          "id": 500,
          "main": "Rain",
          "description": "light rain",
          "icon": "10d"
        }
      ],
      "clouds": {
        "all": 75
      },
      "wind": {
        "speed": 3.2,
        "deg": 235
      },
      "pop": 0.68,
      "rain": {
        "3h": 1.2
      },
      "sys": {
        "pod": "d"
      },
      "dt_txt": "2025-05-30 12:00:00"
    }
  ],
  "city": {
    "id": 1279233,
    "name": "Ahmedabad",
    "coord": {
      "lat": 23.0225,
      "lon": 72.5714
    },
    "country": "IN"
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
- **Location Services**: Proper permission handling and location detection
- **Code Quality**: Readable, maintainable Kotlin code with appropriate patterns

### UI/UX Design (30%)
- **Material Design**: Correct implementation of Material theming and components
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
- Fully functional Rain Check weather application
- All source code with proper git history
- Working APK file

### 2. Documentation
- **README.md** with:
  - Setup and run instructions
  - Known limitations or assumptions
  - Reference to UI design inspiration from rain-check.html

### 3. Demo Video
  - 30 seconds screen recording showing:
  - Location detection and permission flow
  - Date selection functionality
  - Weather forecast display for different dates
  - Error handling (if possible to demonstrate)
