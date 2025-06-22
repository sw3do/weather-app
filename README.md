# 🌤️ Modern Weather App

<div align="center">

![Weather App](https://img.shields.io/badge/Weather-App-blue?style=for-the-badge&logo=weatherapi)
![SvelteKit](https://img.shields.io/badge/SvelteKit-FF3E00?style=for-the-badge&logo=svelte&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)

**A beautiful, modern weather application built with SvelteKit, TypeScript, and TailwindCSS**

[📖 Documentation](#features) | [🐛 Report Bug](../../issues) | [💡 Request Feature](../../issues)

</div>

---

## ✨ Features

### 🌍 **Core Weather Features**
- **Real-time Weather Data** - Current conditions with automatic location detection
- **7-Day Forecast** - Extended weather predictions with detailed information
- **Hourly Forecast** - 24-hour weather timeline
- **Weather Details** - Temperature, humidity, wind speed, pressure, UV index
- **Air Quality Index** - Real-time air quality monitoring
- **Weather Alerts** - Automatic notifications for severe weather conditions

### 🎨 **Beautiful UI/UX**
- **Modern Design** - Ultra-modern interface with gradient backgrounds
- **Responsive Layout** - Perfect on desktop, tablet, and mobile devices
- **Smooth Animations** - AOS animations and hover effects
- **Dark Theme** - Eye-friendly dark interface
- **Dynamic Backgrounds** - Animated clouds, stars, and aurora effects

### 🌟 **Advanced Features**
- **5 Theme System** - Auto, Sunset, Ocean, Forest, Night themes
- **Favorite Cities** - Save and compare multiple locations
- **Interactive Maps** - Leaflet-powered weather maps
- **Weather History** - Track your search history
- **Data Export** - Export weather data in JSON/CSV formats
- **Lifestyle Recommendations** - Outfit suggestions and activity recommendations

### 📱 **Smart Features**
- **Auto Location** - Automatic geolocation detection
- **City Search** - Search weather for any city worldwide
- **Weather Comparison** - Compare weather between cities
- **Professional Data** - Heat index, dew point, moon phases
- **Activity Suggestions** - Running, cycling, beach conditions

---

## 🚀 Quick Start

### Prerequisites
- **Node.js** (v18 or higher)
- **Bun** (recommended) or npm/yarn

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/sw3do/weather-app.git
   cd weather-app
   ```

2. **Install dependencies**
   ```bash
   bun install
   # or
   npm install
   ```

3. **Start development server**
   ```bash
   bun dev
   # or
   npm run dev
   ```

4. **Open your browser**
   ```
   http://localhost:5173
   ```

---

## 🛠️ Tech Stack

| Technology | Purpose | Version |
|------------|---------|---------|
| **SvelteKit** | Frontend Framework | ^2.0.0 |
| **TypeScript** | Type Safety | ^5.0.0 |
| **TailwindCSS** | Styling | ^3.4.0 |
| **AOS** | Animations | ^2.3.4 |
| **Leaflet** | Interactive Maps | ^1.9.4 |
| **wttr.in API** | Weather Data | Free |

---

## 📦 Project Structure

```
weather-app/
├── src/
│   ├── routes/
│   │   ├── +layout.svelte      # Main layout with animations
│   │   └── +page.svelte        # Main weather page
│   ├── app.html                # HTML template
│   └── app.d.ts               # Type definitions
├── static/                     # Static assets
├── package.json               # Dependencies
├── tailwind.config.js         # Tailwind configuration
├── tsconfig.json             # TypeScript configuration
└── README.md                 # Project documentation
```

---

## 🎯 Key Components

### 🌡️ **Weather Display**
- Current temperature with feels-like
- Weather description and icons
- Humidity, wind speed, pressure
- Sunrise/sunset times
- UV index with safety levels

### 📊 **Forecast System**
- 7-day weather forecast
- Hourly predictions (24 hours)
- Min/max temperatures
- Precipitation data
- Weather trend analysis

### 🗺️ **Interactive Maps**
- Leaflet-powered weather maps
- Multiple weather layers
- Location markers with popups
- Weather overlay visualization

### 🎨 **Theme System**
- **Auto Theme**: Changes based on weather/time
- **Sunset Theme**: Warm orange/pink gradients
- **Ocean Theme**: Blue/cyan gradients
- **Forest Theme**: Green/teal gradients
- **Night Theme**: Dark blue gradients

---

## 🌐 API Integration

### Weather Data Source
- **Provider**: wttr.in (Free weather API)
- **Features**: No API key required
- **Coverage**: Global weather data
- **Format**: JSON response
- **Endpoints**: Current weather, forecasts, historical data

### Example API Usage
```javascript
// Current weather by coordinates
const response = await fetch(`https://wttr.in/${lat},${lon}?format=j1`);

// Weather by city name
const response = await fetch(`https://wttr.in/${city}?format=j1`);
```

---

## 🎨 Customization

### Adding New Themes
```javascript
// In the applyTheme function
case 'your-theme':
    gradient = 'linear-gradient(135deg, #color1 0%, #color2 50%, #color3 100%)';
    break;
```

### Custom Weather Icons
```javascript
// In the getWeatherIcon function
const weatherIcons = {
    'your-condition': '🌈',
    // Add more icons
};
```

---

## 📱 Responsive Design

The app is fully responsive and optimized for:
- **Desktop**: Full-featured experience with all panels
- **Tablet**: Optimized layout with touch interactions
- **Mobile**: Compact design with essential features
- **PWA Ready**: Can be installed as a mobile app

---

## 🔧 Development

### Available Scripts
```bash
# Development server
bun dev

# Build for production
bun run build

# Preview production build
bun run preview

# Type checking
bun run check

# Linting
bun run lint
```

### Environment Setup
```bash
# Install dependencies
bun install

# Add new dependency
bun add package-name

# Add dev dependency
bun add -d package-name
```

---

## 🚀 Deployment

### Vercel (Recommended)
```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel
```

### Netlify
```bash
# Build command
bun run build

# Publish directory
build
```

### Static Hosting
```bash
# Generate static files
bun run build

# Upload the 'build' folder to your hosting provider
```

---

## 🤝 Contributing

We welcome contributions! Here's how you can help:

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/amazing-feature
   ```
3. **Commit your changes**
   ```bash
   git commit -m 'Add amazing feature'
   ```
4. **Push to the branch**
   ```bash
   git push origin feature/amazing-feature
   ```
5. **Open a Pull Request**

### Development Guidelines
- Follow TypeScript best practices
- Use TailwindCSS for styling
- Write meaningful commit messages
- Test on multiple devices
- Maintain responsive design

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- **[wttr.in](https://wttr.in)** - Free weather API service
- **[SvelteKit](https://kit.svelte.dev)** - Amazing full-stack framework
- **[TailwindCSS](https://tailwindcss.com)** - Utility-first CSS framework
- **[Leaflet](https://leafletjs.com)** - Interactive maps library
- **[AOS](https://michalsnik.github.io/aos/)** - Animate On Scroll library

---

## 📞 Support

If you have any questions or need help:

- 📧 **Email**: [sw3doo@gmail.com](mailto:sw3doo@gmail.com)
- 🐛 **Issues**: [GitHub Issues](../../issues)
- 💬 **Discussions**: [GitHub Discussions](../../discussions)

---

<div align="center">

**Made with ❤️ by [Your Name]**

⭐ **Star this repo if you like it!** ⭐

</div>
