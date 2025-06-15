# Complete Full-Stack Tutorial: Monolithic vs Separated Architecture

## Table of Contents
1. [Introduction](#introduction)
2. [Part 1: Monolithic Approach](#part-1-monolithic-approach)
3. [Part 2: Separated Architecture](#part-2-separated-architecture)
4. [Comparison and Best Practices](#comparison-and-best-practices)

---

## Introduction

In this tutorial, we'll build the same application using two different architectural approaches:
1. **Monolithic**: Single Node.js application serving both frontend and backend
2. **Separated**: Independent React frontend + Node.js backend API

**What we're building**: A simple Task Manager application with:
- View all tasks
- Add new tasks
- Mark tasks as complete/incomplete
- Delete tasks

### Prerequisites

Before starting, ensure you have:
- **Node.js 14+** and **npm** installed ([Download here](https://nodejs.org/))
- A code editor (VS Code recommended)
- Basic knowledge of HTML, CSS, and JavaScript
- Basic understanding of command line/terminal

**⚠️ IMPORTANT**: Run only ONE approach at a time to avoid port conflicts, or modify the ports as instructed.

---

## Part 1: Monolithic Approach

### What is Monolithic Architecture?

In a monolithic approach, your entire application runs as a single unit. The server handles both:
- **Backend logic** (database operations, business logic)
- **Frontend rendering** (serving HTML, CSS, JavaScript)

### Step 1: Project Setup

Create a new directory and initialize the project:

```bash
mkdir task-manager-monolithic
cd task-manager-monolithic
npm init -y
```

### Step 2: Install Dependencies

```bash
npm install express ejs body-parser
npm install --save-dev nodemon
```

**Why these packages?**
- `express`: Web framework for Node.js
- `ejs`: Template engine to render dynamic HTML
- `body-parser`: Parse incoming request bodies
- `nodemon`: Development tool that restarts server on file changes

### Step 3: Project Structure

Create the following folder structure:

```
task-manager-monolithic/
├── server.js
├── package.json
├── views/
│   ├── index.ejs
│   └── partials/
│       ├── header.ejs
│       └── footer.ejs
├── public/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── script.js
└── data/
    └── tasks.js
```

### Step 4: Create the Server (server.js)

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const path = require('path');

// Import our data (we'll create this next)
const { getTasks, addTask, updateTask, deleteTask } = require('./data/tasks');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware Setup
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'public')));

// Set EJS as template engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Routes

// HOME PAGE - Display all tasks
app.get('/', (req, res) => {
    try {
        const tasks = getTasks();
        res.render('index', { 
            title: 'Task Manager', 
            tasks: tasks 
        });
    } catch (error) {
        console.error('Error rendering home page:', error);
        res.status(500).send('Server Error');
    }
});

// API Routes (these handle AJAX requests from frontend)

// GET all tasks (API endpoint)
app.get('/api/tasks', (req, res) => {
    try {
        res.json(getTasks());
    } catch (error) {
        console.error('Error fetching tasks:', error);
        res.status(500).json({ error: 'Failed to fetch tasks' });
    }
});

// ADD new task
app.post('/api/tasks', (req, res) => {
    try {
        const { title } = req.body;
        
        if (!title || title.trim() === '') {
            return res.status(400).json({ error: 'Task title is required' });
        }
        
        if (title.trim().length > 200) {
            return res.status(400).json({ error: 'Task title must be less than 200 characters' });
        }
        
        const newTask = addTask(title.trim());
        res.status(201).json(newTask);
    } catch (error) {
        console.error('Error creating task:', error);
        res.status(500).json({ error: 'Failed to create task' });
    }
});

// UPDATE task (mark complete/incomplete)
app.put('/api/tasks/:id', (req, res) => {
    try {
        const taskId = parseInt(req.params.id);
        const { completed } = req.body;
        
        if (isNaN(taskId)) {
            return res.status(400).json({ error: 'Invalid task ID' });
        }
        
        const updatedTask = updateTask(taskId, { completed });
        
        if (!updatedTask) {
            return res.status(404).json({ error: 'Task not found' });
        }
        
        res.json(updatedTask);
    } catch (error) {
        console.error('Error updating task:', error);
        res.status(500).json({ error: 'Failed to update task' });
    }
});

// DELETE task
app.delete('/api/tasks/:id', (req, res) => {
    try {
        const taskId = parseInt(req.params.id);
        
        if (isNaN(taskId)) {
            return res.status(400).json({ error: 'Invalid task ID' });
        }
        
        const success = deleteTask(taskId);
        
        if (!success) {
            return res.status(404).json({ error: 'Task not found' });
        }
        
        res.status(204).send();
    } catch (error) {
        console.error('Error deleting task:', error);
        res.status(500).json({ error: 'Failed to delete task' });
    }
});

// Global error handler
app.use((error, req, res, next) => {
    console.error('Unhandled error:', error);
    res.status(500).json({ error: 'Internal server error' });
});

// Start server
app.listen(PORT, () => {
    console.log(`Monolithic server running on http://localhost:${PORT}`);
    console.log('Press Ctrl+C to stop the server');
});
```

**Key Concepts Explained:**
- **Express middleware**: Functions that process requests before they reach your routes
- **Template engine (EJS)**: Allows us to generate dynamic HTML with JavaScript
- **Static files**: CSS/JS files served directly by Express
- **RESTful routes**: Different HTTP methods (GET, POST, PUT, DELETE) for different operations

### Step 5: Create Data Layer (data/tasks.js)

```javascript
// In-memory data storage (in production, you'd use a database)
let tasks = [
    { id: 1, title: 'Learn Node.js', completed: false, createdAt: new Date() },
    { id: 2, title: 'Build a web app', completed: true, createdAt: new Date() },
    { id: 3, title: 'Deploy to production', completed: false, createdAt: new Date() }
];

let nextId = 4;

// Get all tasks
function getTasks() {
    return tasks.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
}

// Add new task
function addTask(title) {
    const newTask = {
        id: nextId++,
        title,
        completed: false,
        createdAt: new Date()
    };
    
    tasks.push(newTask);
    return newTask;
}

// Update task
function updateTask(id, updates) {
    const taskIndex = tasks.findIndex(task => task.id === id);
    
    if (taskIndex === -1) {
        return null;
    }
    
    tasks[taskIndex] = { ...tasks[taskIndex], ...updates };
    return tasks[taskIndex];
}

// Delete task
function deleteTask(id) {
    const taskIndex = tasks.findIndex(task => task.id === id);
    
    if (taskIndex === -1) {
        return false;
    }
    
    tasks.splice(taskIndex, 1);
    return true;
}

module.exports = {
    getTasks,
    addTask,
    updateTask,
    deleteTask
};
```

**Why separate the data layer?**
- **Single Responsibility**: Each function has one job
- **Testability**: Easy to test data operations independently
- **Maintainability**: Changes to data structure only affect this file

### Step 6: Create Views

**views/partials/header.ejs**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= title %></title>
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <header>
        <h1><%= title %></h1>
    </header>
    <main>
```

**views/partials/footer.ejs**
```html
    </main>
    <script src="/js/script.js"></script>
</body>
</html>
```

**views/index.ejs**
```html
<%- include('partials/header') %>

<div class="container">
    <!-- Add Task Form -->
    <div class="add-task-section">
        <h2>Add New Task</h2>
        <form id="add-task-form">
            <input 
                type="text" 
                id="task-input" 
                placeholder="Enter task title..." 
                required
            >
            <button type="submit">Add Task</button>
        </form>
    </div>

    <!-- Tasks List -->
    <div class="tasks-section">
        <h2>Your Tasks</h2>
        <div id="tasks-container">
            <% if (tasks.length === 0) { %>
                <p class="no-tasks">No tasks yet. Add one above!</p>
            <% } else { %>
                <% tasks.forEach(task => { %>
                    <div class="task-item <%= task.completed ? 'completed' : '' %>" data-id="<%= task.id %>">
                        <div class="task-content">
                            <input 
                                type="checkbox" 
                                class="task-checkbox" 
                                <%= task.completed ? 'checked' : '' %>
                            >
                            <span class="task-title"><%= task.title %></span>
                        </div>
                        <div class="task-actions">
                            <button class="delete-btn" data-id="<%= task.id %>">Delete</button>
                        </div>
                    </div>
                <% }); %>
            <% } %>
        </div>
    </div>
</div>

<%- include('partials/footer') %>
```

**EJS Syntax Explained:**
- `<%= %>`: Outputs the value (escaped for security)
- `<%- %>`: Outputs the value (unescaped, used for including other templates)
- `<% %>`: JavaScript code that doesn't output anything
- `<% if (condition) { %>`: Conditional rendering
- `<% forEach() { %>`: Loop through arrays

### Step 7: Add Styling (public/css/style.css)

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    line-height: 1.6;
    color: #333;
    background-color: #f4f4f4;
}

header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    text-align: center;
    padding: 2rem 0;
    margin-bottom: 2rem;
}

header h1 {
    font-size: 2.5rem;
    font-weight: 300;
}

.container {
    max-width: 800px;
    margin: 0 auto;
    padding: 0 2rem;
}

.add-task-section, .tasks-section {
    background: white;
    padding: 2rem;
    margin-bottom: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.add-task-section h2, .tasks-section h2 {
    margin-bottom: 1rem;
    color: #555;
}

#add-task-form {
    display: flex;
    gap: 1rem;
}

#task-input {
    flex: 1;
    padding: 0.75rem;
    border: 2px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
}

