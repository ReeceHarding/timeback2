# TimeBack Backend - 30 Second Setup

## 🚀 Start Everything (Zero Passwords)

```bash
cd backend
./dev-start.sh
```

**That's it!** Everything starts automatically.

## 🎯 What You Get

- **API Server**: http://localhost:3001
- **Database Admin**: http://localhost:5050 
  - Login: `admin@timeback.com` / `admin`

## ⚡ Essential Commands

```bash
# Start everything
./dev-start.sh

# Test if working
curl http://localhost:3001/health

# Stop everything  
./stop.sh

# Fresh restart
./reset.sh && ./dev-start.sh
```

## 🧪 Quick API Test

```bash
# Health check
curl http://localhost:3001/health

# Get student archetypes
curl http://localhost:3001/test/archetype/available-grades

# Test database
curl http://localhost:3001/test/database/map-data
```

## ❗ Common Issues

**Getting "npm error Missing script"?**
→ You're in wrong directory. Run: `cd backend`

**Nothing responding on port 3001?**
→ Run: `./dev-start.sh` (it handles everything)

**Want to start fresh?**
→ Run: `./reset.sh && ./dev-start.sh`

---
**That's literally everything you need to know!** 