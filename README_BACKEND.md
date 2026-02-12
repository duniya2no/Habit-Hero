# Backend Integration Guide

HabitHero now supports backend integration with offline-first architecture. This guide explains how to set up and use the backend features.

## Features

- ✅ **Offline-First**: Works completely offline, syncs when online
- ✅ **Authentication**: User login/registration
- ✅ **Cloud Sync**: Automatic and manual sync
- ✅ **Conflict Resolution**: Smart merging of local and remote data
- ✅ **Queue System**: Queues changes when offline, syncs when online

## Backend Setup

### 1. Update API Configuration

Edit `habit/services/api.ts` and update the `API_BASE_URL`:

```typescript
const API_BASE_URL = __DEV__ 
  ? 'http://localhost:3000/api' // Your development server
  : 'https://your-api-domain.com/api'; // Your production server
```

### 2. Required Backend Endpoints

Your backend needs to implement these endpoints:

#### Authentication
- `POST /api/auth/login` - Login
- `POST /api/auth/register` - Register

#### Habits
- `GET /api/habits` - Get all habits
- `POST /api/habits` - Create habit
- `PUT /api/habits/:id` - Update habit
- `DELETE /api/habits/:id` - Delete habit

#### Completions
- `POST /api/completions` - Log completion
- `DELETE /api/completions/:habitId/:date` - Remove completion

#### Stats
- `GET /api/stats` - Get user stats
- `PUT /api/stats` - Update stats

#### Sync
- `POST /api/sync` - Full sync (upload and download)

### 3. Backend API Examples

#### Login Request/Response
```json
// POST /api/auth/login
{
  "email": "user@example.com",
  "password": "password123"
}

// Response
{
  "token": "jwt_token_here",
  "user": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

#### Create Habit Request/Response
```json
// POST /api/habits
{
  "id": "habit_123456789",
  "name": "Drink Water",
  "icon": "💧",
  "color": "#3B82F6",
  "target": 8,
  "frequency": "daily",
  "timeUnit": "glasses",
  "completions": [],
  "reminder": {
    "enabled": true,
    "time": "08:00",
    "days": [1,2,3,4,5,6,7]
  },
  "createdAt": "2024-01-01T00:00:00Z",
  "isArchived": false,
  "category": "health",
  "difficulty": "easy"
}

// Response
{
  "success": true,
  "data": { /* habit object */ }
}
```

#### Sync Request/Response
```json
// POST /api/sync
{
  "habits": [ /* array of habits */ ],
  "stats": { /* user stats */ }
}

// Response
{
  "success": true,
  "data": {
    "habits": [ /* merged habits from server */ ],
    "stats": { /* merged stats from server */ }
  }
}
```

## Usage

### Authentication

The app automatically shows login screen if not authenticated. Users can:
- Login with email/password
- Register new account
- Logout from profile screen

### Automatic Sync

- Syncs every 5 minutes when authenticated
- Syncs on app launch if authenticated
- Queues changes when offline

### Manual Sync

Users can manually sync from Profile screen:
1. Go to Profile tab
2. Tap "Sync Now" in Cloud Sync section

### Offline Mode

- App works completely offline
- All changes are queued
- Syncs automatically when connection is restored

## Backend Implementation Examples

### Node.js/Express Example

```javascript
// routes/habits.js
const express = require('express');
const router = express.Router();

router.get('/', authenticate, async (req, res) => {
  const habits = await Habit.find({ userId: req.user.id });
  res.json(habits);
});

router.post('/', authenticate, async (req, res) => {
  const habit = new Habit({
    ...req.body,
    userId: req.user.id,
  });
  await habit.save();
  res.json(habit);
});

module.exports = router;
```

### Python/FastAPI Example

```python
# routes/habits.py
from fastapi import APIRouter, Depends
from models import Habit

router = APIRouter()

@router.get("/habits")
async def get_habits(user: User = Depends(get_current_user)):
    habits = await Habit.filter(user_id=user.id)
    return habits

@router.post("/habits")
async def create_habit(habit: HabitCreate, user: User = Depends(get_current_user)):
    habit_obj = Habit(**habit.dict(), user_id=user.id)
    await habit_obj.save()
    return habit_obj
```

## Security Considerations

1. **JWT Tokens**: Use secure JWT tokens for authentication
2. **HTTPS**: Always use HTTPS in production
3. **Input Validation**: Validate all inputs on backend
4. **Rate Limiting**: Implement rate limiting on API endpoints
5. **Data Encryption**: Encrypt sensitive data in transit and at rest

## Testing

1. Start your backend server
2. Update `API_BASE_URL` in `api.ts`
3. Run the app
4. Test login/registration
5. Create habits and verify sync
6. Test offline mode (disable network)
7. Re-enable network and verify sync

## Troubleshooting

### Sync not working
- Check API_BASE_URL is correct
- Verify backend is running
- Check network connectivity
- Review console logs for errors

### Authentication issues
- Verify JWT token is being sent
- Check token expiration
- Ensure backend validates tokens correctly

### Data conflicts
- The app uses last-write-wins strategy
- Habits with more completions take precedence
- Stats are merged (max values)

## Next Steps

1. Set up your backend server
2. Implement the required endpoints
3. Update API_BASE_URL
4. Test authentication
5. Test sync functionality
6. Deploy to production