#task-input:focus {
    outline: none;
    border-color: #667eea;
}

button {
    padding: 0.75rem 1.5rem;
    background: #667eea;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.3s;
}

button:hover {
    background: #5a67d8;
}

.task-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    border: 1px solid #eee;
    border-radius: 4px;
    margin-bottom: 0.5rem;
    transition: all 0.3s;
}

.task-item:hover {
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.task-item.completed {
    opacity: 0.7;
    background-color: #f8f9fa;
}

.task-content {
    display: flex;
    align-items: center;
    gap: 0.75rem;
}

.task-checkbox {
    width: 18px;
    height: 18px;
    cursor: pointer;
}

.task-title {
    font-size: 1.1rem;
}

.task-item.completed .task-title {
    text-decoration: line-through;
    color: #888;
}

.delete-btn {
    background: #e53e3e;
    padding: 0.5rem 1rem;
    font-size: 0.9rem;
}

.delete-btn:hover {
    background: #c53030;
}

.no-tasks {
    text-align: center;
    color: #888;
    font-style: italic;
    padding: 2rem 0;
}

@media (max-width: 600px) {
    .container {
        padding: 0 1rem;
    }
    
    #add-task-form {
        flex-direction: column;
    }
    
    .task-item {
        flex-direction: column;
        align-items: flex-start;
        gap: 1rem;
    }
    
    .task-actions {
        align-self: flex-end;
    }
}
```

### Step 8: Add Frontend JavaScript (public/js/script.js)

```javascript
// Wait for DOM to be fully loaded
document.addEventListener('DOMContentLoaded', function() {
    
    // Get references to DOM elements
    const addTaskForm = document.getElementById('add-task-form');
    const taskInput = document.getElementById('task-input');
    const tasksContainer = document.getElementById('tasks-container');
    
    // Add task form submission
    addTaskForm.addEventListener('submit', async function(e) {
        e.preventDefault(); // Prevent form from submitting normally
        
        const title = taskInput.value.trim();
        
        if (!title) {
            alert('Please enter a task title');
            return;
        }
        
        try {
            // Send POST request to our API
            const response = await fetch('/api/tasks', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ title })
            });
            
            if (response.ok) {
                // Clear input and reload tasks
                taskInput.value = '';
                await loadTasks();
            } else {
                const error = await response.json();
                alert('Error: ' + error.error);
            }
        } catch (error) {
            console.error('Error adding task:', error);
            alert('Failed to add task. Please try again.');
        }
    });
    
    // Handle task interactions (checkboxes and delete buttons)
    tasksContainer.addEventListener('click', async function(e) {
        const taskItem = e.target.closest('.task-item');
        const taskId = taskItem ? parseInt(taskItem.dataset.id) : null;
        
        if (!taskId) return;
        
        // Handle checkbox clicks
        if (e.target.classList.contains('task-checkbox')) {
            const completed = e.target.checked;
            
            try {
                const response = await fetch(`/api/tasks/${taskId}`, {
                    method: 'PUT',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ completed })
                });
                
                if (response.ok) {
                    // Update UI immediately
                    taskItem.classList.toggle('completed', completed);
                } else {
                    // Revert checkbox if request failed
                    e.target.checked = !completed;
                    alert('Failed to update task');
                }
            } catch (error) {
                console.error('Error updating task:', error);
                e.target.checked = !completed;
                alert('Failed to update task. Please try again.');
            }
        }
        
        // Handle delete button clicks
        if (e.target.classList.contains('delete-btn')) {
            if (confirm('Are you sure you want to delete this task?')) {
                try {
                    const response = await fetch(`/api/tasks/${taskId}`, {
                        method: 'DELETE'
                    });
                    
                    if (response.ok) {
                        await loadTasks();
                    } else {
                        alert('Failed to delete task');
                    }
                } catch (error) {
                    console.error('Error deleting task:', error);
                    alert('Failed to delete task. Please try again.');
                }
            }
        }
    });
    
    // Function to reload all tasks
    async function loadTasks() {
        try {
            const response = await fetch('/api/tasks');
            const tasks = await response.json();
            
            // Clear current tasks
            tasksContainer.innerHTML = '';
            
            if (tasks.length === 0) {
                tasksContainer.innerHTML = '<p class="no-tasks">No tasks yet. Add one above!</p>';
                return;
            }
            
            // Create task elements
            tasks.forEach(task => {
                const taskElement = createTaskElement(task);
                tasksContainer.appendChild(taskElement);
            });
            
        } catch (error) {
            console.error('Error loading tasks:', error);
            tasksContainer.innerHTML = '<p class="no-tasks">Error loading tasks. Please refresh the page.</p>';
        }
    }
    
    // Helper function to create task DOM element
    function createTaskElement(task) {
        const taskDiv = document.createElement('div');
        taskDiv.className = `task-item ${task.completed ? 'completed' : ''}`;
        taskDiv.dataset.id = task.id;
        
        taskDiv.innerHTML = `
            <div class="task-content">
                <input 
                    type="checkbox" 
                    class="task-checkbox" 
                    ${task.completed ? 'checked' : ''}
                >
                <span class="task-title">${escapeHtml(task.title)}</span>
            </div>
            <div class="task-actions">
                <button class="delete-btn" data-id="${task.id}">Delete</button>
            </div>
        `;
        
        return taskDiv;
    }
    
    // Helper function to escape HTML (prevent XSS attacks)
    function escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }
});
```

**JavaScript Concepts Explained:**
- **Event Delegation**: Using one event listener on the parent to handle clicks on child elements
- **Async/Await**: Modern way to handle asynchronous operations
- **Fetch API**: Modern way to make HTTP requests
- **DOM Manipulation**: Creating and modifying HTML elements with JavaScript
- **XSS Prevention**: Escaping user input to prevent security vulnerabilities

### Step 9: Update package.json Scripts

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

### Step 10: Run the Monolithic Application

```bash
npm run dev
```

Visit `http://localhost:3000` to see your application!

**⚠️ Important Notes:**
- Make sure no other applications are running on port 3000
- If you get a "port already in use" error, kill the process or change the PORT in server.js
- The server will automatically restart when you make changes (thanks to nodemon)
- Data is stored in memory and will be lost when the server restarts

### Troubleshooting Common Issues

