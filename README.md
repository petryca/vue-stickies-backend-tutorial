# Building a Sticky Notes API - Step by Step Tutorial

Learn to build a complete REST API for managing sticky notes collections by coding one function at a time!

## Prerequisites
- Node.js installed on your computer
- Basic understanding of JavaScript
- A text editor (VS Code recommended)
- A terminal/command prompt

## Setup

### Step 1: Initialize Your Project
```bash
mkdir sticky-notes-api
cd sticky-notes-api
npm init -y
npm install express
```

### Step 2: Create Basic Server Structure
Create `server.js` and start with this foundation:

```javascript
const express = require('express');

const app = express();
const PORT = 3000;

// Start server
app.listen(PORT, () => {
    console.log(`Anonymous Stickies API Server running on http://localhost:${PORT}`);
});

module.exports = app;
```

**Test it:** Run `node server.js` - you should see the server message!

---

## Step 3: Add JSON Middleware

**What it does:** Enables your server to parse JSON data from incoming requests.

**Add this code after `const PORT = 3000;`:**
```javascript
// Middleware to parse JSON
app.use(express.json());
```

**Why we need it:** Without this, your server can't understand JSON data sent in POST/PUT requests.

**Test it:** Server should still start normally with no errors.

---

## Step 4: Add Static File Serving

**What it does:** Serves HTML, CSS, JS files from a "static" folder so users can access a web interface.

**Add this code after the JSON middleware:**
```javascript
// Serve static files from the 'static' folder
app.use(express.static('static'));
```

**Create the static folder and test file:**
```bash
mkdir static
```

Create `static/index.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Sticky Notes API</title>
</head>
<body>
    <h1>Welcome to Sticky Notes API!</h1>
    <p>The API is running successfully.</p>
</body>
</html>
```

**Test it:** 
1. Run `node server.js`
2. Visit `http://localhost:3000` in your browser
3. You should see your HTML page!

---

## Step 5: Create Data Storage

**What it does:** Creates an in-memory array to store all sticky note collections.

**Add this code after the middleware:**
```javascript
// In-memory storage for all sticky sets
const stickiesData = [];
```

**Why we need it:** This array will hold all our sticky note collections. Each collection has an ID and contains multiple sticky notes.

**Test it:** Server should start normally. The array is empty but ready to store data.

---

## Step 6: Add ID Generation Helper

**What it does:** Generates random 8-character IDs for each sticky notes collection.

**Add this helper function:**
```javascript
// Helper function to generate random stickies ID (8 characters)
function generateStickiesId() {
    const chars = 'abcdefghijklmnopqrstuvwxyz0123456789';
    let result = '';
    for (let i = 0; i < 8; i++) {
        result += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return result;
}
```

**How it works:** 
- Uses only lowercase letters and numbers
- Creates 8-character strings like "a5k9m2x1"
- Each ID should be unique

**Test it:** Add a temporary test in your server startup:
```javascript
app.listen(PORT, () => {
    console.log(`Anonymous Stickies API Server running on http://localhost:${PORT}`);
    console.log('Test ID:', generateStickiesId()); // Remove this after testing
});
```

You should see a random ID printed when the server starts!

---

## Step 7: Add Search Helper Function

**What it does:** Finds a sticky collection by its ID in our data array.

**Add this helper function:**
```javascript
// Helper function to find sticky set by ID
function findStickySetById(id) {
    return stickiesData.find(stickySet => stickySet.id === id);
}
```

**How it works:** Uses JavaScript's `find()` method to search the array for a collection with matching ID.

**Test it:** We'll test this after we have some data to search through.

---

## Step 8: Add Access Time Helper

**What it does:** Updates the "last accessed" timestamp when someone views a sticky collection.

**Add this helper function:**
```javascript
// Helper function to update sticky set last accessed time
function updateStickySetAccess(stickySet) {
    stickySet.lastAccessed = new Date().toISOString();
}
```

**Why we need it:** Helps track which collections are being used, useful for statistics.

---

## Step 9: Create New Stickies (POST Route)

**What it does:** Creates a new sticky notes collection with a unique ID.

**Add this route:**
```javascript
// 1. Create new stickies with provided stickies array (only way to create new stickies)
app.post('/api/', (req, res) => {
    const stickies = req.body;

    if (!Array.isArray(stickies)) {
        return res.status(400).json({ error: 'Stickies must be an array' });
    }

    if (stickies.length === 0) {
        return res.status(400).json({ error: 'Stickies array cannot be empty' });
    }

    let id;
    // Ensure unique stickies ID
    do {
        id = generateStickiesId();
    } while (findStickySetById(id));

    const newStickySet = {
        id: id,
        stickies: stickies,
        createdAt: new Date().toISOString(),
        lastAccessed: new Date().toISOString()
    };

    stickiesData.push(newStickySet);
    res.status(201).json(newStickySet);
});
```

**How it works:**
1. Gets the sticky notes array from request body
2. Validates it's an array and not empty
3. Generates a unique ID
4. Creates a new collection object with timestamps
5. Saves it to our data array
6. Returns the new collection

**Test it with curl:**
```bash
curl -X POST http://localhost:3000/api/ \
  -H "Content-Type: application/json" \
  -d '[{"id":1,"text":"My first note","color":"yellow","x":100,"y":200}]'
