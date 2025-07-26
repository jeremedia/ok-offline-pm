# OK-OFFLINE Ecosystem

An offline-first ecosystem for Burning Man participants, providing tools for navigation, weather monitoring, and playa wisdom sharing without requiring connectivity.

## 🌟 Project Overview

OK-OFFLINE is a multi-service ecosystem designed to enhance the Burning Man experience through offline-first technology. The system consists of a Vue.js Progressive Web App, Rails API services, and future components for community wisdom sharing.

## 🏗️ Current Architecture

```
OK-OFFLINE Ecosystem
├── Frontend (Vue.js PWA)           # Main user interface
│   ├── Offline camp/art/event browsing
│   ├── Interactive BRC maps  
│   ├── Personal schedule builder
│   ├── Weather & dust forecasting
│   └── Emergency information storage
├── API Services (Rails)            # Backend services
│   ├── Weather data aggregation
│   ├── Apple WeatherKit integration
│   ├── CORS-compliant endpoints
│   └── Future: Play Wisdom API
└── Future Services
    ├── Play Wisdom (photo/audio/text sharing)
    ├── Mobile companion app
    └── Admin dashboard
```

## 📊 Service Status

| Service | Status | Technology | Repository |
|---------|--------|------------|------------|
| **Frontend PWA** | ✅ Production | Vue 3 + Vite | [okoffline-2025](https://github.com/jeremedia/okoffline-2025) |
| **Weather API** | 🔧 In Development | Rails 8 API | [ok-offline-api](https://github.com/jeremedia/ok-offline-api) |
| **Play Wisdom** | 📋 Planned | TBD | TBD |
| **Mobile App** | 💭 Future | React Native / Flutter | TBD |

## 🚀 Current Features

### Frontend PWA
- **Offline-First Architecture** - Works completely offline once data is synced
- **Interactive Maps** - Leaflet-based maps with BRC coordinates
- **Smart Lists** - Sortable, filterable views of camps, art, and events
- **Personal Schedule** - Build your burn schedule with conflict detection
- **Favorites & Visits** - Track your favorite spots and places you've been
- **Emergency Features** - Store medical info and emergency contacts offline
- **Real-Time Weather** - Dust forecasts and conditions for Black Rock City
- **Global Search** - Search across all camps, art installations, and events

### Weather API (In Development)
- **Multiple Data Sources** - OpenWeatherMap + Apple WeatherKit integration
- **Moon Phase Data** - Essential for playa navigation at night
- **CORS Compliance** - Proper frontend-backend integration
- **Robust Fallbacks** - Multiple weather services for reliability

## 🗺️ Development Roadmap

### Phase 1: Weather Service Integration (Current)
- [x] Rails API service creation
- [x] Apple WeatherKit JWT authentication
- [x] Frontend weather service integration
- [ ] CORS configuration completion
- [ ] Production deployment

### Phase 2: Play Wisdom Feature (Next)
- [ ] API design for photo/audio/text uploads
- [ ] Offline-first sync strategy
- [ ] Community moderation system
- [ ] Mobile-optimized media capture

### Phase 3: Ecosystem Expansion (Future)
- [ ] React Native mobile app
- [ ] Admin dashboard for content management
- [ ] Real-time camp updates during event
- [ ] Community features and social integration

## 🛠️ Technology Stack

### Frontend
- **Vue 3** with Composition API
- **Vite** for build tooling and HMR
- **Vue Router 4** for client-side routing
- **Leaflet 1.9.3** for interactive maps
- **IndexedDB** for offline data storage
- **Service Workers** for PWA functionality

### Backend
- **Rails 8** in API mode
- **PostgreSQL** for data persistence
- **JWT** for authentication (Apple WeatherKit)
- **Rack CORS** for cross-origin requests

### Future Technologies
- **React Native/Flutter** for mobile apps
- **Redis** for caching and sessions
- **AWS S3** for media storage (Play Wisdom)
- **WebRTC** for real-time features

## 🔧 Development Setup

### Prerequisites
- Node.js 18+
- Ruby 3.2+
- PostgreSQL 14+
- Git

### Quick Start
```bash
# Clone the ecosystem
git clone https://github.com/jeremedia/okoffline-2025.git frontend
git clone https://github.com/jeremedia/ok-offline-api.git api

# Setup frontend
cd frontend
npm install
npm run dev  # Runs on port 8000

# Setup API (in another terminal)
cd api
bundle install
rails db:setup
rails server  # Runs on port 3000
```

## 📱 Play Wisdom Feature (Planned)

The Play Wisdom feature will enable burners to capture and share practical knowledge through:

- **Photo Documentation** - Visual guides for shade techniques, camp setups, etc.
- **Audio Recordings** - Capture conversations and wisdom from veterans
- **Text Notes** - Quick tips and insights
- **Offline Sync** - Capture content offline, sync when connectivity returns
- **Community Curation** - Moderation and quality control
- **Search & Discovery** - Find relevant wisdom by topic or location

## 🌐 Deployment

### Production URLs
- **Frontend PWA**: https://offline.oknotok.com
- **Weather API**: TBD (in development)

### CI/CD
- **Automatic deployment** on push to main branches
- **GitHub Actions** for version management
- **Semantic versioning** based on commit conventions

## 🤝 Contributing

This is an open-source project brought to you by Mr. OK of OKNOTOK. Contributions welcome!

### For Developers
1. Check the individual service repositories for specific setup instructions
2. Follow the roadmap for current development priorities
3. Use conventional commits for automatic versioning

### For Claude Code Sessions
- Use this repository as the working directory for cross-service development
- See `CLAUDE.md` for detailed development instructions
- Each service has its own Claude-specific documentation

## 📚 Documentation

- [**CLAUDE.md**](./CLAUDE.md) - Instructions for Claude Code development sessions
- [**ARCHITECTURE.md**](./ARCHITECTURE.md) - System architecture and design decisions
- [**ROADMAP.md**](./ROADMAP.md) - Detailed feature roadmap and priorities
- [**SERVICES.md**](./SERVICES.md) - Service-specific documentation
- [**API_DESIGN.md**](./docs/API_DESIGN.md) - Cross-service API specifications

## 📄 License

MIT License - See LICENSE file for details.

## 🙏 Acknowledgments

- **Data**: [Burning Man Public API](https://api.burningman.org)
- **Weather**: OpenWeatherMap & Apple WeatherKit
- **Maps**: Burning Man Innovate GIS
- **Created by**: Jeremy Roush
- **Brought to you by**: Mr. OK of OKNOTOK

---

*This project embodies the principles of radical self-reliance and gifting by providing essential tools for the playa community. Built with love for the Burning Man community. 🔥*
