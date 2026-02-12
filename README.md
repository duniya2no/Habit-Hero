# HabitHero 🦸

A beautifully designed, gamified habit tracking application that transforms routine building into an engaging, rewarding experience. HabitHero combines elegant design with psychological principles to help users build lasting positive habits through streaks, achievements, and visual progress tracking.

## 🚀 Features

### Core Functionality
- **Intelligent Habit Tracking**: Track daily habits with customizable targets and units
- **Streak System**: Maintain and track streaks with visual indicators
- **Progress Visualization**: Calendar heatmaps, progress bars, and weekly overviews
- **Gamification**: Achievements, levels, and experience points
- **Statistics & Analytics**: Comprehensive insights into your habit performance
- **Offline-First**: All data stored locally with AsyncStorage
- **Data Export/Import**: Backup and restore your progress

### Design Highlights
- **Modern UI**: Clean, intuitive interface with smooth animations
- **Customizable**: Choose icons, colors, and categories for each habit
- **Responsive**: Works seamlessly on iOS and Android
- **Accessible**: High contrast, readable typography

## 📱 Screens

1. **Home Dashboard**: Daily overview with quick habit completion
2. **Habit Detail**: In-depth view with calendar, statistics, and history
3. **Create/Edit Habit**: Intuitive form for setting up habits
4. **Statistics**: Visual progress tracking and achievements
5. **Profile**: Settings, data management, and app information

## 🛠️ Tech Stack

- **React Native** (Expo)
- **TypeScript**
- **Expo Router** (File-based routing)
- **AsyncStorage** (Local data persistence)
- **Day.js** (Date manipulation)
- **Victory Native** (Charts and visualizations)
- **React Native Paper** (UI components)

## 📦 Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd HabitHero/habit
```

2. Install dependencies:
```bash
npm install
```

3. Start the development server:
```bash
npx expo start
```

4. Run on your device:
   - Press `i` for iOS simulator
   - Press `a` for Android emulator
   - Scan QR code with Expo Go app on your device

## 🏗️ Project Structure

```
habit/
├── app/                    # Expo Router screens
│   ├── (tabs)/            # Tab navigation screens
│   │   ├── index.tsx      # Home dashboard
│   │   ├── statistics.tsx # Statistics screen
│   │   └── profile.tsx     # Profile & settings
│   └── habit/             # Habit-related screens
│       ├── [id].tsx       # Habit detail view
│       ├── create.tsx     # Create new habit
│       └── edit/[id].tsx  # Edit existing habit
├── components/            # Reusable UI components
│   ├── HabitCard.tsx
│   ├── ProgressBar.tsx
│   ├── StreakDisplay.tsx
│   └── CalendarHeatmap.tsx
├── context/               # React Context providers
│   └── HabitsContext.tsx
├── utils/                 # Utility functions
│   ├── storage.ts         # AsyncStorage helpers
│   ├── habitHelpers.ts   # Habit calculation logic
│   └── achievements.ts    # Achievement system
├── types/                 # TypeScript type definitions
│   └── index.ts
└── constants/             # Design system constants
    └── theme.ts
```

## 🎮 Usage

### Creating a Habit
1. Tap the floating action button (+) on the home screen
2. Enter habit name, select icon and color
3. Set daily target and unit (e.g., 8 glasses, 30 minutes)
4. Choose category and difficulty
5. Tap "Create Habit"

### Completing a Habit
- Quick complete: Tap the + button on any habit card
- Detailed log: Tap the habit card to open detail view, then tap "Mark Complete"

### Viewing Statistics
- Navigate to the Statistics tab to see:
  - Overall streaks and completion rates
  - Achievement progress
  - Level and experience points
  - Individual habit breakdowns

## 🎯 Gamification System

### Achievements
- **Getting Started**: Complete your first habit
- **Week Warrior**: Maintain a 7-day streak
- **Month Master**: Maintain a 30-day streak
- **Centurion**: Reach 100 total completions
- **Early Bird**: Complete a habit before 8 AM
- **Night Owl**: Complete a habit after 10 PM
- **Perfect Week**: Complete all habits every day for a week
- **Habit Master**: Create 10 habits

### Leveling System
- Gain experience points from completions and achievements
- Level up every 1000 XP
- Visual progress bar shows your journey

## 💾 Data Management

### Export Data
1. Go to Profile tab
2. Tap "Export Data"
3. Data is saved as JSON file

### Import Data
1. Go to Profile tab
2. Tap "Import Data"
3. Select your backup file

## 🔒 Privacy

- All data stored locally on your device
- No cloud sync (premium feature placeholder)
- No account required
- Complete control over your data

## 🚧 Roadmap

### Phase 1: MVP ✅
- Core habit tracking
- Basic UI with navigation
- Local storage
- Essential statistics

### Phase 2: Enhanced Features (In Progress)
- [ ] Advanced notification system
- [ ] Cloud backup and sync
- [ ] Widget support
- [ ] More chart visualizations

### Phase 3: Polish & Launch
- [ ] UI refinements and animations
- [ ] Monetization integration
- [ ] App store optimization
- [ ] Marketing materials

## 📄 License

This project is private and proprietary.

## 🤝 Contributing

This is a private project. For questions or suggestions, please contact the development team.

## 📞 Support

For issues or feature requests, please open an issue in the repository.

---

Built with ❤️ using React Native and Expo