**"Cannot find module" errors:**
- Ensure you're in the correct directory
- Run `npm install` again
- Check that all files are created in the right locations

**Port conflicts:**
- Change the PORT variable in server.js to a different number (e.g., 3001)
- Or stop other applications using port 3000

**Template errors:**
- Ensure the `views` folder structure is correct
- Check that all EJS files are saved with `.ejs` extension

### Monolithic Approach - Summary

**What we built:**
- Single Node.js server handling both frontend and backend
- Server-side rendering with EJS templates
- API endpoints for dynamic interactions
- Complete task management functionality

**Key Characteristics:**
- **Single deployment unit**: Everything runs from one server
- **Shared codebase**: Frontend and backend code in same project
- **Direct data access**: No network calls between frontend and backend logic
- **Server-side rendering**: Initial HTML generated on server

---

## Part 2: Separated Architecture

### What is Separated Architecture?

In separated architecture, we have two independent applications:
1. **Backend API**: Node.js server that only handles data and business logic
2. **Frontend App**: React application that runs in the browser

They communicate through HTTP requests (REST API).

### Step 1: Project Setup

Create the project structure:

```bash
mkdir task-manager-separated
cd task-manager-separated
mkdir backend frontend
```

### Step 2: Backend Setup

```bash
cd backend
npm init -y
npm install express cors
npm install --save-dev nodemon
```

**New package: `cors`**
- **CORS (Cross-Origin Resource Sharing)**: Allows our React app (running on port 3000) to make requests to our API (running on port 5000)

### Step 3: Backend Structure

Create the directory structure manually or run these commands:

```bash
# Create directories
mkdir routes data

# Create files (on Windows, use 'type nul >' instead of 'touch')
touch server.js
touch routes/tasks.js  
touch data/tasks.js
```

Final structure:
```
backend/
├── server.js
├── package.json
├── routes/
│   └── tasks.js
├── data/
│   └── tasks.js
└── node_modules/
```

### Step 4: Backend Data Layer (backend/data/tasks.js)

```javascript
// Same as monolithic version, but exported differently
let tasks = [
    { id: 1, title: 'Learn React', completed: false, createdAt: new Date() },
    { id: 2, title: 'Build REST API', completed: true, createdAt: new Date() },
    { id: 3, title: 'Connect Frontend to Backend', completed: false, createdAt: new Date() }
];

let nextId = 4;

class TaskService {
    static getAllTasks() {
        return tasks.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
    }
    
    static createTask(title) {
        const newTask = {
            id: nextId++,
            title: title.trim(),
            completed: false,
            createdAt: new Date()
        };
        
        tasks.push(newTask);
        return newTask;
    }
    
    static updateTask(id, updates) {
        const taskIndex = tasks.findIndex(task => task.id === id);
        
        if (taskIndex === -1) {
            return null;
        }
        
        tasks[taskIndex] = { 
            ...tasks[taskIndex], 
            ...updates,
            updatedAt: new Date()
        };
        
        return tasks[taskIndex];
    }
    
    static deleteTask(id) {
        const taskIndex = tasks.findIndex(task => task.id === id);
        
        if (taskIndex === -1) {
            return false;
        }
        
        tasks.splice(taskIndex, 1);
        return true;
    }
    
    static getTaskById(id) {
        return tasks.find(task => task.id === id);
    }
}

module.exports = TaskService;
```

**Why use a class?**
- **Organization**: Groups related methods together
- **Consistency**: Standard pattern for service layers
- **Scalability**: Easy to add more methods and functionality

### Step 5: Backend Routes (backend/routes/tasks.js)

```javascript
const express = require('express');
const TaskService = require('../data/tasks');

const router = express.Router();

// GET /api/tasks - Get all tasks
router.get('/', (req, res) => {
    try {
        const tasks = TaskService.getAllTasks();
        res.json({
            success: true,
            data: tasks,
            count: tasks.length
        });
    } catch (error) {
        console.error('Error fetching tasks:', error);
        res.status(500).json({
            success: false,
            error: 'Internal server error'
        });
    }
});

// GET /api/tasks/:id - Get single task
router.get('/:id', (req, res) => {
    try {
        const id = parseInt(req.params.id);
        
        if (isNaN(id)) {
            return res.status(400).json({
                success: false,
                error: 'Invalid task ID'
            });
        }
        
        const task = TaskService.getTaskById(id);
        
        if (!task) {
            return res.status(404).json({
                success: false,
                error: 'Task not found'
            });
        }
        
        res.json({
            success: true,
            data: task
        });
    } catch (error) {
        console.error('Error fetching task:', error);
        res.status(500).json({
            success: false,
            error: 'Internal server error'
        });
    }
});

// POST /api/tasks - Create new task
router.post('/', (req, res) => {
    try {
        const { title } = req.body;
        
        // Validation
        if (!title || typeof title !== 'string' || title.trim() === '') {
            return res.status(400).json({
                success: false,
                error: 'Task title is required and must be a non-empty string'
            });
        }
        
        if (title.trim().length > 200) {
            return res.status(400).json({
                success: false,
                error: 'Task title must be less than 200 characters'
            });
        }
        
        const newTask = TaskService.createTask(title);
        
        res.status(201).json({
            success: true,
            data: newTask,
            message: 'Task created successfully'
        });
    } catch (error) {
        console.error('Error creating task:', error);
        res.status(500).json({
            success: false,
            error: 'Internal server error'
        });
    }
});

// PUT /api/tasks/:id - Update task
router.put('/:id', (req, res) => {
    try {
        const id = parseInt(req.params.id);
        
        if (isNaN(id)) {
            return res.status(400).json({
                success: false,
                error: 'Invalid task ID'
            });
        }
        
        const { title, completed } = req.body;
        const updates = {};
        
        // Validate and add updates
        if (title !== undefined) {
            if (typeof title !== 'string' || title.trim() === '') {
                return res.status(400).json({
                    success: false,
                    error: 'Title must be a non-empty string'
                });
            }
            updates.title = title.trim();
        }
        
        if (completed !== undefined) {
            if (typeof completed !== 'boolean') {
                return res.status(400).json({
                    success: false,
                    error: 'Completed must be a boolean value'
                });
            }
            updates.completed = completed;
        }
        
        const updatedTask = TaskService.updateTask(id, updates);
        
        if (!updatedTask) {
            return res.status(404).json({
                success: false,
                error: 'Task not found'
            });
        }
        
        res.json({
            success: true,
            data: updatedTask,
            message: 'Task updated successfully'
        });
    } catch (error) {
        console.error('Error updating task:', error);
        res.status(500).json({
            success: false,
            error: 'Internal server error'
        });
    }
});

// DELETE /api/tasks/:id - Delete task
router.delete('/:id', (req, res) => {
    try {
        const id = parseInt(req.params.id);
        
        if (isNaN(id)) {
            return res.status(400).json({
                success: false,
                error: 'Invalid task ID'
            });
        }
        
        const success = TaskService.deleteTask(id);
        
        if (!success) {
            return res.status(404).json({
                success: false,
                error: 'Task not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Task deleted successfully'
        });
    } catch (error) {
        console.error('Error deleting task:', error);
        res.status(500).json({
            success: false,
            error: 'Internal server error'
        });
    }
});

module.exports = router;
```

**API Design Best Practices:**
- **Consistent response format**: All responses have `success`, `data`, and optional `error`/`message`
- **Proper HTTP status codes**: 200 (OK), 201 (Created), 400 (Bad Request), 404 (Not Found), 500 (Server Error)
- **Input validation**: Check all user inputs before processing
- **Error handling**: Try-catch blocks and meaningful error messages

### Step 6: Backend Server (backend/server.js)

