<script lang="ts">
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
    import type { Map as LeafletMap } from 'leaflet';

    interface WeatherData {
        location: string;
        temperature: string;
        description: string;
        humidity: string;
        windSpeed: string;
        pressure: string;
        visibility: string;
        feelsLike: string;
        sunrise: string;
        sunset: string;
        timezone: string;
        localTime: string;
        uvIndex: string;
        airQuality: string;
        dewPoint: string;
        cloudCover: string;
        moonPhase: string;
        windDirection: string;
        pollenIndex: string;
        driveConditions: string;
        beachTemp: string;
        outfitSuggestion: string;
    }

    interface ForecastDay {
        date: string;
        temp: string;
        tempMin: string;
        tempMax: string;
        description: string;
        icon: string;
        humidity: string;
        windSpeed: string;
        precipitation: string;
    }

    interface HourlyForecast {
        time: string;
        temp: string;
        icon: string;
        description: string;
        precipitation: string;
    }

    interface FavoriteCity {
        name: string;
        country: string;
        temp: string;
        description: string;
        icon: string;
    }



    const favoriteStore = writable<FavoriteCity[]>([]);
    const themeStore = writable('auto');

    let weatherData: WeatherData | null = null;
    let compareWeatherData: WeatherData | null = null;
    let forecastData: ForecastDay[] = [];
    let hourlyData: HourlyForecast[] = [];
    let favoriteCities: FavoriteCity[] = [];
    let loading = true;
    let error = '';
    let searchCity = '';
    let compareCity = '';
    let currentTime = new Date();
    let activeTab = 'current';
    let showFavorites = false;
    let showComparison = false;
    let currentTheme = 'auto';
    let weatherHistory: any[] = [];
    let showWeatherMap = false;
    let showHistoricalData = false;
    let showExportDialog = false;

    let mapCenter = { lat: 41.0082, lng: 28.9784 };
    let mapZoom = 10;
    let weatherMapType = 'precipitation';
    let leafletMap: LeafletMap | null = null;
    let mapContainer: HTMLElement;

    onMount(() => {
        getCurrentLocationWeather();
        loadFavorites();
        loadTheme();
        loadWeatherHistory();
        
        const interval = setInterval(() => {
            currentTime = new Date();
            updateDynamicTheme();
        }, 1000);
        
        return () => clearInterval(interval);
    });





    function loadFavorites() {
        if (typeof localStorage !== 'undefined') {
            const saved = localStorage.getItem('weather-favorites');
            if (saved) {
                favoriteCities = JSON.parse(saved);
                favoriteStore.set(favoriteCities);
            }
        }
    }

    function saveFavorites() {
        if (typeof localStorage !== 'undefined') {
            localStorage.setItem('weather-favorites', JSON.stringify(favoriteCities));
        }
    }

    function loadTheme() {
        if (typeof localStorage !== 'undefined') {
            const saved = localStorage.getItem('weather-theme');
            if (saved) {
                currentTheme = saved;
                themeStore.set(currentTheme);
                applyTheme(currentTheme);
            }
        }
    }

    function saveTheme(theme: string) {
        currentTheme = theme;
        if (typeof localStorage !== 'undefined') {
            localStorage.setItem('weather-theme', theme);
        }
        themeStore.set(theme);
        applyTheme(theme);
    }

    function applyTheme(theme: string) {
        if (typeof document !== 'undefined') {
            const body = document.body;
            const mainContainer = document.querySelector('.main-container') as HTMLElement;
            
            let gradient = '';
            
            switch(theme) {
                case 'sunset':
                    gradient = 'linear-gradient(135deg, #ff6b6b 0%, #ff8e53 50%, #ff6b9d 100%)';
                    break;
                case 'ocean':
                    gradient = 'linear-gradient(135deg, #667eea 0%, #764ba2 50%, #89f7fe 100%)';
                    break;
                case 'forest':
                    gradient = 'linear-gradient(135deg, #134e5e 0%, #71b280 50%, #38ef7d 100%)';
                    break;
                case 'night':
                    gradient = 'linear-gradient(135deg, #000428 0%, #004e92 50%, #009ffd 100%)';
                    break;
                default:
                    updateDynamicTheme();
                    return;
            }
            
            body.style.background = gradient;
            if (mainContainer) {
                mainContainer.style.background = gradient;
            }
            
            document.documentElement.style.setProperty('--bg-gradient', gradient);
        }
    }

    function updateDynamicTheme() {
        if (currentTheme === 'auto' && weatherData && typeof document !== 'undefined') {
            const body = document.body;
            const mainContainer = document.querySelector('.main-container') as HTMLElement;
            const hour = new Date().getHours();
            const isNight = hour < 6 || hour > 20;
            const temp = parseInt(weatherData.temperature);
            
            let gradient = '';
            
            if (isNight) {
                gradient = 'linear-gradient(135deg, #0f0f23 0%, #2d1b69 50%, #1a1a2e 100%)';
            } else if (temp > 30) {
                gradient = 'linear-gradient(135deg, #ff6b6b 0%, #ff8e53 50%, #ff6b9d 100%)';
            } else if (temp < 10) {
                gradient = 'linear-gradient(135deg, #667eea 0%, #764ba2 50%, #c7d2fe 100%)';
            } else if (weatherData.description.toLowerCase().includes('rain')) {
                gradient = 'linear-gradient(135deg, #536976 0%, #667db6 50%, #5596c2 100%)';
            } else {
                gradient = 'linear-gradient(135deg, #667eea 0%, #764ba2 50%, #ffeaa7 100%)';
            }
            
            body.style.background = gradient;
            if (mainContainer) {
                mainContainer.style.background = gradient;
            }
            
            document.documentElement.style.setProperty('--bg-gradient', gradient);
        }
    }

    function addToFavorites() {
        if (weatherData && !favoriteCities.find(city => city.name === weatherData!.location)) {
            const newFavorite: FavoriteCity = {
                name: weatherData.location,
                country: weatherData.location.split(', ')[1] || '',
                temp: weatherData.temperature,
                description: weatherData.description,
                icon: getWeatherIcon(weatherData.description)
            };
            
            favoriteCities = [...favoriteCities, newFavorite];
            saveFavorites();
            favoriteStore.set(favoriteCities);
            
            showNotification('🌟 Added to favorites!', 'success');
        }
    }

    function removeFromFavorites(cityName: string) {
        favoriteCities = favoriteCities.filter(city => city.name !== cityName);
        saveFavorites();
        favoriteStore.set(favoriteCities);
        showNotification('🗑️ Removed from favorites', 'info');
    }

    function loadFavoriteCity(cityName: string) {
        searchCity = cityName.split(', ')[0];
        handleSearch();
    }



    function showNotification(message: string, type: string) {
        const notification = document.createElement('div');
        notification.className = `fixed top-4 right-4 z-50 p-4 rounded-xl backdrop-blur-xl border text-white font-semibold animate-fade-in ${
            type === 'success' ? 'bg-green-500/20 border-green-400/30' :
            type === 'error' ? 'bg-red-500/20 border-red-400/30' :
            'bg-blue-500/20 border-blue-400/30'
        }`;
        notification.textContent = message;
        document.body.appendChild(notification);
        
        setTimeout(() => {
            notification.remove();
        }, 3000);
    }



    function getOutfitSuggestion(temp: number, description: string): string {
        if (temp > 30) return "👕 Light clothing, shorts, sunglasses, hat";
        if (temp > 20) return "👔 T-shirt, light pants, comfortable shoes";
        if (temp > 10) return "🧥 Light jacket, long pants, closed shoes";
        if (temp > 0) return "🧥 Warm coat, scarf, gloves, boots";
        return "❄️ Heavy winter coat, warm layers, winter boots";
    }

    function getPollenIndex(): string {
        const month = new Date().getMonth();
        const temp = weatherData ? parseInt(weatherData.temperature) : 20;
        
        if (month >= 2 && month <= 5 && temp > 15) return "High";
        if (month >= 6 && month <= 8 && temp > 10) return "Medium";
        return "Low";
    }

    function getDriveConditions(): string {
        if (!weatherData) return "Good";
        
        const visibility = parseInt(weatherData.visibility);
        const description = weatherData.description.toLowerCase();
        
        if (visibility < 5 || description.includes('fog')) return "Poor - Low visibility";
        if (description.includes('rain') || description.includes('snow')) return "Moderate - Wet roads";
        if (description.includes('wind')) return "Moderate - Strong winds";
        return "Good - Clear roads";
    }

    function getBeachConditions(): string {
        if (!weatherData) return "20°C";
        
        const temp = parseInt(weatherData.temperature);
        const seaTemp = Math.max(10, temp - Math.floor(Math.random() * 8) - 2);
        return `${seaTemp}°C`;
    }

    async function getCurrentLocationWeather() {
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(
                (position) => {
                    const { latitude, longitude } = position.coords;
                    fetchWeatherByCoords(latitude, longitude);
                },
                () => {
                    fetchWeatherByCity('Istanbul');
                }
            );
        } else {
            fetchWeatherByCity('Istanbul');
        }
    }

    async function fetchWeatherByCoords(lat: number, lon: number) {
        try {
            loading = true;
            const response = await fetch(`https://wttr.in/${lat},${lon}?format=j1`);
            const data = await response.json();
            parseWeatherData(data);
            error = '';
        } catch (err) {
            error = 'Failed to fetch weather data';
            showDemoData();
            console.error(err);
        } finally {
            loading = false;
        }
    }

    async function fetchWeatherByCity(city: string) {
        try {
            loading = true;
            const response = await fetch(`https://wttr.in/${encodeURIComponent(city)}?format=j1`);
            
            if (!response.ok) {
                throw new Error('City not found');
            }
            
            const data = await response.json();
            parseWeatherData(data);
            error = '';
        } catch (err) {
            error = 'Failed to fetch weather data. Please check the city name.';
            showDemoData();
            console.error(err);
        } finally {
            loading = false;
        }
    }

    async function fetchCompareWeather(city: string) {
        try {
            const response = await fetch(`https://wttr.in/${encodeURIComponent(city)}?format=j1`);
            
            if (!response.ok) {
                throw new Error('City not found');
            }
            
            const data = await response.json();
            parseCompareWeatherData(data);
            showComparison = true;
        } catch (err) {
            showNotification('❌ Could not fetch comparison data', 'error');
            console.error(err);
        }
    }

    function parseWeatherData(data: any) {
        const current = data.current_condition[0];
        const location = data.nearest_area[0];
        
        const temp = parseInt(current.temp_C);
        
        weatherData = {
            location: `${location.areaName[0].value}, ${location.country[0].value}`,
            temperature: current.temp_C,
            description: current.weatherDesc[0].value,
            humidity: current.humidity,
            windSpeed: current.windspeedKmph,
            pressure: current.pressure,
            visibility: current.visibility,
            feelsLike: current.FeelsLikeC,
            sunrise: data.weather[0].astronomy[0].sunrise,
            sunset: data.weather[0].astronomy[0].sunset,
            timezone: location.region?.[0]?.value || 'UTC',
            localTime: new Date().toLocaleString(),
            uvIndex: current.uvIndex || '0',
            airQuality: getAirQualityFromVisibility(current.visibility),
            dewPoint: (parseFloat(current.temp_C) - (100 - parseFloat(current.humidity)) / 5).toFixed(1),
            cloudCover: current.cloudcover,
            moonPhase: getMoonPhase(),
            windDirection: current.winddir16Point,
            pollenIndex: getPollenIndex(),
            driveConditions: getDriveConditions(),
            beachTemp: getBeachConditions(),
            outfitSuggestion: getOutfitSuggestion(temp, current.weatherDesc[0].value)
        };

        forecastData = data.weather.slice(1, 8).map((day: any, index: number) => ({
            date: getDayName(index + 1),
            temp: day.avgtempC,
            tempMin: day.mintempC,
            tempMax: day.maxtempC,
            description: day.hourly[4].weatherDesc[0].value,
            icon: getWeatherIcon(day.hourly[4].weatherCode),
            humidity: day.hourly[4].humidity,
            windSpeed: day.hourly[4].windspeedKmph,
            precipitation: day.hourly[4].precipMM || '0'
        }));

        hourlyData = data.weather[0].hourly.map((hour: any) => ({
            time: formatHour(hour.time),
            temp: hour.tempC,
            icon: getWeatherIcon(hour.weatherCode),
            description: hour.weatherDesc[0].value,
            precipitation: hour.precipMM || '0'
        }));
        
        updateDynamicTheme();
        updateMapCenter();
        saveToHistory(weatherData);
    }

    function parseCompareWeatherData(data: any) {
        const current = data.current_condition[0];
        const location = data.nearest_area[0];
        
        const temp = parseInt(current.temp_C);
        
        compareWeatherData = {
            location: `${location.areaName[0].value}, ${location.country[0].value}`,
            temperature: current.temp_C,
            description: current.weatherDesc[0].value,
            humidity: current.humidity,
            windSpeed: current.windspeedKmph,
            pressure: current.pressure,
            visibility: current.visibility,
            feelsLike: current.FeelsLikeC,
            sunrise: data.weather[0].astronomy[0].sunrise,
            sunset: data.weather[0].astronomy[0].sunset,
            timezone: location.region?.[0]?.value || 'UTC',
            localTime: new Date().toLocaleString(),
            uvIndex: current.uvIndex || '0',
            airQuality: getAirQualityFromVisibility(current.visibility),
            dewPoint: (parseFloat(current.temp_C) - (100 - parseFloat(current.humidity)) / 5).toFixed(1),
            cloudCover: current.cloudcover,
            moonPhase: getMoonPhase(),
            windDirection: current.winddir16Point,
            pollenIndex: getPollenIndex(),
            driveConditions: getDriveConditions(),
            beachTemp: getBeachConditions(),
            outfitSuggestion: getOutfitSuggestion(temp, current.weatherDesc[0].value)
        };
    }

    function showDemoData() {
        weatherData = {
            location: "Istanbul, Turkey",
            temperature: "22",
            description: "Partly cloudy",
            humidity: "65",
            windSpeed: "12",
            pressure: "1013",
            visibility: "10",
            feelsLike: "24",
            sunrise: "06:45 AM",
            sunset: "07:30 PM",
            timezone: "Europe/Istanbul",
            localTime: new Date().toLocaleString(),
            uvIndex: "3",
            airQuality: "Good",
            dewPoint: "15.0",
            cloudCover: "40",
            moonPhase: "🌓",
            windDirection: "NW",
            pollenIndex: "Medium",
            driveConditions: "Good - Clear roads",
            beachTemp: "20°C",
            outfitSuggestion: "👔 T-shirt, light pants, comfortable shoes"
        };

        forecastData = [
            { date: "Tomorrow", temp: "20", tempMin: "16", tempMax: "24", description: "Cloudy", icon: "☁️", humidity: "70", windSpeed: "10", precipitation: "0" },
            { date: "Thursday", temp: "18", tempMin: "14", tempMax: "22", description: "Light rain", icon: "🌧️", humidity: "80", windSpeed: "15", precipitation: "2.5" },
            { date: "Friday", temp: "16", tempMin: "12", tempMax: "20", description: "Sunny", icon: "☀️", humidity: "55", windSpeed: "8", precipitation: "0" },
            { date: "Saturday", temp: "19", tempMin: "15", tempMax: "23", description: "Partly cloudy", icon: "⛅", humidity: "60", windSpeed: "12", precipitation: "0" },
            { date: "Sunday", temp: "21", tempMin: "17", tempMax: "25", description: "Clear", icon: "☀️", humidity: "58", windSpeed: "9", precipitation: "0" },
            { date: "Monday", temp: "23", tempMin: "19", tempMax: "27", description: "Sunny", icon: "☀️", humidity: "52", windSpeed: "11", precipitation: "0" },
            { date: "Tuesday", temp: "20", tempMin: "16", tempMax: "24", description: "Cloudy", icon: "☁️", humidity: "68", windSpeed: "13", precipitation: "0" }
        ];

        hourlyData = [
            { time: "Now", temp: "22", icon: "⛅", description: "Partly cloudy", precipitation: "0" },
            { time: "15:00", temp: "24", icon: "☀️", description: "Sunny", precipitation: "0" },
            { time: "18:00", temp: "21", icon: "⛅", description: "Partly cloudy", precipitation: "0" },
            { time: "21:00", temp: "19", icon: "☁️", description: "Cloudy", precipitation: "0" },
            { time: "00:00", temp: "17", icon: "🌙", description: "Clear night", precipitation: "0" },
            { time: "03:00", temp: "15", icon: "🌙", description: "Clear night", precipitation: "0" },
            { time: "06:00", temp: "16", icon: "🌅", description: "Partly cloudy", precipitation: "0" },
            { time: "09:00", temp: "20", icon: "☀️", description: "Sunny", precipitation: "0" }
        ];
    }

    function handleSearch() {
        if (searchCity.trim()) {
            fetchWeatherByCity(searchCity.trim());
        }
    }

    function handleCompareSearch() {
        if (compareCity.trim()) {
            fetchCompareWeather(compareCity.trim());
        }
    }

    function getWeatherIcon(code: string): string {
        const iconMap: Record<string, string> = {
            '113': '☀️', '116': '⛅', '119': '☁️', '122': '☁️', '143': '🌫️',
            '176': '🌦️', '179': '🌨️', '182': '🌧️', '185': '🌧️', '200': '⛈️',
            '227': '🌨️', '230': '❄️', '248': '🌫️', '260': '🌫️', '263': '🌦️',
            '296': '🌦️', '299': '🌧️', '302': '🌧️', '305': '🌧️', '308': '🌧️',
            '311': '🌧️', '314': '🌧️', '317': '🌧️', '320': '🌨️', '323': '🌨️',
            '326': '🌨️', '329': '❄️', '332': '❄️', '335': '❄️', '338': '❄️',
            '350': '🌧️', '353': '🌦️', '356': '🌧️', '359': '🌧️', '362': '🌨️',
            '365': '🌨️', '368': '🌨️', '371': '❄️', '374': '🌧️', '377': '🌧️',
            '386': '⛈️', '389': '⛈️', '392': '⛈️', '395': '❄️'
        };
        return iconMap[code] || '☀️';
    }

    function getDayName(daysFromNow: number): string {
        const date = new Date();
        date.setDate(date.getDate() + daysFromNow);
        return date.toLocaleDateString('en-US', { weekday: 'short' });
    }

    function formatHour(hour: string): string {
        if (hour === "0") return "Now";
        const time = parseInt(hour);
        return `${String(Math.floor(time / 100)).padStart(2, '0')}:00`;
    }

    function getAirQualityFromVisibility(visibility: string): string {
        const vis = parseFloat(visibility);
        if (vis >= 10) return "Excellent";
        if (vis >= 7) return "Good";
        if (vis >= 4) return "Moderate";
        if (vis >= 2) return "Poor";
        return "Very Poor";
    }

    function getMoonPhase(): string {
        const phases = ['🌑', '🌒', '🌓', '🌔', '🌕', '🌖', '🌗', '🌘'];
        const today = new Date();
        const moonCycle = 29.53059;
        const known = new Date(2000, 0, 6);
        const phase = ((today.getTime() - known.getTime()) / (1000 * 60 * 60 * 24)) % moonCycle;
        return phases[Math.floor((phase / moonCycle) * 8)];
    }

    function getUVLevel(uv: string): string {
        const level = parseFloat(uv);
        if (level <= 2) return "Low";
        if (level <= 5) return "Moderate";
        if (level <= 7) return "High";
        if (level <= 10) return "Very High";
        return "Extreme";
    }

    function getWindDirection(dir: string): string {
        const directions: Record<string, string> = {
            'N': '⬆️', 'NNE': '↗️', 'NE': '↗️', 'ENE': '↗️',
            'E': '➡️', 'ESE': '↘️', 'SE': '↘️', 'SSE': '↘️',
            'S': '⬇️', 'SSW': '↙️', 'SW': '↙️', 'WSW': '↙️',
            'W': '⬅️', 'WNW': '↖️', 'NW': '↖️', 'NNW': '↖️'
        };
        return directions[dir] || '��';
    }

    function loadWeatherHistory() {
        if (typeof localStorage !== 'undefined') {
            const saved = localStorage.getItem('weather-history');
            if (saved) {
                weatherHistory = JSON.parse(saved);
            }
        }
    }

    function saveToHistory(data: WeatherData) {
        const historyEntry = {
            ...data,
            timestamp: Date.now(),
            date: new Date().toLocaleDateString()
        };
        
        weatherHistory = [historyEntry, ...weatherHistory.slice(0, 9)]; // Keep last 10 entries
        
        if (typeof localStorage !== 'undefined') {
            localStorage.setItem('weather-history', JSON.stringify(weatherHistory));
        }
    }

    function exportWeatherData(format: 'json' | 'csv') {
        if (!weatherData) return;
        
        const data = {
            location: weatherData.location,
            temperature: weatherData.temperature,
            description: weatherData.description,
            humidity: weatherData.humidity,
            windSpeed: weatherData.windSpeed,
            pressure: weatherData.pressure,
            uvIndex: weatherData.uvIndex,
            exportDate: new Date().toISOString(),
            forecast: forecastData
        };

        let blob: Blob;
        let filename: string;

        if (format === 'json') {
            blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
            filename = `weather-${weatherData.location.replace(/\s+/g, '-')}-${new Date().toISOString().split('T')[0]}.json`;
        } else {
            const csvContent = [
                'Date,Location,Temperature,Description,Humidity,Wind Speed,Pressure,UV Index',
                `${new Date().toLocaleDateString()},${weatherData.location},${weatherData.temperature}°C,${weatherData.description},${weatherData.humidity}%,${weatherData.windSpeed} km/h,${weatherData.pressure} hPa,${weatherData.uvIndex}`
            ].join('\n');
            
            blob = new Blob([csvContent], { type: 'text/csv' });
            filename = `weather-${weatherData.location.replace(/\s+/g, '-')}-${new Date().toISOString().split('T')[0]}.csv`;
        }

        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = filename;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        
        showNotification(`📄 Exported as ${format.toUpperCase()}`, 'success');
        showExportDialog = false;
    }

    function getWeatherMapUrl(type: string): string {
        const apiKey = 'YOUR_OPENWEATHER_API_KEY'; // Gerçek kullanımda API key ekleyin
        const lat = mapCenter.lat;
        const lon = mapCenter.lng;
        const zoom = mapZoom;
        
        const mapTypes = {
            precipitation: `https://tile.openweathermap.org/map/precipitation_new/${zoom}/${Math.floor((lon + 180) / 360 * Math.pow(2, zoom))}/${Math.floor((1 - Math.log(Math.tan(lat * Math.PI / 180) + 1 / Math.cos(lat * Math.PI / 180)) / Math.PI) / 2 * Math.pow(2, zoom))}.png?appid=${apiKey}`,
            temperature: `https://tile.openweathermap.org/map/temp_new/${zoom}/${Math.floor((lon + 180) / 360 * Math.pow(2, zoom))}/${Math.floor((1 - Math.log(Math.tan(lat * Math.PI / 180) + 1 / Math.cos(lat * Math.PI / 180)) / Math.PI) / 2 * Math.pow(2, zoom))}.png?appid=${apiKey}`,
            wind: `https://tile.openweathermap.org/map/wind_new/${zoom}/${Math.floor((lon + 180) / 360 * Math.pow(2, zoom))}/${Math.floor((1 - Math.log(Math.tan(lat * Math.PI / 180) + 1 / Math.cos(lat * Math.PI / 180)) / Math.PI) / 2 * Math.pow(2, zoom))}.png?appid=${apiKey}`,
            clouds: `https://tile.openweathermap.org/map/clouds_new/${zoom}/${Math.floor((lon + 180) / 360 * Math.pow(2, zoom))}/${Math.floor((1 - Math.log(Math.tan(lat * Math.PI / 180) + 1 / Math.cos(lat * Math.PI / 180)) / Math.PI) / 2 * Math.pow(2, zoom))}.png?appid=${apiKey}`
        };
        
        return mapTypes[type as keyof typeof mapTypes] || mapTypes.precipitation;
    }

    function updateMapCenter() {
        if (weatherData) {
            const locations = {
                'Istanbul, Turkey': { lat: 41.0082, lng: 28.9784 },
                'London, United Kingdom': { lat: 51.5074, lng: -0.1278 },
                'New York, United States': { lat: 40.7128, lng: -74.0060 },
                'Tokyo, Japan': { lat: 35.6762, lng: 139.6503 },
                'Paris, France': { lat: 48.8566, lng: 2.3522 }
            };
            
            const location = locations[weatherData.location as keyof typeof locations];
            if (location) {
                mapCenter = location;
                if (leafletMap) {
                    leafletMap.setView([location.lat, location.lng], mapZoom);
                }
            }
        }
    }

    async function initLeafletMap() {
        console.log('initLeafletMap called');
        console.log('mapContainer:', mapContainer);
        console.log('leafletMap:', leafletMap);
        
        if (typeof window === 'undefined') {
            console.log('Not in browser environment');
            return;
        }
        
        if (!mapContainer) {
            console.log('Map container not found');
            return;
        }
        
        if (leafletMap) {
            console.log('Map already initialized');
            return;
        }

        try {
            const L = await import('leaflet');
            console.log('Leaflet imported successfully');
            
            delete (L.Icon.Default.prototype as any)._getIconUrl;
            L.Icon.Default.mergeOptions({
                iconRetinaUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/images/marker-icon-2x.png',
                iconUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/images/marker-icon.png',
                shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/images/marker-shadow.png',
            });
            
            console.log('Container dimensions:', mapContainer.offsetWidth, mapContainer.offsetHeight);
            
            leafletMap = L.map(mapContainer, {
                center: [mapCenter.lat, mapCenter.lng],
                zoom: mapZoom,
                zoomControl: true,
                attributionControl: true
            });
            
            console.log('Map created');

            const tileLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap contributors',
                maxZoom: 18
            });
            
            tileLayer.addTo(leafletMap);
            console.log('Tile layer added');

            if (weatherData) {
                const marker = L.marker([mapCenter.lat, mapCenter.lng])
                    .addTo(leafletMap)
                    .bindPopup(`
                        <div style="text-align: center; padding: 8px;">
                            <h3 style="font-weight: bold; font-size: 16px; margin: 0 0 8px 0;">${weatherData.location}</h3>
                            <p style="font-size: 18px; margin: 0 0 4px 0;">${weatherData.temperature}°C</p>
                            <p style="margin: 0;">${weatherData.description}</p>
                        </div>
                    `);
                console.log('Marker added');
            }

            addWeatherLayer(L);
            
            setTimeout(() => {
                if (leafletMap) {
                    leafletMap.invalidateSize();
                    console.log('Map size invalidated');
                }
            }, 100);
            
            console.log('Map initialization completed');

        } catch (error) {
            console.error('Error initializing map:', error);
        }
    }

    function addWeatherLayer(L: any) {
        if (!leafletMap) {
            console.log('No leaflet map for weather layer');
            return;
        }

        try {
            leafletMap.eachLayer((layer: any) => {
                if (layer.options && layer.options.weatherLayer) {
                    leafletMap?.removeLayer(layer);
                }
            });

            const overlayColor = {
                precipitation: 'rgba(0, 100, 255, 0.3)',
                temperature: 'rgba(255, 100, 0, 0.3)', 
                wind: 'rgba(0, 255, 100, 0.3)',
                clouds: 'rgba(200, 200, 200, 0.3)'
            };

            const bounds = leafletMap.getBounds();
            const weatherOverlay = L.rectangle(bounds, {
                color: overlayColor[weatherMapType as keyof typeof overlayColor],
                weight: 1,
                fillOpacity: 0.2,
                weatherLayer: true
            }).addTo(leafletMap);

            console.log('Weather layer added:', weatherMapType);
        } catch (error) {
            console.error('Error adding weather layer:', error);
        }
    }

    function changeWeatherMapType(type: string) {
        weatherMapType = type;
        if (leafletMap && typeof window !== 'undefined') {
            import('leaflet').then(L => {
                addWeatherLayer(L);
            });
        }
    }