```

**Expected response:**
```json
{
  "id": "a5k9m2x1",
  "stickies": [{"id":1,"text":"My first note","color":"yellow","x":100,"y":200}],
  "createdAt": "2025-06-10T10:30:00.000Z",
  "lastAccessed": "2025-06-10T10:30:00.000Z"
}
```

**Test error cases:**
```bash
# Should return error - not an array
curl -X POST http://localhost:3000/api/ \
  -H "Content-Type: application/json" \
  -d '{"invalid": "data"}'

# Should return error - empty array
curl -X POST http://localhost:3000/api/ \
  -H "Content-Type: application/json" \
  -d '[]'
```

---

## Step 10: Get Stickies by ID (GET Route)

**What it does:** Retrieves a sticky notes collection by its ID.

**Add this route:**
```javascript
// 2. Stickies endpoint - serve stickies or return 404 if not exists
app.get('/api/:id', (req, res) => {
    const id = req.params.id;

    // Validate stickies ID format (8 alphanumeric characters)
    if (!/^[a-z0-9]{8}$/.test(id)) {
        return res.status(400).json({ error: 'Invalid stickies ID format' });
    }

    const stickySet = findStickySetById(id);

    if (!stickySet) {
        return res.status(404).json({ error: 'Stickies not found' });
    }

    updateStickySetAccess(stickySet);

    // Return complete stickies data
    res.json(stickySet);
});
```

**How it works:**
1. Gets the ID from the URL parameter
2. Validates the ID format (8 characters, letters and numbers only)
3. Searches for the collection
4. Updates access time if found
5. Returns the collection or error

**Test it:**
1. First create a collection (use the POST request from Step 9)
2. Note the ID returned (e.g., "a5k9m2x1")
3. Get that collection:

```bash
curl -X GET http://localhost:3000/api/a5k9m2x1
```

**Test error cases:**
```bash
# Invalid ID format
curl -X GET http://localhost:3000/api/invalid

# Non-existent ID
curl -X GET http://localhost:3000/api/zzzzzzzz
```

---

## Step 11: Update Stickies (PUT Route)

**What it does:** Replaces all sticky notes in a collection with new ones.

**Add this route:**
```javascript
// 3. Update entire stickies (replace all stickies)
app.put('/api/:id', (req, res) => {
    const id = req.params.id;
    const stickies = req.body;

    if (!Array.isArray(stickies)) {
        return res.status(400).json({ error: 'Stickies must be an array' });
    }

    if (stickies.length === 0) {
        return res.status(400).json({ error: 'Stickies array cannot be empty. Use DELETE to remove stickies.' });
    }

    const stickySet = findStickySetById(id);
    if (!stickySet) {
        return res.status(404).json({ error: 'Stickies not found' });
    }

    // Replace all stickies with the provided array
    stickySet.stickies = stickies;
    stickySet.lastAccessed = new Date().toISOString();

    res.json(stickySet);
});
```

**How it works:**
1. Gets ID and new stickies array
2. Validates the array format and ensures it's not empty
3. Finds the existing collection
4. Replaces all stickies and updates timestamp

**Test it:**
```bash
# Update with new stickies
curl -X PUT http://localhost:3000/api/a5k9m2x1 \
  -H "Content-Type: application/json" \
  -d '[{"id":1,"text":"Updated note","color":"blue","x":150,"y":250}, {"id":2,"text":"Second note","color":"green","x":300,"y":100}]'

# Try to update with empty array (should return error)
curl -X PUT http://localhost:3000/api/a5k9m2x1 \
  -H "Content-Type: application/json" \
  -d '[]'
```

---

## Step 12: Delete Stickies (DELETE Route)

**What it does:** Completely deletes a sticky notes collection by its ID.

**Add this route:**
```javascript
// 4. Delete entire stickies collection
app.delete('/api/:id', (req, res) => {
    const id = req.params.id;

    // Validate stickies ID format (8 alphanumeric characters)
    if (!/^[a-z0-9]{8}$/.test(id)) {
        return res.status(400).json({ error: 'Invalid stickies ID format' });
    }

    const stickySetIndex = stickiesData.findIndex(set => set.id === id);
    
    if (stickySetIndex === -1) {
        return res.status(404).json({ error: 'Stickies not found' });
    }

    const deletedStickySet = stickiesData.splice(stickySetIndex, 1)[0];

    res.json({
        message: 'Stickies deleted successfully',
        id: deletedStickySet.id,
        stickiesCount: deletedStickySet.stickies.length
    });
});
```

**How it works:**
1. Gets the ID from the URL parameter
2. Validates the ID format
3. Finds the collection index in the array
4. Removes it from the array using `splice()`
5. Returns confirmation with details about what was deleted

**Test it:**
```bash
# Delete a sticky collection
curl -X DELETE http://localhost:3000/api/a5k9m2x1