```javascript
const express = require('express');
const cors = require('cors');

// Import routes
const taskRoutes = require('./routes/tasks');

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors({
    origin: ['http://localhost:3000', 'http://127.0.0.1:3000'], // React app URLs
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));

app.use(express.json({ limit: '10mb' })); // Parse JSON bodies
app.use(express.urlencoded({ extended: true, limit: '10mb' })); // Parse URL-encoded bodies

// Request logging middleware
app.use((req, res, next) => {
    console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
    next();
});

// API Routes
app.use('/api/tasks', taskRoutes);

// Health check endpoint
app.get('/health', (req, res) => {
    res.json({
        success: true,
        message: 'Backend server is running!',
        timestamp: new Date().toISOString(),
        port: PORT
    });
});

// Root endpoint
app.get('/', (req, res) => {
    res.json({
        message: 'Task Manager API',
        version: '1.0.0',
        endpoints: {
            health: '/health',
            tasks: '/api/tasks'
        }
    });
});

// 404 handler for undefined routes
app.use('*', (req, res) => {
    res.status(404).json({
        success: false,
        error: `Route ${req.method} ${req.originalUrl} not found`
    });
});

// Global error handler
app.use((error, req, res, next) => {
    console.error('Unhandled error:', error);
    res.status(500).json({
        success: false,
        error: 'Internal server error',
        message: process.env.NODE_ENV === 'development' ? error.message : 'Something went wrong'
    });
});

app.listen(PORT, () => {
    console.log(`Backend server running on http://localhost:${PORT}`);
    console.log(`Health check: http://localhost:${PORT}/health`);
    console.log(`API documentation: http://localhost:${PORT}/`);
    console.log('Press Ctrl+C to stop the server');
});
```

### Step 7: Backend package.json Scripts

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

### Step 8: Frontend Setup

Open a new terminal and navigate to the frontend directory:

```bash
cd ../frontend  # Go back to main directory, then to frontend
npx create-react-app . --template typescript
```

**If you get an error about the directory not being empty:**
```bash
# Go back and create a new frontend directory
cd ..
rm -rf frontend  # On Windows use: rmdir /s frontend
mkdir frontend
cd frontend
npx create-react-app . --template typescript
```

Wait for installation (this may take 5-10 minutes), then install additional dependencies:

```bash
npm install axios
```

**Why axios?**
- **HTTP client**: Makes API requests easier than fetch
- **Request/response interceptors**: Automatic error handling
- **Better error handling**: More detailed error information
- **Timeout support**: Prevents hanging requests

### Verify Installation

Check that these files exist:
- `package.json` (should contain React and TypeScript dependencies)
- `src/App.tsx` (TypeScript React component)
- `tsconfig.json` (TypeScript configuration)

Test the React app works:
```bash
npm start
```

You should see the default React app at `http://localhost:3000`. Stop it with Ctrl+C before continuing.

### Step 9: Frontend Structure

Create the directory structure for components:

```bash
# Create directories
mkdir src/components src/services src/types src/hooks

# Create files (adjust for Windows if needed)
touch src/components/TaskForm.tsx
touch src/components/TaskList.tsx  
touch src/components/TaskItem.tsx
touch src/services/api.ts
touch src/types/task.ts
touch src/hooks/useTasks.ts
```

Final structure:
```
frontend/
├── public/
├── src/
│   ├── components/
│   │   ├── TaskForm.tsx
│   │   ├── TaskList.tsx
│   │   └── TaskItem.tsx
│   ├── services/
│   │   └── api.ts
│   ├── types/
│   │   └── task.ts
│   ├── hooks/
│   │   └── useTasks.ts
│   ├── App.tsx
│   ├── App.css
│   ├── index.tsx
│   └── index.css
├── package.json
└── tsconfig.json
```

### Step 10: Define Types (src/types/task.ts)

```typescript
export interface Task {
    id: number;
    title: string;
    completed: boolean;
    createdAt: string;
    updatedAt?: string;
}

export interface ApiResponse<T> {
    success: boolean;
    data?: T;
    error?: string;
    message?: string;
    count?: number;
}

export interface CreateTaskRequest {
    title: string;
}

export interface UpdateTaskRequest {
    title?: string;
    completed?: boolean;
}
```

**Why TypeScript?**
- **Type safety**: Catch errors at compile time, not runtime
- **Better IDE support**: Autocomplete, refactoring, navigation
- **Self-documenting code**: Types serve as documentation
- **Easier refactoring**: Compiler helps you update all affected code

### Step 11: API Service (src/services/api.ts)

```typescript
import axios from 'axios';
import { Task, ApiResponse, CreateTaskRequest, UpdateTaskRequest } from '../types/task';

// Create axios instance with base configuration
const api = axios.create({
    baseURL: 'http://localhost:5000/api',
    timeout: 10000,
    headers: {
        'Content-Type': 'application/json',
    },
});

// Request interceptor - runs before every request
api.interceptors.request.use(
    (config) => {
        console.log(`Making ${config.method?.toUpperCase()} request to ${config.url}`);
        return config;
    },
    (error) => {
        console.error('Request error:', error);
        return Promise.reject(error);
    }
);

// Response interceptor - runs after every response
api.interceptors.response.use(
    (response) => {
        console.log('Response received:', response.status);
        return response;
    },
    (error) => {
        console.error('Response error:', error.response?.data || error.message);
        
        // Handle different error types
        if (error.response?.status === 404) {
            console.error('Resource not found');
        } else if (error.response?.status === 500) {
            console.error('Server error');
        } else if (error.code === 'ECONNABORTED') {
            console.error('Request timeout');
        }
        
        return Promise.reject(error);
    }
);

// API service class
export class TaskService {
    
    // Get all tasks
    static async getAllTasks(): Promise<Task[]> {
        try {
            const response = await api.get<ApiResponse<Task[]>>('/tasks');
            return response.data.data || [];
        } catch (error) {
            console.error('Error fetching tasks:', error);
            throw new Error('Failed to fetch tasks');
        }
    }
    
    // Get single task
    static async getTaskById(id: number): Promise<Task> {
        try {
            const response = await api.get<ApiResponse<Task>>(`/tasks/${id}`);
            if (!response.data.data) {
                throw new Error('Task not found');
            }
            return response.data.data;
        } catch (error) {
            console.error('Error fetching task:', error);
            throw new Error('Failed to fetch task');
        }
    }
    
    // Create new task
    static async createTask(taskData: CreateTaskRequest): Promise<Task> {
        try {
            const response = await api.post<ApiResponse<Task>>('/tasks', taskData);
            if (!response.data.data) {
                throw new Error('Failed to create task');
            }
            return response.data.data;
        } catch (error) {
            console.error('Error creating task:', error);
            if (axios.isAxiosError(error) && error.response?.data?.error) {
                throw new Error(error.response.data.error);
            }
            throw new Error('Failed to create task');
        }
    }
    
    // Update task
    static async updateTask(id: number, updates: UpdateTaskRequest): Promise<Task> {
        try {
            const response = await api.put<ApiResponse<Task>>(`/tasks/${id}`, updates);
            if (!response.data.data) {
                throw new Error('Failed to update task');
            }
            return response.data.data;
        } catch (error) {
            console.error('Error updating task:', error);
            if (axios.isAxiosError(error) && error.response?.data?.error) {
                throw new Error(error.response.data.error);
            }
            throw new Error('Failed to update task');
        }
    }
    
    // Delete task
    static async deleteTask(id: number): Promise<void> {
        try {
            await api.delete(`/tasks/${id}`);
        } catch (error) {
            console.error('Error deleting task:', error);
            if (axios.isAxiosError(error) && error.response?.data?.error) {
                throw new Error(error.response.data.error);
            }
            throw new Error('Failed to delete task');
        }
    }
}
```

**Service Layer Benefits:**
- **Separation of concerns**: UI components don't need to know about HTTP details
- **Reusability**: Multiple components can use the same service methods
- **Error handling**: Centralized error handling and logging
- **Type safety**: TypeScript ensures correct data types

### Step 12: Custom Hook (src/hooks/useTasks.ts)

