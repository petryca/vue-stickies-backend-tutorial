# Vue Sticky Notes: Backend Server Integration Quiz

## Introduction

This assignment is based on the **"Vue Sticky Notes: Migrate from localStorage to Server Integration"** tutorial document. The questions below cover all major concepts, implementation details, and best practices discussed in the migration guide. This assignment will test your understanding of converting a localStorage-based Vue.js application to a server-integrated solution with debounced auto-save functionality.

## Submission Guidelines

- **Format**: Must be submitted as a markdown document (.md)
- **Filename**: `server-integration-answers.md`
- **Structure**: Include the question number and your complete answer for each question
- **Code Examples**: When relevant, include code snippets to support your explanations
- **Length**: Provide comprehensive answers that demonstrate understanding of the concepts

---

## Questions

### Step 1: Remove localStorage Dependencies

**1.1 Remove localStorage Properties**
Q: What is the `storageKey` variable and why should it be removed when migrating to server integration?

**1.2 localStorage Method Removal**
Q: List the two localStorage methods that need to be completely removed and explain what each method was responsible for in the original implementation.

**1.3 Watcher Update**
Q: How does the watcher's functionality change when migrating from localStorage to server integration?

### Step 2: Add Server Integration Properties

**2.1 New Data Properties**
Q: List and explain the purpose of each of the six new data properties that need to be added for server integration.

**2.2 Property Relationships**
Q: Explain how `saveTimeout`, `isLoading`, and `hasUnsavedChanges` work together to manage the debounced auto-save functionality.

### Step 3: Implement URL Detection and Loading

**3.1 URL Pattern Validation**
Q: What is the specific regex pattern used to validate sticky note IDs in URLs, and why is this validation important?

**3.2 Mounted Lifecycle Changes**
Q: Describe the complete flow of what happens in the `mounted()` lifecycle method when a user visits a URL with a valid stickies ID versus an invalid or missing ID.

### Step 4: Create Server Communication Methods

**4.1 loadFromServer() Method**
Q: What are the three different response scenarios that `loadFromServer()` handles, and what specific action is taken for each scenario?

**4.2 saveToServer() Logic Split**
Q: Explain why the `saveToServer()` method uses different HTTP methods (PUT vs DELETE) based on the stickies array state, and describe the complete flow for each case.

**4.3 Error Handling Strategy**
Q: How does the application handle server communication errors in both loading and saving operations, and what fallback behavior is implemented?

### Step 5: Implement Debounced Auto-Save

**5.1 Debouncing Mechanism**
Q: Explain how the debouncing mechanism works in `debouncedSave()`, including the timeout duration and the logic for clearing existing timeouts.

**5.2 New vs Existing Stickies**
Q: In `createNewStickiesOnServer()`, what condition determines whether a new sticky set should be created on the server, and why is this check important?

**5.3 URL History Management**
Q: How does the application update the browser URL without causing a page reload when a new sticky set is created, and why is this approach beneficial?

### Step 6: Handle Deletion with Dedicated DELETE Route

**6.1 RESTful API Design**
Q: List all four API routes mentioned in the tutorial and explain how this follows RESTful design principles.

**6.2 Deletion Logic**
Q: In the updated `deleteStickie()` method, what happens when the last sticky note is deleted, and how is this different from deleting a sticky note when others remain?

**6.3 State Reset Process**
Q: Describe what the `refreshToNewStickySet()` method does and when it gets called in the application flow.

### Step 7: Add Bonus Features

**7.1 Share Functionality**
Q: Explain the complete flow of the `shareStickies()` method, including what happens when no `currentStickiesId` exists and the fallback mechanism for browsers without clipboard API support.

**7.2 Clear Storage Update**
Q: How does the `clearStorage()` method differ from the original localStorage version, and what additional cleanup is required for server integration?

### Comprehensive Understanding Questions

**C.1 Migration Benefits**
Q: Compare the limitations of localStorage-based storage with the advantages provided by server integration, considering collaboration, persistence, and sharing capabilities.

**C.2 Data Flow Analysis**
Q: Trace the complete data flow from when a user types in a sticky note to when the data is successfully saved on the server, including all the intermediate states and timing considerations.
