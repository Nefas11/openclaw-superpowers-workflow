# Implementation Plan: Health Check Endpoint

**Date:** 2026-02-17  
**Feature:** Add Health Check Endpoint  
**Design Doc:** `docs/design/2026-02-17-health-check.md`

---

## Task Overview

| Task | Description | Estimated Time |
|------|-------------|----------------|
| Task 1 | Create Express.js server with /health route | 5 min |
| Task 2 | Add health check logic (timestamp, uptime) | 5 min |
| Task 3 | Add Jest tests for endpoint | 5 min |

---

## Task 1: Create Express.js Server with /health Route

### Files to Create/Modify
- **Create:** `projects/health-check-demo/package.json`
- **Create:** `projects/health-check-demo/src/server.js`

### Implementation Details

**package.json:**
```json
{
  "name": "health-check-demo",
  "version": "1.0.0",
  "description": "Health Check Endpoint Demo",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "supertest": "^6.3.3"
  }
}
```

**src/server.js (initial):**
```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

// Start server
const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = { app, server };
```

### Verification Command
```bash
cd projects/health-check-demo
npm install
npm start &
sleep 2
curl http://localhost:3000/health
# Expected: {"status":"ok"}
```

---

## Task 2: Add Health Check Logic

### Files to Modify
- **Modify:** `projects/health-check-demo/src/server.js`

### Implementation Details

Add start time tracking and full health check response:

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// Track server start time
const startTime = Date.now();

// Health check endpoint
app.get('/health', (req, res) => {
  try {
    const uptime = (Date.now() - startTime) / 1000;
    const response = {
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: uptime
    };
    res.status(200).json(response);
  } catch (error) {
    res.status(503).json({
      status: 'error',
      timestamp: new Date().toISOString(),
      uptime: (Date.now() - startTime) / 1000
    });
  }
});

// Start server
const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = { app, server };
```

### Verification Command
```bash
curl http://localhost:3000/health
# Expected: {"status":"ok","timestamp":"2026-02-17T...","uptime":1.234}
```

---

## Task 3: Add Jest Tests

### Files to Create
- **Create:** `projects/health-check-demo/tests/health.test.js`

### Implementation Details

**tests/health.test.js:**
```javascript
const request = require('supertest');
const { app, server } = require('../src/server');

describe('Health Check Endpoint', () => {
  afterAll(() => {
    server.close();
  });

  test('GET /health should return 200 status', async () => {
    const response = await request(app)
      .get('/health')
      .expect(200);
    
    expect(response.body.status).toBe('ok');
  });

  test('GET /health should return JSON with required fields', async () => {
    const response = await request(app)
      .get('/health')
      .expect('Content-Type', /json/)
      .expect(200);
    
    expect(response.body).toHaveProperty('status');
    expect(response.body).toHaveProperty('timestamp');
    expect(response.body).toHaveProperty('uptime');
    expect(response.body.status).toBe('ok');
  });

  test('GET /health timestamp should be valid ISO 8601', async () => {
    const response = await request(app)
      .get('/health')
      .expect(200);
    
    const timestamp = response.body.timestamp;
    expect(new Date(timestamp).toISOString()).toBe(timestamp);
  });

  test('GET /health uptime should be positive number', async () => {
    const response = await request(app)
      .get('/health')
      .expect(200);
    
    expect(response.body.uptime).toBeGreaterThan(0);
    expect(typeof response.body.uptime).toBe('number');
  });
});
```

### Verification Command
```bash
npm test
# Expected: All 4 tests pass
```

---

## Implementation Order

1. **Task 1** → Create server structure and basic route
2. **Task 2** → Add health check logic with timestamp/uptime
3. **Task 3** → Add comprehensive tests

## Final Verification

After all tasks complete:
```bash
npm test          # All tests pass
curl localhost:3000/health
# Expected: {"status":"ok","timestamp":"...","uptime":...}
```