```typescript
import { useState, useEffect, useCallback } from 'react';
import { Task, CreateTaskRequest, UpdateTaskRequest } from '../types/task';
import { TaskService } from '../services/api';

interface UseTasksReturn {
    tasks: Task[];
    loading: boolean;
    error: string | null;
    createTask: (taskData: CreateTaskRequest) => Promise<void>;
    updateTask: (id: number, updates: UpdateTaskRequest) => Promise<void>;
    deleteTask: (id: number) => Promise<void>;
    refetchTasks: () => Promise<void>;
}

export const useTasks = (): UseTasksReturn => {
    const [tasks, setTasks] = useState<Task[]>([]);
    const [loading, setLoading] = useState<boolean>(true);
    const [error, setError] = useState<string | null>(null);
    
    // Fetch tasks function
    const fetchTasks = useCallback(async () => {
        try {
            setLoading(true);
            setError(null);
            const fetchedTasks = await TaskService.getAllTasks();
            setTasks(fetchedTasks);
        } catch (err) {
            setError(err instanceof Error ? err.message : 'Failed to fetch tasks');
            console.error('Error fetching tasks:', err);
        } finally {
            setLoading(false);
        }
    }, []);
    
    // Create task
    const createTask = useCallback(async (taskData: CreateTaskRequest) => {
        try {
            setError(null);
            const newTask = await TaskService.createTask(taskData);
            setTasks(prevTasks => [newTask, ...prevTasks]);
        } catch (err) {
            const errorMessage = err instanceof Error ? err.message : 'Failed to create task';
            setError(errorMessage);
            throw new Error(errorMessage);
        }
    }, []);
    
    // Update task
    const updateTask = useCallback(async (id: number, updates: UpdateTaskRequest) => {
        try {
            setError(null);
            const updatedTask = await TaskService.updateTask(id, updates);
            setTasks(prevTasks => 
                prevTasks.map(task => 
                    task.id === id ? updatedTask : task
                )
            );
        } catch (err) {
            const errorMessage = err instanceof Error ? err.message : 'Failed to update task';
            setError(errorMessage);
            throw new Error(errorMessage);
        }
    }, []);
    
    // Delete task
    const deleteTask = useCallback(async (id: number) => {
        try {
            setError(null);
            await TaskService.deleteTask(id);
            setTasks(prevTasks => prevTasks.filter(task => task.id !== id));
        } catch (err) {
            const errorMessage = err instanceof Error ? err.message : 'Failed to delete task';
            setError(errorMessage);
            throw new Error(errorMessage);
        }
    }, []);
    
    // Refetch tasks (for manual refresh)
    const refetchTasks = useCallback(async () => {
        await fetchTasks();
    }, [fetchTasks]);
    
    // Fetch tasks on component mount
    useEffect(() => {
        fetchTasks();
    }, [fetchTasks]);
    
    return {
        tasks,
        loading,
        error,
        createTask,
        updateTask,
        deleteTask,
        refetchTasks,
    };
};
```

**Custom Hook Benefits:**
- **Reusability**: Multiple components can use the same state logic
- **Separation of concerns**: Components focus on UI, hooks handle data
- **Testability**: Easier to test business logic separately
- **Performance**: `useCallback` prevents unnecessary re-renders

### Step 13: Task Form Component (src/components/TaskForm.tsx)

```typescript
import React, { useState } from 'react';
import { CreateTaskRequest } from '../types/task';

interface TaskFormProps {
    onSubmit: (taskData: CreateTaskRequest) => Promise<void>;
    loading?: boolean;
}

export const TaskForm: React.FC<TaskFormProps> = ({ onSubmit, loading = false }) => {
    const [title, setTitle] = useState('');
    const [isSubmitting, setIsSubmitting] = useState(false);
    const [error, setError] = useState<string | null>(null);
    
    const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
        e.preventDefault();
        
        const trimmedTitle = title.trim();
        
        if (!trimmedTitle) {
            setError('Task title is required');
            return;
        }
        
        if (trimmedTitle.length > 200) {
            setError('Task title must be less than 200 characters');
            return;
        }
        
        try {
            setIsSubmitting(true);
            setError(null);
            
            await onSubmit({ title: trimmedTitle });
            setTitle(''); // Clear form on success
        } catch (err) {
            setError(err instanceof Error ? err.message : 'Failed to create task');
        } finally {
            setIsSubmitting(false);
        }
    };
    
    const handleTitleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        setTitle(e.target.value);
        if (error) setError(null); // Clear error when user starts typing
    };
    
    return (
        <div className="task-form-container">
            <h2>Add New Task</h2>
            
            <form onSubmit={handleSubmit} className="task-form">
                <div className="form-group">
                    <input
                        type="text"
                        value={title}
                        onChange={handleTitleChange}
                        placeholder="Enter task title..."
                        className={`task-input ${error ? 'error' : ''}`}
                        disabled={isSubmitting || loading}
                        maxLength={200}
                    />
                    <button
                        type="submit"
                        disabled={isSubmitting || loading || !title.trim()}
                        className="submit-button"
                    >
                        {isSubmitting ? 'Adding...' : 'Add Task'}
                    </button>
                </div>
                
                {error && (
                    <div className="error-message">
                        {error}
                    </div>
                )}
                
                <div className="form-info">
                    {title.length}/200 characters
                </div>
            </form>
        </div>
    );
};
```

### Step 14: Task Item Component (src/components/TaskItem.tsx)

```typescript
import React, { useState } from 'react';
import { Task } from '../types/task';

interface TaskItemProps {
    task: Task;
    onUpdate: (id: number, updates: { completed?: boolean; title?: string }) => Promise<void>;
    onDelete: (id: number) => Promise<void>;
}

export const TaskItem: React.FC<TaskItemProps> = ({ task, onUpdate, onDelete }) => {
    const [isEditing, setIsEditing] = useState(false);
    const [editTitle, setEditTitle] = useState(task.title);
    const [isUpdating, setIsUpdating] = useState(false);
    const [isDeleting, setIsDeleting] = useState(false);
    
    const handleCheckboxChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
        try {
            setIsUpdating(true);
            await onUpdate(task.id, { completed: e.target.checked });
        } catch (error) {
            console.error('Failed to update task:', error);
            // Revert checkbox state on error
            e.target.checked = !e.target.checked;
        } finally {
            setIsUpdating(false);
        }
    };
    
    const handleEditSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        
        const trimmedTitle = editTitle.trim();
        
        if (!trimmedTitle) {
            setEditTitle(task.title); // Reset to original
            setIsEditing(false);
            return;
        }
        
        if (trimmedTitle === task.title) {
            setIsEditing(false);
            return;
        }
        
        try {
            setIsUpdating(true);
            await onUpdate(task.id, { title: trimmedTitle });
            setIsEditing(false);
        } catch (error) {
            console.error('Failed to update task:', error);
            setEditTitle(task.title); // Reset to original
        } finally {
            setIsUpdating(false);
        }
    };
    
    const handleEditCancel = () => {
        setEditTitle(task.title);
        setIsEditing(false);
    };
    
    const handleDelete = async () => {
        if (window.confirm('Are you sure you want to delete this task?')) {
            try {
                setIsDeleting(true);
                await onDelete(task.id);
            } catch (error) {
                console.error('Failed to delete task:', error);
                setIsDeleting(false);
            }
        }
    };
    
    const formatDate = (dateString: string) => {
        return new Date(dateString).toLocaleDateString('en-US', {
            year: 'numeric',
            month: 'short',
            day: 'numeric',
            hour: '2-digit',
            minute: '2-digit'
        });
    };
    
    return (
        <div className={`task-item ${task.completed ? 'completed' : ''} ${isUpdating ? 'updating' : ''}`}>
            <div className="task-content">
                <input
                    type="checkbox"
                    checked={task.completed}
                    onChange={handleCheckboxChange}
                    disabled={isUpdating || isDeleting}
                    className="task-checkbox"
                />
                
                {isEditing ? (
                    <form onSubmit={handleEditSubmit} className="edit-form">
                        <input
                            type="text"
                            value={editTitle}
                            onChange={(e) => setEditTitle(e.target.value)}
                            className="edit-input"
                            disabled={isUpdating}
                            autoFocus
                            maxLength={200}
                        />
                        <div className="edit-buttons">
                            <button
                                type="submit"
                                disabled={isUpdating || !editTitle.trim()}
                                className="save-button"
                            >
                                Save
                            </button>
                            <button
                                type="button"
                                onClick={handleEditCancel}
                                disabled={isUpdating}
                                className="cancel-button"
                            >
                                Cancel
                            </button>
                        </div>
                    </form>
                ) : (
                    <div className="task-info">
                        <span
                            className="task-title"
                            onDoubleClick={() => setIsEditing(true)}
                            title="Double-click to edit"
                        >
                            {task.title}
                        </span>
                        <span className="task-date">
                            Created: {formatDate(task.createdAt)}
                        </span>
                    </div>
                )}
            </div>
            
            <div className="task-actions">
                {!isEditing && (
                    <button
                        onClick={() => setIsEditing(true)}
                        disabled={isUpdating || isDeleting}
                        className="edit-button"
                    >
                        Edit
                    </button>
                )}
                
                <button
                    onClick={handleDelete}
                    disabled={isUpdating || isDeleting}
                    className="delete-button"
                >
                    {isDeleting ? 'Deleting...' : 'Delete'}
                </button>
            </div>
        </div>
    );
};
```