# Expected response:
# {
#   "message": "Stickies deleted successfully",
#   "id": "a5k9m2x1",
#   "stickiesCount": 2
# }

# Try to delete the same collection again (should return 404)
curl -X DELETE http://localhost:3000/api/a5k9m2x1
```

---

## Step 13: Add Health Check Endpoint

**What it does:** Provides statistics about the API - how many collections exist, total sticky notes, etc.

**Add this route:**
```javascript
// Health check endpoint
app.get('/api/health', (req, res) => {
    const totalStickies = stickiesData.reduce((sum, stickySet) => sum + stickySet.stickies.length, 0);

    res.json({
        status: 'OK',
        timestamp: new Date().toISOString(),
        totalStickiesCollections: stickiesData.length,
        totalStickies: totalStickies,
        recentStickies: stickiesData
            .sort((a, b) => new Date(b.lastAccessed) - new Date(a.lastAccessed))
            .slice(0, 5)
            .map(stickySet => ({
                id: stickySet.id,
                stickiesCount: stickySet.stickies.length,
                lastAccessed: stickySet.lastAccessed
            }))
    });
});
```

**How it works:**
1. Counts total sticky notes across all collections
2. Shows number of collections
3. Lists the 5 most recently accessed collections
4. Provides current timestamp

**Test it:**
```bash
curl -X GET http://localhost:3000/api/health
```

**Expected response:**
```json
{
  "status": "OK",
  "timestamp": "2025-06-10T10:45:00.000Z",
  "totalStickiesCollections": 2,
  "totalStickies": 5,
  "recentStickies": [
    {
      "id": "a5k9m2x1",
      "stickiesCount": 3,
      "lastAccessed": "2025-06-10T10:44:00.000Z"
    }
  ]
}
```

---

## Step 14: Add Error Handling

**What it does:** Catches unexpected errors and provides proper error responses.

**Add these middleware functions before `app.listen()`:**
```javascript
// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ error: 'Something went wrong!' });
});

// 404 handler
app.use((req, res) => {
    res.status(404).json({ error: 'Endpoint not found' });
});
```

**How it works:**
1. First middleware catches any unhandled errors
2. Second middleware handles requests to non-existent endpoints

**Test it:**
```bash
# Test 404 handler
curl -X GET http://localhost:3000/nonexistent

# Expected response:
# {"error": "Endpoint not found"}
```

---

## Complete Testing Workflow

Now test the complete API workflow:

### 1. Create a sticky collection:
```bash
curl -X POST http://localhost:3000/api/ \
  -H "Content-Type: application/json" \
  -d '[{"id":1,"text":"Learn Express.js","color":"yellow","x":100,"y":200}]'
```

### 2. Get the collection (use the ID from step 1):
```bash
curl -X GET http://localhost:3000/api/[YOUR_ID_HERE]
```

### 3. Update the collection:
```bash
curl -X PUT http://localhost:3000/api/[YOUR_ID_HERE] \
  -H "Content-Type: application/json" \
  -d '[{"id":1,"text":"Learn Express.js","color":"yellow","x":100,"y":200},{"id":2,"text":"Build sticky notes app","color":"blue","x":300,"y":150}]'
```

### 4. Check health status:
```bash
curl -X GET http://localhost:3000/api/health
```

### 5. Delete the collection:
```bash
curl -X DELETE http://localhost:3000/api/[YOUR_ID_HERE]
```

## Congratulations!

You've built a complete REST API with:
- ✅ Create sticky collections (POST)
- ✅ Read sticky collections (GET)
- ✅ Update sticky collections (PUT)
- ✅ Delete sticky collections (DELETE)
- ✅ Health monitoring
- ✅ Static file serving
- ✅ Error handling
- ✅ Input validation

## Next Steps

1. **Add a frontend:** Create HTML/CSS/JavaScript in the `static` folder to interact with your API
2. **Add persistence:** Save data to a file or database instead of memory
3. **Add authentication:** Protect certain endpoints
4. **Add more features:** Search, filtering, categories, etc.

## Common Issues & Solutions

**Server won't start:**
- Check if port 3000 is already in use
- Make sure you installed express: `npm install express`

**Curl commands not working:**
- On Windows, use double quotes instead of single quotes
- Make sure the server is running
- Check the URL and port number

**JSON parsing errors:**
- Ensure Content-Type header is set correctly
- Check JSON syntax (use a JSON validator)

**404 errors:**
- Double-check the URL path
- Make sure you're using the correct HTTP method (GET, POST, PUT)