</script>

<div class="main-container min-h-screen w-full bg-gradient-to-br from-purple-900 via-blue-900 to-indigo-900 text-white overflow-hidden">
    <!-- Favori Şehirler Paneli -->
    {#if showFavorites}
    <div class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4">
        <div class="bg-white/10 backdrop-blur-xl rounded-3xl p-8 max-w-2xl w-full max-h-[80vh] overflow-y-auto border border-white/20">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-black">🌟 Favorite Cities</h2>
                <button 
                    on:click={() => showFavorites = false}
                    class="text-white/70 hover:text-white text-2xl"
                >×</button>
            </div>
            
            {#if favoriteCities.length === 0}
                <p class="text-white/70 text-center py-8">No favorite cities yet. Add some!</p>
            {:else}
                <div class="grid gap-4">
                    {#each favoriteCities as city}
                        <div class="bg-white/10 rounded-2xl p-4 flex items-center justify-between hover:bg-white/20 transition-all">
                            <div class="flex items-center gap-4">
                                <span class="text-3xl">{city.icon}</span>
                                <div>
                                    <h3 class="font-bold text-lg">{city.name}</h3>
                                    <p class="text-white/70">{city.description}</p>
                                </div>
                            </div>
                            <div class="flex items-center gap-3">
                                <span class="text-2xl font-black">{city.temp}°C</span>
                                <button 
                                    on:click={() => loadFavoriteCity(city.name)}
                                    class="bg-purple-500 hover:bg-purple-600 px-4 py-2 rounded-xl text-sm font-semibold transition-all"
                                >Load</button>
                                <button 
                                    on:click={() => removeFromFavorites(city.name)}
                                    class="text-red-400 hover:text-red-300 text-xl"
                                >🗑️</button>
                            </div>
                        </div>
                    {/each}
                </div>
            {/if}
        </div>
    </div>
    {/if}

    <!-- Şehir Karşılaştırma Paneli -->
    {#if showComparison}
    <div class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4">
        <div class="bg-white/10 backdrop-blur-xl rounded-3xl p-8 max-w-6xl w-full max-h-[80vh] overflow-y-auto border border-white/20">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-black">🏙️ City Comparison</h2>
                <button 
                    on:click={() => showComparison = false}
                    class="text-white/70 hover:text-white text-2xl"
                >×</button>
            </div>
            
            {#if weatherData && compareWeatherData}
                <div class="grid md:grid-cols-2 gap-6">
                    <!-- Şehir 1 -->
                    <div class="bg-white/10 rounded-2xl p-6 border border-white/20">
                        <h3 class="text-xl font-black mb-4">{weatherData.location}</h3>
                        <div class="text-center mb-4">
                            <div class="text-6xl font-black">{weatherData.temperature}°C</div>
                            <div class="text-white/70">{weatherData.description}</div>
                        </div>
                        <div class="grid grid-cols-2 gap-4 text-sm">
                            <div>Feels Like: {weatherData.feelsLike}°C</div>
                            <div>Humidity: {weatherData.humidity}%</div>
                            <div>Wind: {weatherData.windSpeed} km/h</div>
                            <div>UV Index: {weatherData.uvIndex}</div>
                        </div>
                    </div>
                    
                    <!-- Şehir 2 -->
                    <div class="bg-white/10 rounded-2xl p-6 border border-white/20">
                        <h3 class="text-xl font-black mb-4">{compareWeatherData.location}</h3>
                        <div class="text-center mb-4">
                            <div class="text-6xl font-black">{compareWeatherData.temperature}°C</div>
                            <div class="text-white/70">{compareWeatherData.description}</div>
                        </div>
                        <div class="grid grid-cols-2 gap-4 text-sm">
                            <div>Feels Like: {compareWeatherData.feelsLike}°C</div>
                            <div>Humidity: {compareWeatherData.humidity}%</div>
                            <div>Wind: {compareWeatherData.windSpeed} km/h</div>
                            <div>UV Index: {compareWeatherData.uvIndex}</div>
                        </div>
                    </div>
                </div>
                
                <!-- Karşılaştırma Özeti -->
                <div class="mt-6 bg-white/10 rounded-2xl p-6 border border-white/20">
                    <h4 class="text-lg font-black mb-4">📊 Comparison Summary</h4>
                    <div class="grid md:grid-cols-3 gap-4 text-sm">
                        <div>
                            <strong>Temperature Difference:</strong>
                            {Math.abs(parseInt(weatherData.temperature) - parseInt(compareWeatherData.temperature))}°C
                        </div>
                        <div>
                            <strong>Humidity Difference:</strong>
                            {Math.abs(parseInt(weatherData.humidity) - parseInt(compareWeatherData.humidity))}%
                        </div>
                        <div>
                            <strong>Wind Difference:</strong>
                            {Math.abs(parseInt(weatherData.windSpeed) - parseInt(compareWeatherData.windSpeed))} km/h
                        </div>
                    </div>
                </div>
            {/if}
        </div>
    </div>
    {/if}

    <!-- Weather Map Paneli -->
    {#if showWeatherMap}
    <div class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4">
        <div class="bg-white/10 backdrop-blur-xl rounded-3xl p-8 max-w-6xl w-full max-h-[80vh] overflow-y-auto border border-white/20">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-black">🗺️ Weather Map</h2>
                <button 
                    on:click={() => showWeatherMap = false}
                    class="text-white/70 hover:text-white text-2xl"
                >×</button>
            </div>
            
            <!-- Map Controls -->
            <div class="flex gap-2 mb-6 overflow-x-auto">
                <button 
                    on:click={() => changeWeatherMapType('precipitation')}
                    class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {weatherMapType === 'precipitation' ? 'bg-blue-500 text-white' : 'bg-white/10 text-white/70'}"
                >🌧️ Rain</button>
                <button 
                    on:click={() => changeWeatherMapType('temperature')}
                    class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {weatherMapType === 'temperature' ? 'bg-red-500 text-white' : 'bg-white/10 text-white/70'}"
                >🌡️ Temperature</button>
                <button 
                    on:click={() => changeWeatherMapType('wind')}
                    class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {weatherMapType === 'wind' ? 'bg-green-500 text-white' : 'bg-white/10 text-white/70'}"
                >💨 Wind</button>
                <button 
                    on:click={() => changeWeatherMapType('clouds')}
                    class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {weatherMapType === 'clouds' ? 'bg-gray-500 text-white' : 'bg-white/10 text-white/70'}"
                >☁️ Clouds</button>
            </div>
            
            <!-- Leaflet Map Container -->
            <div class="bg-white/10 rounded-2xl p-4 border border-white/20">
                <div class="mb-4 text-center">
                    <h3 class="text-xl font-black mb-2">🗺️ Interactive Weather Map</h3>
                    <p class="text-white/70 text-sm">
                        Showing {weatherMapType} overlay for {weatherData?.location || 'current location'}
                    </p>
                </div>
                
                <!-- Harita Container -->
                <div class="relative">
                    <div 
                        bind:this={mapContainer}
                        class="w-full h-96 rounded-xl overflow-hidden bg-gray-800 border-2 border-white/20"
                        style="min-height: 384px;"
                    >
                        {#if !leafletMap}
                            <div class="flex items-center justify-center h-full text-white/70">
                                <div class="text-center">
                                    <div class="text-4xl mb-2">🗺️</div>
                                    <p class="text-lg">Loading interactive map...</p>
                                </div>
                            </div>
                        {/if}
                    </div>
                    
                    {#if !leafletMap}
                        <div class="absolute inset-0 flex items-center justify-center">
                            <button 
                                on:click={initLeafletMap}
                                class="bg-purple-500 hover:bg-purple-600 px-6 py-3 rounded-xl font-semibold transition-all z-10"
                            >
                                🗺️ Load Interactive Map
                            </button>
                        </div>
                    {/if}
                </div>
                
                <!-- Harita bilgileri -->
                <div class="mt-4 grid grid-cols-2 gap-4 text-sm">
                    <div class="bg-white/5 rounded-xl p-3">
                        <p class="text-white/60">📍 Location</p>
                        <p class="font-bold">{mapCenter.lat.toFixed(4)}, {mapCenter.lng.toFixed(4)}</p>
                    </div>
                    <div class="bg-white/5 rounded-xl p-3">
                        <p class="text-white/60">🔍 Status</p>
                        <p class="font-bold">{leafletMap ? '✅ Active' : '⏳ Ready to Load'}</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
    {/if}

    <!-- Historical Data Paneli -->
    {#if showHistoricalData}
    <div class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4">
        <div class="bg-white/10 backdrop-blur-xl rounded-3xl p-8 max-w-4xl w-full max-h-[80vh] overflow-y-auto border border-white/20">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-black">📊 Weather History</h2>
                <button 
                    on:click={() => showHistoricalData = false}
                    class="text-white/70 hover:text-white text-2xl"
                >×</button>
            </div>
            
            {#if weatherHistory.length === 0}
                <div class="text-center py-8">
                    <div class="text-6xl mb-4">📈</div>
                    <p class="text-white/70">No historical data yet. Search for cities to build history!</p>
                </div>
            {:else}
                <div class="space-y-4">
                    {#each weatherHistory as entry}
                        <div class="bg-white/10 rounded-2xl p-4 border border-white/20">
                            <div class="flex justify-between items-start mb-2">
                                <div>
                                    <h3 class="text-lg font-black">{entry.location}</h3>
                                    <p class="text-sm text-white/70">{entry.date}</p>
                                </div>
                                <div class="text-right">
                                    <div class="text-2xl font-black">{entry.temperature}°C</div>
                                    <p class="text-sm text-white/70">{entry.description}</p>
                                </div>
                            </div>
                            <div class="grid grid-cols-4 gap-4 text-sm">
                                <div>💧 {entry.humidity}%</div>
                                <div>💨 {entry.windSpeed} km/h</div>
                                <div>🔆 UV {entry.uvIndex}</div>
                                <div>👁️ {entry.visibility} km</div>
                            </div>
                        </div>
                    {/each}
                </div>
            {/if}
        </div>
    </div>
    {/if}

    <!-- Export Dialog -->
    {#if showExportDialog}
    <div class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4">
        <div class="bg-white/10 backdrop-blur-xl rounded-3xl p-8 max-w-md w-full border border-white/20">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-black">📄 Export Data</h2>
                <button 
                    on:click={() => showExportDialog = false}
                    class="text-white/70 hover:text-white text-2xl"
                >×</button>
            </div>
            
            <div class="space-y-4">
                <button 
                    on:click={() => exportWeatherData('json')}
                    class="w-full bg-blue-500 hover:bg-blue-600 p-4 rounded-2xl text-left transition-all"
                >
                    <div class="text-lg font-black">📊 JSON Format</div>
                    <div class="text-sm text-white/70">Complete data with forecast</div>
                </button>
                
                <button 
                    on:click={() => exportWeatherData('csv')}
                    class="w-full bg-green-500 hover:bg-green-600 p-4 rounded-2xl text-left transition-all"
                >
                    <div class="text-lg font-black">📈 CSV Format</div>
                    <div class="text-sm text-white/70">Spreadsheet compatible</div>
                </button>
            </div>
        </div>
    </div>
    {/if}



    <!-- Header -->
    <header class="relative z-10 p-6 pb-0">
        <!-- Tema ve Kontrol Paneli -->
        <div class="flex justify-between items-center mb-6">
            <div class="flex gap-2">
                <!-- Tema Butonları -->
                <button 
                    on:click={() => saveTheme('auto')}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all {currentTheme === 'auto' ? 'bg-white/20' : 'bg-white/5'}"
                    title="Auto Theme"
                >🌈</button>
                <button 
                    on:click={() => saveTheme('sunset')}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all {currentTheme === 'sunset' ? 'bg-white/20' : 'bg-white/5'}"
                    title="Sunset Theme"
                >🌅</button>
                <button 
                    on:click={() => saveTheme('ocean')}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all {currentTheme === 'ocean' ? 'bg-white/20' : 'bg-white/5'}"
                    title="Ocean Theme"
                >🌊</button>
                <button 
                    on:click={() => saveTheme('forest')}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all {currentTheme === 'forest' ? 'bg-white/20' : 'bg-white/5'}"
                    title="Forest Theme"
                >🌲</button>
                <button 
                    on:click={() => saveTheme('night')}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all {currentTheme === 'night' ? 'bg-white/20' : 'bg-white/5'}"
                    title="Night Theme"
                >🌙</button>
            </div>
            
            <div class="flex gap-2 flex-wrap">

                

                
                <!-- Weather Map -->
                <button 
                    on:click={() => { 
                        showWeatherMap = true; 
                        setTimeout(() => {
                            if (!leafletMap && mapContainer) {
                                initLeafletMap();
                            }
                        }, 500);
                    }}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all"
                    title="Weather Map"
                >🗺️</button>
                
                <!-- Favori Şehirler -->
                <button 
                    on:click={() => showFavorites = true}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all"
                    title="Favorite Cities"
                >⭐</button>
                
                <!-- Historical Data -->
                <button 
                    on:click={() => showHistoricalData = true}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all relative"
                    title="Weather History"
                >
                    📊
                    {#if weatherHistory.length > 0}
                        <span class="absolute -top-1 -right-1 bg-blue-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center">
                            {weatherHistory.length}
                        </span>
                    {/if}
                </button>
                
                <!-- Export Data -->
                <button 
                    on:click={() => showExportDialog = true}
                    class="p-2 rounded-xl backdrop-blur-xl border border-white/20 hover:bg-white/10 transition-all"
                    title="Export Data"
                >📄</button>
                

            </div>
        </div>

        <!-- Tarih ve Saat -->
        <div class="grid md:grid-cols-2 gap-4 mb-6">
            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                <div class="text-center">
                    <div class="text-3xl font-black mb-2">
                        {currentTime.toLocaleDateString('en-US', { 
                            weekday: 'long', 
                            year: 'numeric', 
                            month: 'long', 
                            day: 'numeric' 
                        })}
                    </div>
                    <div class="text-5xl font-black text-cyan-300">
                        {currentTime.toLocaleTimeString('en-US', { 
                            hour: '2-digit', 
                            minute: '2-digit',
                            hour12: false
                        })}
                    </div>
                </div>
            </div>
            
            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                <div class="text-center">
                    {#if weatherData}
                        <div class="text-xl font-black mb-2">📍 {weatherData.location}</div>
                        <div class="text-sm text-white/70 mb-1">Time Zone: {weatherData.timezone}</div>
                        <div class="text-sm text-white/70">Local Time: {weatherData.localTime}</div>
                    {:else}
                        <div class="text-xl font-black mb-2">📍 Loading...</div>
                        <div class="text-sm text-white/70">Getting location...</div>
                    {/if}
                </div>
            </div>
        </div>

        <!-- Arama Bölümü -->
        <div class="grid md:grid-cols-3 gap-4 mb-6">
            <!-- Ana Arama -->
            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-4 border border-white/20">
                <div class="flex gap-3">
                    <input 
                        type="text" 
                        bind:value={searchCity}
                        placeholder="Search city..."
                        class="flex-1 bg-transparent border-none outline-none text-white placeholder-white/50 font-semibold"
                        on:keydown={(e) => e.key === 'Enter' && handleSearch()}
                    />
                    <button 
                        on:click={handleSearch}
                        class="bg-purple-500 hover:bg-purple-600 px-4 py-2 rounded-xl text-sm font-semibold transition-all"
                    >🔍</button>
                </div>
            </div>
            
            <!-- Karşılaştırma Arama -->
            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-4 border border-white/20">
                <div class="flex gap-3">
                    <input 
                        type="text" 
                        bind:value={compareCity}
                        placeholder="Compare with..."
                        class="flex-1 bg-transparent border-none outline-none text-white placeholder-white/50 font-semibold"
                        on:keydown={(e) => e.key === 'Enter' && handleCompareSearch()}
                    />
                    <button 
                        on:click={handleCompareSearch}
                        class="bg-cyan-500 hover:bg-cyan-600 px-4 py-2 rounded-xl text-sm font-semibold transition-all"
                    >⚖️</button>
                </div>
            </div>
            
            <!-- Favori Ekleme -->
            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-4 border border-white/20">
                <button 
                    on:click={addToFavorites}
                    class="w-full bg-gradient-to-r from-purple-500 to-pink-500 hover:from-purple-600 hover:to-pink-600 px-4 py-2 rounded-xl text-sm font-semibold transition-all"
                    disabled={!weatherData}
                >
                    ⭐ Add to Favorites
                </button>
            </div>
        </div>

        <!-- Tab Navigation -->
        <div class="flex justify-center mb-6">
            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-2 border border-white/20">
                <div class="flex gap-2">
                    <button 
                        class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {activeTab === 'current' ? 'bg-purple-500 text-white' : 'text-white/70 hover:text-white hover:bg-white/10'}"
                        on:click={() => activeTab = 'current'}
                    >Current</button>
                    <button 
                        class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {activeTab === 'hourly' ? 'bg-purple-500 text-white' : 'text-white/70 hover:text-white hover:bg-white/10'}"
                        on:click={() => activeTab = 'hourly'}
                    >Hourly</button>
                    <button 
                        class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {activeTab === 'forecast' ? 'bg-purple-500 text-white' : 'text-white/70 hover:text-white hover:bg-white/10'}"
                        on:click={() => activeTab = 'forecast'}
                    >7-Day</button>
                    <button 
                        class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {activeTab === 'details' ? 'bg-purple-500 text-white' : 'text-white/70 hover:text-white hover:bg-white/10'}"
                        on:click={() => activeTab = 'details'}
                    >Details</button>
                    <button 
                        class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {activeTab === 'lifestyle' ? 'bg-purple-500 text-white' : 'text-white/70 hover:text-white hover:bg-white/10'}"
                        on:click={() => activeTab = 'lifestyle'}
                    >Lifestyle</button>
                    <button 
                        class="px-4 py-2 rounded-xl text-sm font-semibold transition-all {activeTab === 'advanced' ? 'bg-purple-500 text-white' : 'text-white/70 hover:text-white hover:bg-white/10'}"
                        on:click={() => activeTab = 'advanced'}
                    >Advanced</button>
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="relative z-10 px-6 pb-6">
        {#if loading}
            <div class="text-center py-20">
                <div class="animate-spin text-6xl mb-4">🌀</div>
                <div class="text-xl font-bold">Loading weather data...</div>
            </div>
        {:else if error}
            <div class="text-center py-20">
                <div class="text-6xl mb-4">⚠️</div>
                <div class="text-xl font-bold text-red-400">{error}</div>
            </div>
        {:else if weatherData}
            <!-- Current Weather Tab -->
            {#if activeTab === 'current'}
                <div class="max-w-4xl mx-auto">
                    <!-- Ana Hava Durumu Kartı -->
                    <div class="bg-white/10 backdrop-blur-xl rounded-3xl p-8 mb-6 border border-white/20 text-center">
                        <div class="text-9xl font-black mb-4 bg-gradient-to-r from-cyan-400 via-purple-400 to-pink-400 bg-clip-text text-transparent">
                            {weatherData.temperature}°C
                        </div>
                        <div class="text-3xl font-bold mb-2">{weatherData.description}</div>
                        <div class="text-xl text-white/70 mb-4">Feels like {weatherData.feelsLike}°C</div>
                        <div class="text-lg text-white/60">{weatherData.location}</div>
                    </div>

                    <!-- Hızlı Bilgiler -->
                    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20 text-center hover:scale-105 transition-transform">
                            <div class="text-3xl mb-2">💧</div>
                            <div class="text-2xl font-black">{weatherData.humidity}%</div>
                            <div class="text-sm text-white/70">Humidity</div>
                        </div>
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20 text-center hover:scale-105 transition-transform">
                            <div class="text-3xl mb-2">💨</div>
                            <div class="text-2xl font-black">{weatherData.windSpeed} km/h</div>
                            <div class="text-sm text-white/70">Wind</div>
                        </div>
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20 text-center hover:scale-105 transition-transform">
                            <div class="text-3xl mb-2">🌅</div>
                            <div class="text-2xl font-black">
                                {weatherData.sunrise}
                            </div>
                            <div class="text-sm text-white/70">Sunrise</div>
                        </div>
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20 text-center hover:scale-105 transition-transform">
                            <div class="text-3xl mb-2">🌇</div>
                            <div class="text-2xl font-black">
                                {weatherData.sunset}
                            </div>
                            <div class="text-sm text-white/70">Sunset</div>
                        </div>
                    </div>
                </div>
            {/if}

            <!-- Lifestyle Tab -->
            {#if activeTab === 'lifestyle'}
                <div class="max-w-4xl mx-auto">
                    <div class="grid md:grid-cols-2 gap-6">
                        <!-- Outfit Suggestion -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">👗 Outfit Recommendation</h3>
                            <div class="text-center">
                                <div class="text-4xl mb-4">👔</div>
                                <p class="text-lg">{weatherData.outfitSuggestion}</p>
                            </div>
                        </div>

                        <!-- Drive Conditions -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">🚗 Driving Conditions</h3>
                            <div class="text-center">
                                <div class="text-4xl mb-4">
                                    {weatherData.driveConditions.includes('Good') ? '✅' : 
                                     weatherData.driveConditions.includes('Moderate') ? '⚠️' : '❌'}
                                </div>
                                <p class="text-lg">{weatherData.driveConditions}</p>
                            </div>
                        </div>

                        <!-- Beach Conditions -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">🏖️ Beach Conditions</h3>
                            <div class="text-center">
                                <div class="text-4xl mb-4">🌊</div>
                                <p class="text-lg">Sea Temperature: {weatherData.beachTemp}</p>
                                <p class="text-sm text-white/70 mt-2">
                                    {parseInt(weatherData.beachTemp) > 20 ? 'Perfect for swimming!' : 
                                     parseInt(weatherData.beachTemp) > 15 ? 'A bit cool but manageable' : 
                                     'Too cold for swimming'}
                                </p>
                            </div>
                        </div>

                        <!-- Pollen Index -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">🌸 Pollen Index</h3>
                            <div class="text-center">
                                <div class="text-4xl mb-4">
                                    {weatherData.pollenIndex === 'High' ? '🤧' : 
                                     weatherData.pollenIndex === 'Medium' ? '😷' : '😊'}
                                </div>
                                <p class="text-lg">{weatherData.pollenIndex} Level</p>
                                <p class="text-sm text-white/70 mt-2">
                                    {weatherData.pollenIndex === 'High' ? 'Take allergy medication' : 
                                     weatherData.pollenIndex === 'Medium' ? 'Moderate risk for allergies' : 
                                     'Low allergy risk'}
                                </p>
                            </div>
                        </div>
                    </div>
                </div>
            {/if}

            <!-- Hourly Forecast Tab -->
            {#if activeTab === 'hourly'}
                <div class="max-w-6xl mx-auto">
                    <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                        <h2 class="text-2xl font-black mb-6 text-center">24-Hour Forecast</h2>
                        <div class="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-8 gap-4">
                            {#each hourlyData as hour}
                                <div class="text-center p-4 bg-white/10 rounded-xl border border-white/20 hover:scale-105 transition-transform">
                                    <div class="text-sm font-semibold text-white/70 mb-2">{hour.time}</div>
                                    <div class="text-2xl mb-2">{hour.icon}</div>
                                    <div class="text-xl font-black mb-1">{hour.temp}°C</div>
                                    <div class="text-xs text-white/60">{hour.precipitation}mm</div>
                                </div>
                            {/each}
                        </div>
                    </div>
                </div>
            {/if}

            <!-- 7-Day Forecast Tab -->
            {#if activeTab === 'forecast'}
                <div class="max-w-4xl mx-auto">
                    <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                        <h2 class="text-2xl font-black mb-6 text-center">7-Day Forecast</h2>
                        <div class="space-y-4">
                            {#each forecastData as day}
                                <div class="flex items-center justify-between p-4 bg-white/10 rounded-xl border border-white/20 hover:bg-white/20 transition-all">
                                    <div class="flex items-center gap-4">
                                        <div class="text-3xl">{day.icon}</div>
                                        <div>
                                            <div class="text-lg font-black">{day.date}</div>
                                            <div class="text-sm text-white/70">{day.description}</div>
                                        </div>
                                    </div>
                                    <div class="text-right">
                                        <div class="text-2xl font-black">{day.temp}°C</div>
                                        <div class="text-sm text-white/70">{day.tempMin}° / {day.tempMax}°</div>
                                    </div>
                                </div>
                            {/each}
                        </div>
                    </div>
                </div>
            {/if}

            <!-- Details Tab -->
            {#if activeTab === 'details'}
                <div class="max-w-4xl mx-auto">
                    <div class="grid md:grid-cols-2 gap-6">
                        <!-- Sol Kolon -->
                        <div class="space-y-6">
                            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                                <h3 class="text-xl font-black mb-4">🌡️ Temperature Details</h3>
                                <div class="space-y-3">
                                    <div class="flex justify-between">
                                        <span>Current:</span>
                                        <span class="font-bold">{weatherData.temperature}°C</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Feels Like:</span>
                                        <span class="font-bold">{weatherData.feelsLike}°C</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Dew Point:</span>
                                        <span class="font-bold">{weatherData.dewPoint}°C</span>
                                    </div>
                                </div>
                            </div>

                            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                                <h3 class="text-xl font-black mb-4">💨 Wind & Air</h3>
                                <div class="space-y-3">
                                    <div class="flex justify-between">
                                        <span>Wind Speed:</span>
                                        <span class="font-bold">{weatherData.windSpeed} km/h</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Direction:</span>
                                        <span class="font-bold">{getWindDirection(weatherData.windDirection)} {weatherData.windDirection}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Air Quality:</span>
                                        <span class="font-bold">{weatherData.airQuality}</span>
                                    </div>
                                </div>
                            </div>

                            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                                <h3 class="text-xl font-black mb-4">☀️ UV & Visibility</h3>
                                <div class="space-y-3">
                                    <div class="flex justify-between">
                                        <span>UV Index:</span>
                                        <span class="font-bold">{weatherData.uvIndex} ({getUVLevel(weatherData.uvIndex)})</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Visibility:</span>
                                        <span class="font-bold">{weatherData.visibility} km</span>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- Sağ Kolon -->
                        <div class="space-y-6">
                            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                                <h3 class="text-xl font-black mb-4">🌧️ Atmospheric</h3>
                                <div class="space-y-3">
                                    <div class="flex justify-between">
                                        <span>Humidity:</span>
                                        <span class="font-bold">{weatherData.humidity}%</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Pressure:</span>
                                        <span class="font-bold">{weatherData.pressure} hPa</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Cloud Cover:</span>
                                        <span class="font-bold">{weatherData.cloudCover}%</span>
                                    </div>
                                </div>
                            </div>

                            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                                <h3 class="text-xl font-black mb-4">🌙 Astronomy</h3>
                                <div class="space-y-3">
                                    <div class="flex justify-between">
                                        <span>Sunrise:</span>
                                        <span class="font-bold">{weatherData.sunrise}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Sunset:</span>
                                        <span class="font-bold">{weatherData.sunset}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Moon Phase:</span>
                                        <span class="font-bold">{weatherData.moonPhase}</span>
                                    </div>
                                </div>
                            </div>

                            <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                                <h3 class="text-xl font-black mb-4">🌍 Location</h3>
                                <div class="space-y-3">
                                    <div class="flex justify-between">
                                        <span>Location:</span>
                                        <span class="font-bold text-right">{weatherData.location}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Timezone:</span>
                                        <span class="font-bold">{weatherData.timezone}</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span>Local Time:</span>
                                        <span class="font-bold text-right">{weatherData.localTime}</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            {/if}

            <!-- Advanced Tab -->
            {#if activeTab === 'advanced'}
                <div class="max-w-6xl mx-auto">
                    <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
                        <!-- Air Quality Details -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">🌬️ Air Quality Index</h3>
                            <div class="text-center">
                                <div class="text-4xl mb-4">
                                    {weatherData?.airQuality === 'Excellent' ? '🟢' : 
                                     weatherData?.airQuality === 'Good' ? '🟡' : 
                                     weatherData?.airQuality === 'Moderate' ? '🟠' : '🔴'}
                                </div>
                                <p class="text-lg font-bold">{weatherData?.airQuality || 'Unknown'}</p>
                                <div class="bg-white/10 rounded-xl p-4 mt-4">
                                    <div class="text-sm space-y-2">
                                        <div class="flex justify-between">
                                            <span>PM2.5:</span>
                                            <span class="font-bold">
                                                {weatherData?.airQuality === 'Excellent' ? '5 μg/m³' : 
                                                 weatherData?.airQuality === 'Good' ? '15 μg/m³' : 
                                                 weatherData?.airQuality === 'Moderate' ? '35 μg/m³' : '55 μg/m³'}
                                            </span>
                                        </div>
                                        <div class="flex justify-between">
                                            <span>PM10:</span>
                                            <span class="font-bold">
                                                {weatherData?.airQuality === 'Excellent' ? '10 μg/m³' : 
                                                 weatherData?.airQuality === 'Good' ? '25 μg/m³' : 
                                                 weatherData?.airQuality === 'Moderate' ? '50 μg/m³' : '75 μg/m³'}
                                            </span>
                                        </div>
                                        <div class="flex justify-between">
                                            <span>O3:</span>
                                            <span class="font-bold">
                                                {weatherData?.airQuality === 'Excellent' ? '60 μg/m³' : 
                                                 weatherData?.airQuality === 'Good' ? '100 μg/m³' : 
                                                 weatherData?.airQuality === 'Moderate' ? '140 μg/m³' : '180 μg/m³'}
                                            </span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- UV Index Detailed -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">☀️ UV Protection Guide</h3>
                            <div class="text-center">
                                <div class="text-4xl mb-4">
                                    {getUVLevel(weatherData?.uvIndex || '0') === 'Low' ? '🟢' : 
                                     getUVLevel(weatherData?.uvIndex || '0') === 'Moderate' ? '🟡' : 
                                     getUVLevel(weatherData?.uvIndex || '0') === 'High' ? '🟠' : 
                                     getUVLevel(weatherData?.uvIndex || '0') === 'Very High' ? '🔴' : '🟣'}
                                </div>
                                <p class="text-lg font-bold">{getUVLevel(weatherData?.uvIndex || '0')}</p>
                                <p class="text-sm text-white/70 mb-4">Index: {weatherData?.uvIndex || '0'}</p>
                                <div class="bg-white/10 rounded-xl p-4">
                                    <p class="text-sm">
                                        {getUVLevel(weatherData?.uvIndex || '0') === 'Low' ? '👕 No protection needed. Safe to be outside.' : 
                                         getUVLevel(weatherData?.uvIndex || '0') === 'Moderate' ? '🧴 Apply SPF 15+ sunscreen. Wear sunglasses.' : 
                                         getUVLevel(weatherData?.uvIndex || '0') === 'High' ? '🧴 Apply SPF 30+ sunscreen. Wear hat and sunglasses.' : 
                                         getUVLevel(weatherData?.uvIndex || '0') === 'Very High' ? '🧴 Apply SPF 50+ sunscreen. Seek shade 10AM-4PM.' : 
                                         '🧴 Extreme caution! Avoid sun 10AM-4PM. Use SPF 50+.'}
                                    </p>
                                </div>
                            </div>
                        </div>

                        <!-- Astronomical Data -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">🌌 Astronomy</h3>
                            <div class="space-y-4">
                                <div class="flex justify-between items-center">
                                    <span>🌅 Sunrise</span>
                                    <span class="font-bold">{weatherData?.sunrise}</span>
                                </div>
                                <div class="flex justify-between items-center">
                                    <span>🌇 Sunset</span>
                                    <span class="font-bold">{weatherData?.sunset}</span>
                                </div>
                                <div class="flex justify-between items-center">
                                    <span>🌙 Moon Phase</span>
                                    <span class="font-bold">{weatherData?.moonPhase}</span>
                                </div>
                                <div class="bg-white/10 rounded-xl p-4">
                                    <div class="text-sm space-y-2">
                                        <div class="flex justify-between">
                                            <span>Day Length:</span>
                                            <span class="font-bold">
                                                {(() => {
                                                    if (!weatherData?.sunrise || !weatherData?.sunset) return '12h 00m';
                                                    const sunrise = new Date(`2024-01-01 ${weatherData.sunrise}`);
                                                    const sunset = new Date(`2024-01-01 ${weatherData.sunset}`);
                                                    const diff = sunset.getTime() - sunrise.getTime();
                                                    const hours = Math.floor(diff / (1000 * 60 * 60));
                                                    const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
                                                    return `${hours}h ${minutes}m`;
                                                })()}
                                            </span>
                                        </div>
                                        <div class="flex justify-between">
                                            <span>Golden Hour:</span>
                                            <span class="font-bold text-yellow-400">Now</span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- Weather Trends -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">📈 Weather Trends</h3>
                            <div class="space-y-4">
                                <div class="bg-white/10 rounded-xl p-4">
                                    <h4 class="font-bold mb-2">🌡️ Temperature Trend</h4>
                                    <div class="flex items-center justify-between text-sm">
                                        <span>Yesterday</span>
                                        <div class="flex items-center gap-2">
                                            <span>{parseInt(weatherData?.temperature || '20') - 2}°C</span>
                                            <span class="text-green-400">↗️</span>
                                        </div>
                                    </div>
                                </div>
                                <div class="bg-white/10 rounded-xl p-4">
                                    <h4 class="font-bold mb-2">💧 Humidity Trend</h4>
                                    <div class="flex items-center justify-between text-sm">
                                        <span>24h Change</span>
                                        <div class="flex items-center gap-2">
                                            <span>+5%</span>
                                            <span class="text-blue-400">↗️</span>
                                        </div>
                                    </div>
                                </div>
                                <div class="bg-white/10 rounded-xl p-4">
                                    <h4 class="font-bold mb-2">🌪️ Pressure Trend</h4>
                                    <div class="flex items-center justify-between text-sm">
                                        <span>Barometric</span>
                                        <div class="flex items-center gap-2">
                                            <span>Stable</span>
                                            <span class="text-gray-400">→</span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- Comfort Index -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">😌 Comfort Index</h3>
                            <div class="space-y-4">
                                <div class="text-center">
                                    <div class="text-4xl mb-2">
                                        {(() => {
                                            const temp = parseInt(weatherData?.temperature || '20');
                                            const humidity = parseInt(weatherData?.humidity || '50');
                                            if (temp >= 18 && temp <= 24 && humidity >= 40 && humidity <= 60) return '😊';
                                            if (temp >= 15 && temp <= 27 && humidity >= 30 && humidity <= 70) return '🙂';
                                            return '😐';
                                        })()}
                                    </div>
                                    <p class="text-lg font-bold">
                                        {(() => {
                                            const temp = parseInt(weatherData?.temperature || '20');
                                            const humidity = parseInt(weatherData?.humidity || '50');
                                            if (temp >= 18 && temp <= 24 && humidity >= 40 && humidity <= 60) return 'Excellent';
                                            if (temp >= 15 && temp <= 27 && humidity >= 30 && humidity <= 70) return 'Good';
                                            return 'Fair';
                                        })()}
                                    </p>
                                </div>
                                <div class="bg-white/10 rounded-xl p-4">
                                    <div class="text-sm space-y-2">
                                        <div class="flex justify-between">
                                            <span>Heat Index:</span>
                                            <span class="font-bold">{weatherData?.feelsLike}°C</span>
                                        </div>
                                        <div class="flex justify-between">
                                            <span>Wind Chill:</span>
                                            <span class="font-bold">
                                                {parseInt(weatherData?.temperature || '20') - Math.floor(parseInt(weatherData?.windSpeed || '0') / 5)}°C
                                            </span>
                                        </div>
                                        <div class="flex justify-between">
                                            <span>Comfort Score:</span>
                                            <span class="font-bold">
                                                {(() => {
                                                    const temp = parseInt(weatherData?.temperature || '20');
                                                    const humidity = parseInt(weatherData?.humidity || '50');
                                                    const wind = parseInt(weatherData?.windSpeed || '0');
                                                    let score = 50;
                                                    if (temp >= 18 && temp <= 24) score += 20;
                                                    if (humidity >= 40 && humidity <= 60) score += 20;
                                                    if (wind <= 15) score += 10;
                                                    return Math.min(100, score);
                                                })()}/100
                                            </span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- Activity Recommendations -->
                        <div class="bg-white/10 backdrop-blur-xl rounded-2xl p-6 border border-white/20">
                            <h3 class="text-xl font-black mb-4">🎯 Activity Guide</h3>
                            <div class="space-y-3">
                                {#each [
                                    { icon: '🏃‍♂️', activity: 'Running', condition: parseInt(weatherData?.temperature || '20') >= 10 && parseInt(weatherData?.temperature || '20') <= 25 },
                                    { icon: '🚴‍♂️', activity: 'Cycling', condition: parseInt(weatherData?.windSpeed || '0') <= 20 },
                                    { icon: '🏖️', activity: 'Beach Day', condition: parseInt(weatherData?.temperature || '20') >= 22 && !weatherData?.description.includes('rain') },
                                    { icon: '🥾', activity: 'Hiking', condition: parseInt(weatherData?.visibility || '10') >= 8 },
                                    { icon: '☔', activity: 'Indoor Activities', condition: weatherData?.description.includes('rain') || parseInt(weatherData?.temperature || '20') < 5 }
                                ] as activity}
                                    <div class="flex items-center justify-between p-3 bg-white/10 rounded-xl {activity.condition ? 'border-green-400/50' : 'border-red-400/50'} border">
                                        <div class="flex items-center gap-3">
                                            <span class="text-2xl">{activity.icon}</span>
                                            <span class="font-semibold">{activity.activity}</span>
                                        </div>
                                        <span class="text-sm {activity.condition ? 'text-green-400' : 'text-red-400'}">
                                            {activity.condition ? 'Recommended' : 'Not Ideal'}
                                        </span>
                                    </div>
                                {/each}
                            </div>
                        </div>
                    </div>
                </div>
            {/if}
        {/if}
    </main>
</div>

<style>
    :global(:root) {
        --bg-gradient: linear-gradient(to bottom right, #667eea, #764ba2, #ffeaa7);
    }
    
    :global(body) {
        background: var(--bg-gradient) !important;
        background-attachment: fixed !important;
        min-height: 100vh;
    }
    
    :global(.theme-sunset) {
        background: linear-gradient(to bottom right, #ff6b6b, #ff8e53, #ff6b9d) !important;
    }
    
    :global(.theme-ocean) {
        background: linear-gradient(to bottom right, #667eea, #764ba2, #89f7fe) !important;
    }
    
    :global(.theme-forest) {
        background: linear-gradient(to bottom right, #134e5e, #71b280, #38ef7d) !important;
    }
    
    :global(.theme-night) {
        background: linear-gradient(to bottom right, #000428, #004e92, #009ffd) !important;
    }
    
    /* Leaflet harita stilleri */
    :global(.leaflet-container) {
        height: 100% !important;
        width: 100% !important;
        border-radius: 0.75rem;
        background: #374151 !important;
    }
    
    :global(.leaflet-popup-content) {
        color: #000 !important;
        font-family: inherit;
    }
    
    :global(.leaflet-control-zoom) {
        border: none !important;
        border-radius: 0.5rem !important;
    }
    
    :global(.leaflet-control-zoom a) {
        background: rgba(255, 255, 255, 0.9) !important;
        color: #333 !important;
        border-radius: 0.25rem !important;
    }
    
  
    
    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(-10px); }
        to { opacity: 1; transform: translateY(0); }
    }
</style>