### Step 15: Task List Component (src/components/TaskList.tsx)

```typescript
import React from 'react';
import { Task } from '../types/task';
import { TaskItem } from './TaskItem';

interface TaskListProps {
    tasks: Task[];
    loading: boolean;
    error: string | null;
    onUpdateTask: (id: number, updates: { completed?: boolean; title?: string }) => Promise<void>;
    onDeleteTask: (id: number) => Promise<void>;
    onRefresh: () => Promise<void>;
}

export const TaskList: React.FC<TaskListProps> = ({
    tasks,
    loading,
    error,
    onUpdateTask,
    onDeleteTask,
    onRefresh
}) => {
    const completedTasks = tasks.filter(task => task.completed);
    const incompleteTasks = tasks.filter(task => !task.completed);
    
    if (loading) {
        return (
            <div className="task-list-container">
                <div className="loading-spinner">
                    <div className="spinner"></div>
                    <p>Loading tasks...</p>
                </div>
            </div>
        );
    }
    
    if (error) {
        return (
            <div className="task-list-container">
                <div className="error-container">
                    <h3>Error Loading Tasks</h3>
                    <p>{error}</p>
                    <button onClick={onRefresh} className="retry-button">
                        Try Again
                    </button>
                </div>
            </div>
        );
    }
    
    return (
        <div className="task-list-container">
            <div className="task-list-header">
                <h2>Your Tasks</h2>
                <div className="task-stats">
                    <span>Total: {tasks.length}</span>
                    <span>Completed: {completedTasks.length}</span>
                    <span>Remaining: {incompleteTasks.length}</span>
                </div>
                <button onClick={onRefresh} className="refresh-button">
                    Refresh
                </button>
            </div>
            
            {tasks.length === 0 ? (
                <div className="no-tasks">
                    <h3>No tasks yet!</h3>
                    <p>Add your first task above to get started.</p>
                </div>
            ) : (
                <div className="task-sections">
                    {incompleteTasks.length > 0 && (
                        <div className="task-section">
                            <h3>Incomplete Tasks ({incompleteTasks.length})</h3>
                            <div className="task-list">
                                {incompleteTasks.map(task => (
                                    <TaskItem
                                        key={task.id}
                                        task={task}
                                        onUpdate={onUpdateTask}
                                        onDelete={onDeleteTask}
                                    />
                                ))}
                            </div>
                        </div>
                    )}
                    
                    {completedTasks.length > 0 && (
                        <div className="task-section">
                            <h3>Completed Tasks ({completedTasks.length})</h3>
                            <div className="task-list">
                                {completedTasks.map(task => (
                                    <TaskItem
                                        key={task.id}
                                        task={task}
                                        onUpdate={onUpdateTask}
                                        onDelete={onDeleteTask}
                                    />
                                ))}
                            </div>
                        </div>
                    )}
                </div>
            )}
        </div>
    );
};
```

### Step 16: Main App Component (src/App.tsx)

```typescript
import React from 'react';
import { TaskForm } from './components/TaskForm';
import { TaskList } from './components/TaskList';
import { useTasks } from './hooks/useTasks';
import './App.css';

function App() {
    const {
        tasks,
        loading,
        error,
        createTask,
        updateTask,
        deleteTask,
        refetchTasks
    } = useTasks();
    
    return (
        <div className="App">
            <header className="app-header">
                <h1>Task Manager</h1>
                <p>Manage your tasks efficiently</p>
            </header>
            
            <main className="app-main">
                <div className="container">
                    <TaskForm
                        onSubmit={createTask}
                        loading={loading}
                    />
                    
                    <TaskList
                        tasks={tasks}
                        loading={loading}
                        error={error}
                        onUpdateTask={updateTask}
                        onDeleteTask={deleteTask}
                        onRefresh={refetchTasks}
                    />
                </div>
            </main>
        </div>
    );
}

export default App;
```

### Step 17: App Styling (src/App.css)

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
        'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
        sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    background-color: #f5f7fa;
    color: #333;
    line-height: 1.6;
}

.App {
    min-height: 100vh;
}

.app-header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    text-align: center;
    padding: 3rem 0;
    margin-bottom: 2rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.app-header h1 {
    font-size: 3rem;
    font-weight: 300;
    margin-bottom: 0.5rem;
}

.app-header p {
    font-size: 1.2rem;
    opacity: 0.9;
}

.app-main {
    padding-bottom: 2rem;
}

.container {
    max-width: 1000px;
    margin: 0 auto;
    padding: 0 2rem;
}

/* Task Form Styles */
.task-form-container {
    background: white;
    padding: 2rem;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    margin-bottom: 2rem;
}

.task-form-container h2 {
    margin-bottom: 1.5rem;
    color: #4a5568;
    font-weight: 600;
}

.task-form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.form-group {
    display: flex;
    gap: 1rem;
}

.task-input {
    flex: 1;
    padding: 0.875rem;
    border: 2px solid #e2e8f0;
    border-radius: 8px;
    font-size: 1rem;
    transition: border-color 0.2s, box-shadow 0.2s;
}

.task-input:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.task-input.error {
    border-color: #e53e3e;
}

.submit-button {
    padding: 0.875rem 1.5rem;
    background: #667eea;
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s, transform 0.1s;
    white-space: nowrap;
}

.submit-button:hover:not(:disabled) {
    background: #5a67d8;
    transform: translateY(-1px);
}

.submit-button:active {
    transform: translateY(0);
}

.submit-button:disabled {
    background: #a0aec0;
    cursor: not-allowed;
    transform: none;
}

.error-message {
    color: #e53e3e;
    font-size: 0.875rem;
    padding: 0.5rem;
    background: #fed7d7;
    border-radius: 4px;
    border-left: 4px solid #e53e3e;
}

.form-info {
    font-size: 0.875rem;
    color: #718096;
    text-align: right;
}

/* Task List Styles */
.task-list-container {
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    overflow: hidden;
}

.task-list-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1.5rem 2rem;
    background: #f8f9fa;
    border-bottom: 1px solid #e2e8f0;
}

.task-list-header h2 {
    color: #4a5568;
    font-weight: 600;
}

.task-stats {
    display: flex;
    gap: 1rem;
    font-size: 0.875rem;
    color: #718096;
}

.task-stats span {
    padding: 0.25rem 0.5rem;
    background: white;
    border-radius: 4px;
    border: 1px solid #e2e8f0;
}

