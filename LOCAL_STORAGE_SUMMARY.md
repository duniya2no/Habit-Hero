# Local Storage Summary

## Overview

All data in HabitHero is now stored locally on the device using AsyncStorage. The app works completely offline and does not require any backend server.

## Data Stored Locally

### 1. User Accounts
- **Storage Key**: `@habithero:users`
- **Data**: Array of all registered users
- **Structure**:
  ```typescript
  {
    id: string,
    email: string,
    name: string,
    password: string  // Plain text (for local only - will be hashed in backend)
  }[]
  ```

### 2. Current User Session
- **Storage Key**: `@habithero:user`
- **Data**: Currently logged-in user (without password)
- **Structure**:
  ```typescript
  {
    id: string,
    email: string,
    name: string
  }
  ```

### 3. Habits Data
- **Storage Key**: `@habithero:habits`
- **Data**: Array of all habits
- **Structure**: Full habit objects with completions, reminders, etc.
- **Size**: Can grow large with many completions

### 4. User Statistics
- **Storage Key**: `@habithero:userStats`
- **Data**: User performance metrics
- **Structure**:
  ```typescript
  {
    currentStreak: number,
    longestStreak: number,
    totalCompletions: number,
    completionRate: number,
    achievements: string[],
    level: number,
    experience: number
  }
  ```

### 5. Theme Preferences
- **Storage Key**: `@habithero:theme`
- **Data**: Theme mode preference
- **Structure**:
  ```typescript
  {
    mode: 'light' | 'dark' | 'auto'
  }
  ```

### 6. Premium Status
- **Storage Key**: `@habithero:premium`
- **Data**: Boolean indicating premium subscription
- **Type**: `boolean`

### 7. Profile Image
- **Storage Key**: `@habithero:profileImage`
- **Data**: URI/path to profile image
- **Type**: `string | null`

## Storage Functions

All storage operations are handled in `habit/utils/storage.ts`:

- `saveHabits()` - Save habits array
- `loadHabits()` - Load habits array
- `saveUserStats()` - Save user statistics
- `loadUserStats()` - Load user statistics
- `saveTheme()` - Save theme preference
- `loadTheme()` - Load theme preference
- `savePremiumStatus()` - Save premium status
- `loadPremiumStatus()` - Load premium status
- `saveProfileImage()` - Save profile image URI
- `loadProfileImage()` - Load profile image URI
- `exportData()` - Export all data as JSON
- `exportDataReadable()` - Export data as human-readable text
- `importData()` - Import data from JSON
- `clearAllData()` - Clear habits and stats (keeps theme/premium)

## Authentication (Local Only)

Authentication is handled in `habit/context/AuthContext.tsx`:

- **Registration**: Creates user account and stores in `@habithero:users`
- **Login**: Checks credentials against stored users
- **Logout**: Removes current session from `@habithero:user`

**Note**: Passwords are stored in plain text locally. This is for development only. When implementing backend, passwords MUST be hashed.

## Data Persistence

- All data persists across app restarts
- Data is stored on device storage (not in memory)
- Data survives app updates
- Data is cleared only when:
  - User explicitly clears data
  - App is uninstalled
  - Device storage is cleared

## Storage Limits

AsyncStorage has limits:
- **iOS**: ~50MB per app
- **Android**: ~10MB per app (can be increased)

For most users, this is sufficient. If you need more storage, consider:
- Using SQLite for larger datasets
- Implementing backend sync to offload data
- Compressing data before storage

## Backup & Export

Users can export their data:
- **JSON Format**: For backup/restore
- **Readable Format**: For viewing/printing

Data can be imported from JSON files.

## What Needs Backend Implementation

See `BACKEND_IMPLEMENTATION_GUIDE.md` for details on:
- User authentication with password hashing
- Cloud sync for multi-device support
- Data backup in cloud
- Conflict resolution
- Offline queue processing

## Current Implementation Status

✅ **Complete (Local Storage)**:
- User registration/login
- Habits CRUD operations
- Completion logging
- Statistics calculation
- Theme preferences
- Premium status
- Profile images
- Data export/import

⏳ **Future (Backend)**:
- Password hashing
- Cloud sync
- Multi-device support
- Data backup
- Conflict resolution
