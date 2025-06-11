# Stickies App: Client-Side Routing

## Learning Objectives
By the end of this worksheet, you will:
- Understand how to handle client-side routing in a Single Page Application (SPA)
- Implement URL routing that serves your Vue app for specific URL patterns
- Extract URL parameters in your Vue application
- Test routing functionality

## Problem Statement
Currently, your stickies app works great when users create new stickies or visit the main page. However, if someone tries to visit a direct URL like `http://localhost:3000/abc12345` (where `abc12345` is a stickies ID), they get a 404 error instead of loading the stickies app.

**Why does this happen?**
- Your Express server only serves static files and API routes
- When someone visits `/abc12345`, Express doesn't know what to do with it
- The server returns a 404 instead of serving your Vue app

**What we want to achieve:**
- Direct URLs like `/abc12345` should load your Vue app
- The Vue app should detect the stickies ID from the URL
- Later, the Vue app can use this ID to load the specific stickies

## Part 1: Update server.js

### Step 1: Add the required import
Add this line at the top of your `server.js` file with your other imports:

```javascript
const path = require('path');
```
---

### Step 2: Add the client-side routing handler
Add this code **BEFORE** your `app.use(express.static(...))` line:

```javascript
// Handle client-side routing for stickies IDs
app.get('/:id', (req, res, next) => {
    const id = req.params.id;
    
    // Only handle if it looks like a stickies ID (8 alphanumeric chars)
    if (/^[a-z0-9]{8}$/.test(id)) {
        // Serve your main HTML file (Vue app)
        res.sendFile(path.join(__dirname, 'static', 'index.html'));
    } else {
        // Let static middleware handle other files (CSS, JS, etc.)
        next();
    }
});
```

---

### Step 3: Update your static middleware
Change this line:
```javascript
app.use(express.static('static'));
```

To this:
```javascript
app.use(express.static(path.join(__dirname, 'static')));
```

---

## Part 2: Update your Vue app

### Step 4: Add URL detection to your Vue app
In your Vue app's main component, add this code to detect the stickies ID from the URL. Add this to your component's `mounted()` lifecycle hook (or create one if it doesn't exist):

```javascript
mounted() {
    // Check if there's a stickies ID in the URL
    const path = window.location.pathname;
    const id = path.substring(1); // Remove the leading slash
    
    // Check if it looks like a valid stickies ID
    if (/^[a-z0-9]{8}$/.test(id)) {
        console.log('Found stickies ID in URL:', id);
        // TODO: Later we'll load the stickies here
    } else {
        console.log('No valid stickies ID in URL');
    }
}
```

## Part 3: Testing

### Step 5: Test your implementation

1. **Start your server** and make sure it's running without errors

2. **Test the main page:**
   - Visit `http://localhost:3000`
   - Open browser dev tools (F12) and check the Console tab
   - You should see: "No valid stickies ID in URL"

3. **Test with a fake stickies ID:**
   - Visit `http://localhost:3000/abc12345` 
   - Check the Console tab
   - You should see: "Found stickies ID in URL: abc12345"

4. **Test with an invalid ID:**
   - Visit `http://localhost:3000/invalid-id`

5. **Test static files still work:**
   - Make sure your CSS and JavaScript files still load properly
   - The Vue app should still function normally