.refresh-button {
    padding: 0.5rem 1rem;
    background: #48bb78;
    color: white;
    border: none;
    border-radius: 6px;
    font-size: 0.875rem;
    cursor: pointer;
    transition: background-color 0.2s;
}

.refresh-button:hover {
    background: #38a169;
}

/* Loading Spinner */
.loading-spinner {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 4rem 2rem;
    color: #718096;
}

.spinner {
    width: 40px;
    height: 40px;
    border: 4px solid #e2e8f0;
    border-top: 4px solid #667eea;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 1rem;
}

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

/* Error Container */
.error-container {
    text-align: center;
    padding: 4rem 2rem;
    color: #e53e3e;
}

.error-container h3 {
    margin-bottom: 1rem;
    font-size: 1.5rem;
}

.error-container p {
    margin-bottom: 2rem;
    color: #718096;
}

.retry-button {
    padding: 0.75rem 1.5rem;
    background: #e53e3e;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background-color 0.2s;
}

.retry-button:hover {
    background: #c53030;
}

/* No Tasks */
.no-tasks {
    text-align: center;
    padding: 4rem 2rem;
    color: #718096;
}

.no-tasks h3 {
    margin-bottom: 1rem;
    color: #4a5568;
}

/* Task Sections */
.task-sections {
    padding: 0 2rem 2rem;
}

.task-section {
    margin-bottom: 2rem;
}

.task-section:last-child {
    margin-bottom: 0;
}

.task-section h3 {
    margin-bottom: 1rem;
    color: #4a5568;
    font-size: 1.1rem;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.task-section h3::before {
    content: '';
    width: 4px;
    height: 20px;
    background: #667eea;
    border-radius: 2px;
}

.task-list {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
}

/* Task Item */
.task-item {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    padding: 1.5rem;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    background: #fafbfc;
    transition: all 0.2s;
    position: relative;
}

.task-item:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    transform: translateY(-1px);
}

.task-item.completed {
    opacity: 0.7;
    background: #f7fafc;
}

.task-item.updating {
    opacity: 0.6;
    pointer-events: none;
}

.task-content {
    display: flex;
    align-items: flex-start;
    gap: 1rem;
    flex: 1;
}

.task-checkbox {
    width: 20px;
    height: 20px;
    margin-top: 2px;
    cursor: pointer;
    accent-color: #667eea;
}

.task-info {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    flex: 1;
}

.task-title {
    font-size: 1.1rem;
    font-weight: 500;
    color: #2d3748;
    cursor: pointer;
}

.task-item.completed .task-title {
    text-decoration: line-through;
    color: #a0aec0;
}

.task-date {
    font-size: 0.875rem;
    color: #718096;
}

.task-actions {
    display: flex;
    gap: 0.5rem;
    align-items: flex-start;
}

.edit-button, .delete-button {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    font-size: 0.875rem;
    cursor: pointer;
    transition: all 0.2s;
    font-weight: 500;
}

.edit-button {
    background: #edf2f7;
    color: #4a5568;
}

.edit-button:hover:not(:disabled) {
    background: #e2e8f0;
    color: #2d3748;
}

.delete-button {
    background: #fed7d7;
    color: #c53030;
}

.delete-button:hover:not(:disabled) {
    background: #feb2b2;
    color: #9b2c2c;
}

.edit-button:disabled, .delete-button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

/* Edit Form */
.edit-form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    width: 100%;
}

.edit-input {
    padding: 0.75rem;
    border: 2px solid #e2e8f0;
    border-radius: 4px;
    font-size: 1rem;
    font-weight: 500;
}

.edit-input:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.edit-buttons {
    display: flex;
    gap: 0.5rem;
}

.save-button, .cancel-button {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    font-size: 0.875rem;
    cursor: pointer;
    font-weight: 500;
    transition: all 0.2s;
}

.save-button {
    background: #48bb78;
    color: white;
}

.save-button:hover:not(:disabled) {
    background: #38a169;
}

.save-button:disabled {
    background: #a0aec0;
    cursor: not-allowed;
}

.cancel-button {
    background: #edf2f7;
    color: #4a5568;
}

.cancel-button:hover:not(:disabled) {
    background: #e2e8f0;
}

/* Responsive Design */
@media (max-width: 768px) {
    .container {
        padding: 0 1rem;
    }
    
    .app-header {
        padding: 2rem 0;
    }
    
    .app-header h1 {
        font-size: 2rem;
    }
    
    .task-form-container, .task-list-container {
        padding: 1.5rem;
    }
    
    .form-group {
        flex-direction: column;
    }
    
    .task-list-header {
        flex-direction: column;
        gap: 1rem;
        align-items: flex-start;
    }
    
    .task-stats {
        flex-wrap: wrap;
    }
    
    .task-item {
        flex-direction: column;
        gap: 1rem;
        align-items: stretch;
    }
    
    .task-content {
        align-items: flex-start;
    }
    
    .task-actions {
        align-self: flex-end;
    }
    
    .edit-buttons {
        justify-content: flex-end;
    }
}

@media (max-width: 480px) {
    .app-header h1 {
        font-size: 1.75rem;
    }
    
    .app-header p {
        font-size: 1rem;
    }
    
    .task-form-container, .task-list-container {
        padding: 1rem;
    }
    
    .task-stats {
        font-size: 0.8rem;
    }
    
    .task-stats span {
        padding: 0.2rem 0.4rem;
    }
}
```

### Step 18: Run Both Applications

**IMPORTANT**: You need TWO separate terminal windows/tabs for this to work.

**Terminal 1 - Backend:**
```bash
cd task-manager-separated/backend
npm run dev
```

You should see:
```
Backend server running on http://localhost:5000
Health check: http://localhost:5000/health
Press Ctrl+C to stop the server
```

**Terminal 2 - Frontend:**
```bash
cd task-manager-separated/frontend
npm start
```

You should see:
```
Local:            http://localhost:3000
On Your Network:  http://192.168.x.x:3000
```

The backend will run on `http://localhost:5000` and the frontend on `http://localhost:3000`.

### Step 19: Test the Separated Application

1. **Test Backend First:**
   - Visit `http://localhost:5000/health` - should show server status
   - Visit `http://localhost:5000/api/tasks` - should show JSON tasks

2. **Test Frontend:**
   - Open `http://localhost:3000` in your browser
   - Try adding, editing, completing, and deleting tasks
   - Open browser developer tools (F12) to see API requests in the Network tab

3. **Test Integration:**
   - Add a task in the frontend
   - Check that it appears in the backend API
   - Refresh the frontend page to ensure data persists

### Troubleshooting Common Issues

**CORS Errors:**
- Ensure backend is running on port 5000
- Check browser console for specific CORS error messages
- Verify frontend is accessing `http://localhost:3000`

**"Network Error" in frontend:**
- Ensure backend server is running
- Check backend console for error messages
- Verify API URLs in `api.ts` match backend server

**TypeScript Errors:**
- Run `npm run build` in frontend to check for type errors
- Ensure all imports are correct
- Check that interface definitions match API responses

**Port Conflicts:**
- If port 3000 is in use, React will offer to run on a different port
- If port 5000 is in use, change PORT in backend server.js

**React App Won't Start:**
- Delete `node_modules` and `package-lock.json`
- Run `npm install` again
- Ensure you're using Node.js 14+ (`node --version`)

**Data Not Persisting:**
- This is expected - data is stored in memory
- Data will reset when backend server restarts
- This is normal for this tutorial

---

## Comparison and Best Practices

### Architecture Comparison

