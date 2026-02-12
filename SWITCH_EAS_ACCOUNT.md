# Switching EAS Account for Build

## Steps to Switch EAS Account:

### 1. Log out of current account:
```bash
cd habit
eas logout
```

### 2. Log in to new account:
```bash
eas login
```
(Enter credentials for the new account)

### 3. Option A: Use existing project (if you have access)
If the new account has access to project `ac65682b-64dd-4506-b50d-742fab074379`, 
you can keep the current projectId in app.json and build directly.

### 3. Option B: Create new project (recommended)
If you want a fresh project for the new account:

1. Initialize new EAS project:
```bash
cd habit
eas init
```
(This will create a new projectId)

2. Or manually update projectId in app.json after creating project on expo.dev

### 4. Build with new account:
```bash
eas build --platform android --profile preview
```

## Notes:
- The keystore will be different for the new account/project
- If you need the same keystore, you'll need to download and configure it manually
- Each EAS account has its own build limits
