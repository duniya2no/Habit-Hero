# HabitHero Backend Requirements

This document outlines all the backend components and features needed for your HabitHero app.

## 📋 Table of Contents

1. [Core Features](#core-features)
2. [Database Schema](#database-schema)
3. [API Endpoints](#api-endpoints)
4. [Authentication & Security](#authentication--security)
5. [Additional Features](#additional-features)
6. [Technology Stack Recommendations](#technology-stack-recommendations)
7. [Deployment Considerations](#deployment-considerations)

---

## 🎯 Core Features

### 1. **User Authentication**
- User registration (email, password, name)
- User login (email, password)
- JWT token-based authentication
- Password hashing (bcrypt/argon2)
- Token refresh mechanism
- Password reset functionality (optional)

### 2. **Habits Management**
- Create habits
- Read/Get all habits for a user
- Update habits
- Delete habits
- Archive/Unarchive habits
- User-specific habit isolation

### 3. **Completions Tracking**
- Log habit completions
- Remove completions
- Track completion history
- Date-based completion queries

### 4. **User Statistics**
- Current streak calculation
- Longest streak tracking
- Total completions count
- Completion rate calculation
- Level and experience points
- Achievements tracking

### 5. **Cloud Sync**
- Full sync (upload local + download remote)
- Conflict resolution (last-write-wins strategy)
- Offline queue processing
- Incremental sync support

---

## 🗄️ Database Schema

### Users Table
```sql
CREATE TABLE users (
  id VARCHAR(255) PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  is_premium BOOLEAN DEFAULT FALSE,
  premium_expires_at TIMESTAMP NULL
);
```

### Habits Table
```sql
CREATE TABLE habits (
  id VARCHAR(255) PRIMARY KEY,
  user_id VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  icon VARCHAR(10) NOT NULL,
  color VARCHAR(7) NOT NULL,
  target INTEGER NOT NULL,
  frequency VARCHAR(20) DEFAULT 'daily',
  time_unit VARCHAR(50) NOT NULL,
  reminder_enabled BOOLEAN DEFAULT FALSE,
  reminder_time VARCHAR(5),
  reminder_days JSON,
  created_at TIMESTAMP NOT NULL,
  is_archived BOOLEAN DEFAULT FALSE,
  category VARCHAR(50),
  difficulty VARCHAR(20),
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### Completions Table
```sql
CREATE TABLE completions (
  id VARCHAR(255) PRIMARY KEY,
  habit_id VARCHAR(255) NOT NULL,
  user_id VARCHAR(255) NOT NULL,
  date DATE NOT NULL,
  count INTEGER DEFAULT 1,
  time VARCHAR(8),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (habit_id) REFERENCES habits(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  UNIQUE(habit_id, date)
);
```

### User Stats Table
```sql
CREATE TABLE user_stats (
  user_id VARCHAR(255) PRIMARY KEY,
  current_streak INTEGER DEFAULT 0,
  longest_streak INTEGER DEFAULT 0,
  total_completions INTEGER DEFAULT 0,
  completion_rate DECIMAL(5,2) DEFAULT 0,
  level INTEGER DEFAULT 1,
  experience INTEGER DEFAULT 0,
  achievements JSON,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### Sync Queue Table (Optional - for server-side queue)
```sql
CREATE TABLE sync_queue (
  id VARCHAR(255) PRIMARY KEY,
  user_id VARCHAR(255) NOT NULL,
  type VARCHAR(50) NOT NULL,
  data JSON NOT NULL,
  timestamp BIGINT NOT NULL,
  processed BOOLEAN DEFAULT FALSE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

---

## 🔌 API Endpoints

### Authentication Endpoints

#### POST `/api/auth/register`
**Request:**
```json
{
  "email": "user@example.com",
  "password": "password123",
  "name": "John Doe"
}
```

**Response:**
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

#### POST `/api/auth/login`
**Request:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
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

### Habits Endpoints

#### GET `/api/habits`
**Headers:** `Authorization: Bearer <token>`

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "habit_123",
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
  ]
}
```

#### POST `/api/habits`
**Headers:** `Authorization: Bearer <token>`

**Request:** (Full Habit object)

**Response:**
```json
{
  "success": true,
  "data": { /* habit object */ }
}
```

#### PUT `/api/habits/:id`
**Headers:** `Authorization: Bearer <token>`

**Request:** (Updated Habit object)

**Response:**
```json
{
  "success": true,
  "data": { /* updated habit object */ }
}
```

#### DELETE `/api/habits/:id`
**Headers:** `Authorization: Bearer <token>`

**Response:**
```json
{
  "success": true
}
```

### Completions Endpoints

#### POST `/api/completions`
**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "habitId": "habit_123",
  "count": 1,
  "notes": "Feeling great!",
  "date": "2024-01-15T10:30:00Z"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "completion_123",
    "habitId": "habit_123",
    "date": "2024-01-15",
    "count": 1,
    "time": "10:30",
    "notes": "Feeling great!"
  }
}
```

#### DELETE `/api/completions/:habitId/:date`
**Headers:** `Authorization: Bearer <token>`

**Response:**
```json
{
  "success": true
}
```

### Stats Endpoints

#### GET `/api/stats`
**Headers:** `Authorization: Bearer <token>`

**Response:**
```json
{
  "success": true,
  "data": {
    "currentStreak": 7,
    "longestStreak": 30,
    "totalCompletions": 150,
    "completionRate": 85,
    "level": 5,
    "experience": 2500,
    "achievements": ["first_habit", "week_streak"]
  }
}
```

#### PUT `/api/stats`
**Headers:** `Authorization: Bearer <token>`

**Request:** (UserStats object)

**Response:**
```json
{
  "success": true,
  "data": { /* updated stats */ }
}
```

### Sync Endpoint

#### POST `/api/sync`
**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "habits": [ /* array of habits */ ],
  "stats": { /* user stats */ }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "habits": [ /* merged habits from server */ ],
    "stats": { /* merged stats from server */ }
  }
}
```

**Sync Logic:**
1. Receive local habits and stats
2. Fetch user's habits and stats from database
3. Merge using conflict resolution:
   - Habits: Use habit with more completions or newer timestamp
   - Stats: Use maximum values for streaks/completions, merge achievements
4. Save merged data to database
5. Return merged data to client

---

## 🔐 Authentication & Security

### JWT Token Structure
```json
{
  "userId": "user_123",
  "email": "user@example.com",
  "iat": 1234567890,
  "exp": 1234654290
}
```

### Security Requirements

1. **Password Security**
   - Hash passwords using bcrypt (cost factor 10+) or argon2
   - Never store plain text passwords
   - Enforce minimum password requirements (8+ chars, mixed case, numbers)

2. **Token Security**
   - Use secure JWT secret (32+ characters)
   - Set appropriate expiration (e.g., 7 days)
   - Implement token refresh mechanism
   - Store tokens securely on client

3. **API Security**
   - Use HTTPS only in production
   - Implement rate limiting (e.g., 100 requests/minute per user)
   - Validate all input data
   - Sanitize user inputs
   - Use parameterized queries to prevent SQL injection

4. **Authorization**
   - Verify user owns resources before allowing access
   - Check JWT token on every protected endpoint
   - Return 401 for invalid/expired tokens
   - Return 403 for unauthorized access

5. **Data Validation**
   - Validate email format
   - Validate habit data structure
   - Validate date formats
   - Enforce data constraints (e.g., target > 0)

---

## ✨ Additional Features

### 1. **Premium Features** (Optional)
- Premium subscription management
- Unlimited habits (free tier: 5 habits)
- Advanced analytics
- Custom themes
- Export/Import features
- Priority support

### 2. **Analytics & Insights** (Optional)
- Weekly/Monthly reports
- Habit completion trends
- Best performing habits
- Time-based analytics
- Category-based insights

### 3. **Social Features** (Optional)
- Share achievements
- Friend connections
- Leaderboards
- Habit challenges
- Community support

### 4. **Notifications** (Optional)
- Push notification service integration
- Email notifications
- Reminder notifications
- Achievement notifications
- Weekly recap emails

### 5. **Data Export/Import** (Optional)
- Export user data as JSON/CSV
- Import data from backup
- Data portability (GDPR compliance)

### 6. **Admin Dashboard** (Optional)
- User management
- Analytics dashboard
- System health monitoring
- Error logging and tracking

---

## 🛠️ Technology Stack Recommendations

### Backend Framework Options

#### Option 1: Node.js/Express
```javascript
// Pros: JavaScript/TypeScript, large ecosystem
// Cons: Single-threaded, requires careful async handling

Tech Stack:
- Express.js or Fastify
- TypeScript
- Prisma or TypeORM (ORM)
- PostgreSQL or MongoDB
- JWT (jsonwebtoken)
- bcrypt
```

#### Option 2: Python/FastAPI
```python
# Pros: Fast, modern, great for APIs
# Cons: Python ecosystem

Tech Stack:
- FastAPI
- SQLAlchemy or Tortoise ORM
- PostgreSQL
- PyJWT
- passlib (bcrypt)
```

#### Option 3: Go/Gin
```go
// Pros: Fast, concurrent, simple
// Cons: Smaller ecosystem

Tech Stack:
- Gin or Echo
- GORM
- PostgreSQL
- golang-jwt
- golang.org/x/crypto/bcrypt
```

### Database Options

1. **PostgreSQL** (Recommended)
   - Relational database
   - ACID compliance
   - JSON support for flexible data
   - Great for complex queries

2. **MongoDB**
   - NoSQL document database
   - Flexible schema
   - Good for rapid development
   - Native JSON support

3. **MySQL**
   - Traditional relational database
   - Widely supported
   - Good performance

### Recommended Stack (Production-Ready)

```
Backend: Node.js + Express + TypeScript
Database: PostgreSQL
ORM: Prisma
Authentication: JWT
Caching: Redis (optional)
File Storage: AWS S3 / Cloudinary (for profile images)
Hosting: AWS / DigitalOcean / Heroku / Railway
```

---

## 🚀 Deployment Considerations

### Environment Variables
```env
# Server
PORT=3000
NODE_ENV=production

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/habithero

# JWT
JWT_SECRET=your-super-secret-jwt-key-here-min-32-chars
JWT_EXPIRES_IN=7d

# CORS
CORS_ORIGIN=https://your-app-domain.com

# Email (optional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-password
```

### Production Checklist

- [ ] Use HTTPS only
- [ ] Set up database backups
- [ ] Implement rate limiting
- [ ] Set up error logging (Sentry, LogRocket)
- [ ] Configure CORS properly
- [ ] Set up monitoring (UptimeRobot, New Relic)
- [ ] Implement database connection pooling
- [ ] Set up CI/CD pipeline
- [ ] Configure environment variables securely
- [ ] Set up automated testing
- [ ] Implement API versioning
- [ ] Set up load balancing (if needed)

### Scaling Considerations

1. **Database**
   - Use connection pooling
   - Add database indexes on frequently queried fields
   - Consider read replicas for high traffic

2. **Caching**
   - Cache user stats
   - Cache frequently accessed habits
   - Use Redis for session storage

3. **CDN**
   - Serve static assets via CDN
   - Cache API responses where appropriate

---

## 📝 Implementation Priority

### Phase 1: Core Features (MVP)
1. ✅ User authentication (register/login)
2. ✅ Habits CRUD operations
3. ✅ Completions tracking
4. ✅ Basic stats calculation
5. ✅ Cloud sync

### Phase 2: Enhanced Features
1. Premium subscription system
2. Advanced analytics
3. Push notifications
4. Data export/import

### Phase 3: Social & Advanced
1. Social features
2. Admin dashboard
3. Advanced reporting
4. Machine learning insights

---

## 🔍 Testing Requirements

### Unit Tests
- Authentication logic
- Habit CRUD operations
- Stats calculations            
- Sync conflict resolution

### Integration Tests
- API endpoints
- Database operations
- Authentication flow

### E2E Tests
- User registration/login flow
- Habit creation and completion
- Sync functionality

---

## 📚 Additional Resources

- [JWT Best Practices](https://tools.ietf.org/html/rfc8725)
- [OWASP API Security](https://owasp.org/www-project-api-security/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [REST API Design Best Practices](https://restfulapi.net/)

---

## 💡 Quick Start Example (Node.js/Express)

```javascript
// server.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Middleware
app.use(helmet());
app.use(cors({ origin: process.env.CORS_ORIGIN }));
app.use(express.json());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/habits', require('./routes/habits'));
app.use('/api/completions', require('./routes/completions'));
app.use('/api/stats', require('./routes/stats'));
app.use('/api/sync', require('./routes/sync'));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

**Next Steps:**
1. Choose your technology stack
2. Set up database schema
3. Implement authentication
4. Build API endpoints
5. Test thoroughly
6. Deploy to production

Good luck building your HabitHero backend! 🚀
