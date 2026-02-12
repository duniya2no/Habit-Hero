# Backend Implementation Guide

This document outlines what needs to be implemented in the backend when you're ready to add cloud sync and authentication features.

## Current State

**All data is currently stored locally on the device using AsyncStorage:**
- ✅ User accounts (email, password, name) - stored locally
- ✅ Habits data - stored locally
- ✅ User stats - stored locally
- ✅ Theme preferences - stored locally
- ✅ Premium status - stored locally
- ✅ Profile images - stored locally

**The app works completely offline and all data persists on the device.**

## What to Implement in Backend

### 1. Authentication System

#### User Registration
- **Endpoint**: `POST /api/auth/register`
- **Purpose**: Create new user account
- **Request Body**:
  ```json
  {
    "email": "user@example.com",
    "password": "hashed_password_here",
    "name": "John Doe"
  }
  ```
- **Response**:
  ```json
  {
    "success": true,
    "data": {
      "token": "jwt_token_here",
      "user": {
        "id": "user_123",
        "email": "user@example.com",
        "name": "John Doe"
      }
    }
  }
  ```
- **Notes**:
  - Hash passwords using bcrypt or similar (NEVER store plain passwords)
  - Validate email format
  - Check for duplicate emails
  - Generate JWT token for authentication

#### User Login
- **Endpoint**: `POST /api/auth/login`
- **Purpose**: Authenticate user and return token
- **Request Body**:
  ```json
  {
    "email": "user@example.com",
    "password": "plain_password_here"
  }
  ```
- **Response**:
  ```json
  {
    "success": true,
    "data": {
      "token": "jwt_token_here",
      "user": {
        "id": "user_123",
        "email": "user@example.com",
        "name": "John Doe"
      }
    }
  }
  ```
- **Notes**:
  - Verify password hash
  - Return JWT token
  - Token should expire (e.g., 7 days, 30 days)

#### Token Validation
- **Middleware**: Verify JWT token on protected routes
- **Purpose**: Ensure user is authenticated before accessing data

### 2. Habits Management

#### Get All Habits
- **Endpoint**: `GET /api/habits`
- **Headers**: `Authorization: Bearer <token>`
- **Response**:
  ```json
  {
    "success": true,
    "data": [
      {
        "id": "habit_123",
        "name": "Drink Water",
        "icon": "💧",
        "category": "health",
        "target": 8,
        "timeUnit": "glasses",
        "frequency": "daily",
        "difficulty": "easy",
        "color": "#3B82F6",
        "completions": [
          {
            "date": "2024-01-15",
            "count": 8,
            "time": "14:30",
            "notes": "Great day!"
          }
        ],
        "reminder": {
          "enabled": true,
          "time": "09:00"
        },
        "createdAt": "2024-01-01T00:00:00Z",
        "isArchived": false
      }
    ]
  }
  ```

#### Create Habit
- **Endpoint**: `POST /api/habits`
- **Headers**: `Authorization: Bearer <token>`
- **Request Body**: Full habit object (same structure as above)
- **Response**: Created habit object

#### Update Habit
- **Endpoint**: `PUT /api/habits/:id`
- **Headers**: `Authorization: Bearer <token>`
- **Request Body**: Updated habit object
- **Response**: Updated habit object
- **Notes**: Verify user owns the habit before updating

#### Delete Habit
- **Endpoint**: `DELETE /api/habits/:id`
- **Headers**: `Authorization: Bearer <token>`
- **Response**: `{ "success": true }`
- **Notes**: Verify user owns the habit before deleting

### 3. Completions Management

#### Log Completion
- **Endpoint**: `POST /api/completions`
- **Headers**: `Authorization: Bearer <token>`
- **Request Body**:
  ```json
  {
    "habitId": "habit_123",
    "count": 8,
    "date": "2024-01-15",
    "time": "14:30",
    "notes": "Great day!"
  }
  ```
- **Response**: Updated habit with new completion

#### Remove Completion
- **Endpoint**: `DELETE /api/completions/:habitId/:date`
- **Headers**: `Authorization: Bearer <token>`
- **Response**: `{ "success": true }`
- **Notes**: Verify user owns the habit

### 4. User Stats

#### Get User Stats
- **Endpoint**: `GET /api/stats`
- **Headers**: `Authorization: Bearer <token>`
- **Response**:
  ```json
  {
    "success": true,
    "data": {
      "currentStreak": 7,
      "longestStreak": 30,
      "totalCompletions": 150,
      "completionRate": 85,
      "achievements": ["first_completion", "7_day_streak"],
      "level": 5,
      "experience": 4500
    }
  }
  ```

#### Update User Stats
- **Endpoint**: `PUT /api/stats`
- **Headers**: `Authorization: Bearer <token>`
- **Request Body**: Stats object
- **Response**: Updated stats object