| Aspect | Monolithic | Separated |
|--------|------------|-----------|
| **Deployment** | Single unit (1 server) | Two deployments (2 servers) |
| **Development** | Simpler setup (1 project) | More complex setup (2 projects) |
| **Scaling** | Scale entire app together | Scale components independently |
| **Technology** | Single stack (Node.js + templates) | Different technologies possible |
| **Network** | No network calls internally | HTTP requests between services |
| **Debugging** | Easier to trace (single codebase) | Requires distributed debugging |
| **Team Structure** | Best for small teams (1-3 people) | Better for larger teams with specialization |
| **Data Flow** | Direct function calls | API requests over HTTP |
| **Initial Setup Time** | ~30 minutes | ~1-2 hours |
| **Learning Curve** | Gentler (traditional web dev) | Steeper (modern full-stack) |

### Performance Comparison

**Monolithic Performance:**
- ✅ Faster initial page loads (server-side rendering)
- ✅ No network latency between frontend and backend logic
- ✅ Single database connection pool
- ✅ Easier caching strategies
- ❌ Entire app must scale together
- ❌ Single point of failure

**Separated Performance:**
- ✅ Better caching of static assets (frontend)
- ✅ Can use CDN for frontend distribution
- ✅ Independent scaling of services
- ✅ Frontend caching in browser
- ❌ Network latency between services
- ❌ More complex deployment pipeline

### When to Use Each Approach

**Choose Monolithic When:**
- 👥 Small to medium applications (< 10,000 users)
- 👨‍💻 Small development team (1-3 developers)
- 🚀 Rapid prototyping or MVP development
- 📦 Simple deployment requirements
- 🔗 All business logic is closely related
- ⚡ Performance is critical (no network overhead)
- 💰 Limited hosting budget
- 📚 Team is learning web development

**Choose Separated When:**
- 🏢 Large, complex applications (> 10,000 users)
- 👥 Multiple teams working independently
- 📱 Multiple frontend clients (web, mobile, desktop)
- 🔄 Different scaling requirements for frontend/backend
- 🛠️ Microservices architecture plans
- 🎯 Need for technology flexibility
- 🌐 API will be consumed by third parties
- 🔒 Complex authentication/authorization requirements

### Real-World Examples

**Monolithic Success Stories:**
- **Basecamp**: Rails monolith serving millions of users
- **Shopify**: Started monolithic, evolved gradually
- **GitLab**: Large Rails monolith with great performance

**Separated Architecture Success Stories:**
- **Netflix**: Microservices with React frontends
- **Uber**: Separated mobile apps with API backends
- **Airbnb**: React frontend with Rails/Node.js APIs

### Migration Path

Many successful companies follow this pattern:
1. **Start Monolithic**: Build MVP quickly, validate product-market fit
2. **Identify Pain Points**: Monitor performance, team productivity
3. **Extract Services**: Gradually separate high-load or independent features
4. **Full Separation**: When team size and complexity justify the overhead

**Migration Example:**
```
Phase 1: Monolithic (0-6 months)
├── All features in one app
├── Fast development
└── Quick deployment

Phase 2: API Layer (6-12 months)
├── Add API endpoints to monolith
├── Create mobile app consuming API
└── Frontend still server-rendered

Phase 3: Frontend Separation (12-18 months)
├── Extract frontend to React/Vue
├── Keep backend as API
└── Better user experience

Phase 4: Service Extraction (18+ months)
├── Extract user management service
├── Extract payment service
└── Microservices architecture
```

### Best Practices for Both Approaches

#### Monolithic Best Practices:
1. **Organize by features**: Group related files together
2. **Separate concerns**: Keep business logic separate from presentation
3. **Use middleware**: Leverage Express middleware for common functionality
4. **Environment configuration**: Use environment variables for different settings
5. **Error handling**: Implement comprehensive error handling
6. **Security**: Validate inputs, sanitize outputs, use HTTPS

#### Separated Architecture Best Practices:
1. **API design**: Follow RESTful principles, use consistent response formats
2. **Error handling**: Implement proper HTTP status codes and error messages
3. **CORS configuration**: Properly configure cross-origin requests
4. **Type safety**: Use TypeScript for better development experience
5. **State management**: Use proper state management (React hooks, Redux, etc.)
6. **API documentation**: Document your API endpoints
7. **Testing**: Test components and API endpoints separately

### Performance Considerations

**Monolithic Performance:**
- Faster initial page loads (server-side rendering)
- No network latency between frontend and backend
- Single database connection pool
- Easier caching strategies

**Separated Performance:**
- Better caching of static assets (frontend)
- Can use CDN for frontend distribution
- Independent scaling of services
- Potential network latency between services

### Security Considerations

**Monolithic Security:**
- Single authentication system
- No cross-service communication vulnerabilities
- Easier to implement security policies
- Single SSL certificate

**Separated Security:**
- Need CORS configuration
- API authentication required
- Multiple attack surfaces
- Need to secure API endpoints
- Potential for JWT tokens or session management

### Conclusion

Both architectural approaches have their place in modern web development. The choice depends on your specific requirements:

- **Start with monolithic** for simpler projects, small teams, or when you need to move quickly
- **Move to separated architecture** as your application grows, when you need multiple frontends, or when different parts of your application have different scaling requirements

The most important thing is to understand both approaches and choose the one that best fits your project's current and future needs. Many successful applications start monolithic and evolve to separated architecture as they grow.

### Final Checklist for Students

Before considering this tutorial complete, ensure you can:

**Monolithic Application:**
- [ ] Create and run the monolithic server successfully
- [ ] Add new tasks through the web interface
- [ ] Mark tasks as complete/incomplete
- [ ] Delete tasks
- [ ] Understand how EJS templates work
- [ ] Explain the difference between server-side and client-side code

**Separated Application:**
- [ ] Run both backend and frontend servers simultaneously
- [ ] See tasks loading from the API in browser dev tools
- [ ] Add, edit, and delete tasks through React interface
- [ ] Understand how API calls work between frontend and backend
- [ ] Explain TypeScript benefits in the frontend
- [ ] Handle CORS and cross-origin requests

**Understanding:**
- [ ] Explain when to choose each architecture
- [ ] Identify the trade-offs between approaches
- [ ] Understand the data flow in both architectures
- [ ] Know how to debug issues in both setups

### Next Steps for Learning

1. **Add a Database**: Replace in-memory storage with MongoDB or PostgreSQL
   ```bash
   npm install mongoose  # For MongoDB
   # or
   npm install pg        # For PostgreSQL
   ```

2. **Add Authentication**: Implement user login and authorization
   ```bash
   npm install jsonwebtoken bcryptjs
   ```

3. **Add Real-time Features**: Use WebSockets for live updates
   ```bash
   npm install socket.io
   ```

4. **Add Testing**: Write unit and integration tests
   ```bash
   npm install --save-dev jest supertest @testing-library/react
   ```

5. **Add Deployment**: Deploy to cloud platforms
   - Backend: Heroku, Railway, DigitalOcean
   - Frontend: Vercel, Netlify, GitHub Pages
   - Database: MongoDB Atlas, ElephantSQL

6. **Add Advanced Features**:
   - File uploads with Multer
   - Email notifications with Nodemailer
   - Caching with Redis
   - API rate limiting
   - Input sanitization and validation
   - Logging with Winston
   - Environment configuration with dotenv

7. **Add Monitoring**: Implement logging and error tracking
   ```bash
   npm install winston morgan  # Backend logging
   npm install @sentry/react   # Frontend error tracking
   ```

8. **Performance Optimization**:
   - Frontend: Code splitting, lazy loading, memoization
   - Backend: Query optimization, caching, compression
   - Database: Indexing, connection pooling

### Additional Resources

**Documentation:**
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

**Tools:**
- [Postman](https://www.postman.com/) - API testing
- [VS Code](https://code.visualstudio.com/) - Code editor with great extensions
- [React Developer Tools](https://react.dev/learn/react-developer-tools) - Browser extension
- [MongoDB Compass](https://www.mongodb.com/products/compass) - Database GUI

This tutorial provides a solid foundation for understanding both monolithic and separated architectures. Practice building applications with both approaches to gain hands-on experience and develop your full-stack skills!