### 5. Sync System

#### Full Sync
- **Endpoint**: `POST /api/sync`
- **Headers**: `Authorization: Bearer <token>`
- **Purpose**: Upload local changes and download remote changes
- **Request Body**:
  ```json
  {
    "habits": [...all habits...],
    "stats": {...user stats...},
    "lastSync": "2024-01-15T10:00:00Z"
  }
  ```
- **Response**:
  ```json
  {
    "success": true,
    "data": {
      "habits": [...merged habits...],
      "stats": {...merged stats...},
      "conflicts": []
    }
  }
  ```
- **Notes**:
  - Implement conflict resolution (last-write-wins or merge strategy)
  - Compare timestamps to determine which data is newer
  - Return conflicts if manual resolution is needed

### 6. Database Schema

#### Users Table
```sql
CREATE TABLE users (
  id VARCHAR(255) PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Habits Table
```sql
CREATE TABLE habits (
  id VARCHAR(255) PRIMARY KEY,
  user_id VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  icon VARCHAR(10),
  category VARCHAR(50),
  target INTEGER,
  time_unit VARCHAR(50),
  frequency VARCHAR(50),
  difficulty VARCHAR(50),
  color VARCHAR(7),
  reminder_enabled BOOLEAN DEFAULT false,
  reminder_time VARCHAR(10),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  is_archived BOOLEAN DEFAULT false,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

#### Completions Table
```sql
CREATE TABLE completions (
  id VARCHAR(255) PRIMARY KEY,
  habit_id VARCHAR(255) NOT NULL,
  date DATE NOT NULL,
  count INTEGER DEFAULT 1,
  time VARCHAR(10),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (habit_id) REFERENCES habits(id) ON DELETE CASCADE,
  UNIQUE(habit_id, date)
);
```

#### User Stats Table
```sql
CREATE TABLE user_stats (
  user_id VARCHAR(255) PRIMARY KEY,
  current_streak INTEGER DEFAULT 0,
  longest_streak INTEGER DEFAULT 0,
  total_completions INTEGER DEFAULT 0,
  completion_rate INTEGER DEFAULT 0,
  achievements JSON,
  level INTEGER DEFAULT 1,
  experience INTEGER DEFAULT 0,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 7. Security Considerations

1. **Password Hashing**: Use bcrypt with salt rounds (minimum 10)
2. **JWT Tokens**: 
   - Use secure secret key
   - Set appropriate expiration times
   - Implement token refresh mechanism
3. **Input Validation**: Validate all user inputs
4. **SQL Injection**: Use parameterized queries
5. **CORS**: Configure CORS properly for your domain
6. **Rate Limiting**: Implement rate limiting on auth endpoints
7. **HTTPS**: Always use HTTPS in production

### 8. Migration Strategy

When implementing the backend:

1. **Phase 1**: Set up authentication endpoints
   - Update `AuthContext` to use API instead of local storage
   - Keep local storage as fallback

2. **Phase 2**: Implement habits endpoints
   - Update `HabitsContext` to sync with backend
   - Keep local storage for offline support

3. **Phase 3**: Implement sync system
   - Add sync service back
   - Handle conflicts and offline queue

4. **Phase 4**: Add data migration
   - Allow users to upload their local data to cloud
   - Merge local and cloud data

### 9. Testing Checklist

- [ ] User registration with valid/invalid data
- [ ] User login with correct/incorrect credentials
- [ ] Token validation on protected routes
- [ ] CRUD operations for habits
- [ ] Completion logging and removal
- [ ] Stats calculation and updates
- [ ] Sync with conflict resolution
- [ ] Offline queue processing
- [ ] Data migration from local to cloud

### 10. API Base URL Configuration

When ready to implement, update `habit/services/api.ts`:

```typescript
const API_BASE_URL = __DEV__ 
  ? 'http://localhost:3000/api' // Development
  : 'https://your-api-domain.com/api'; // Production
```

## Current Local Storage Keys

All data is stored in AsyncStorage with these keys:
- `@habithero:users` - All registered users (local only)
- `@habithero:user` - Current logged-in user session
- `@habithero:habits` - All habits data
- `@habithero:userStats` - User statistics
- `@habithero:theme` - Theme preferences
- `@habithero:premium` - Premium status
- `@habithero:profileImage` - Profile image URI

## Notes

- **Passwords**: Currently stored in plain text locally (for development only). Backend MUST hash passwords.
- **Security**: Local storage is not encrypted. For production, consider encrypting sensitive data.
- **Offline Support**: App works completely offline. Backend should support offline-first architecture.
- **Data Sync**: When implementing backend, use timestamp-based conflict resolution.
