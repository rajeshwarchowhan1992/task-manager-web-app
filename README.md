# Task Manager Web App 🚀

A modern, full-stack Task Manager application demonstrating best practices in web development. Built with React for the frontend and Node.js/Express for the backend, this project serves as an excellent example of full-stack JavaScript development.

## 📋 Project Description

This Task Manager Web App is a comprehensive solution for managing tasks with user authentication, real-time updates, and a clean, intuitive interface. It showcases the integration of modern frontend and backend technologies to create a production-ready application.

## ✨ Key Features

- **Task Creation**: Create new tasks with titles, descriptions, priorities, and due dates
- **Status Tracking**: Track task progress with customizable statuses (To Do, In Progress, Completed)
- **User Authentication**: Secure JWT-based authentication system with login and registration
- **Task Filtering**: Filter tasks by status, priority, and date
- **Responsive Design**: Mobile-first design that works seamlessly across all devices
- **Real-time Updates**: Instant UI updates when tasks are modified
- **Task Priority Levels**: Categorize tasks as Low, Medium, or High priority
- **Search Functionality**: Quick search through your tasks

## 🛠️ Tech Stack

### Frontend
- **React 18**: Modern UI development with hooks
- **React Router**: Client-side routing
- **Axios**: HTTP client for API requests
- **CSS3**: Responsive styling with Flexbox/Grid
- **Context API**: State management

### Backend
- **Node.js**: JavaScript runtime
- **Express.js**: Web application framework
- **MongoDB**: NoSQL database for data storage
- **Mongoose**: MongoDB object modeling
- **JWT**: JSON Web Tokens for authentication
- **bcrypt**: Password hashing
- **dotenv**: Environment variable management

## 📦 Installation & Setup

### Prerequisites
- Node.js (v14 or higher)
- npm or yarn
- MongoDB (local or MongoDB Atlas account)

### Backend Setup

```bash
# Clone the repository
git clone https://github.com/rajeshwarchowhan1992/task-manager-web-app.git
cd task-manager-web-app

# Navigate to backend directory
cd backend

# Install dependencies
npm install

# Create .env file
touch .env

# Add environment variables to .env:
# PORT=5000
# MONGODB_URI=your_mongodb_connection_string
# JWT_SECRET=your_jwt_secret_key

# Start the backend server
npm start
```

### Frontend Setup

```bash
# Navigate to frontend directory (from project root)
cd frontend

# Install dependencies
npm install

# Create .env file
touch .env

# Add environment variables to .env:
# REACT_APP_API_URL=http://localhost:5000/api

# Start the development server
npm start
```

The application will be available at `http://localhost:3000`

## 📁 Project Structure

```
task-manager-web-app/
├── backend/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   ├── authController.js
│   │   └── taskController.js
│   ├── middleware/
│   │   └── authMiddleware.js
│   ├── models/
│   │   ├── User.js
│   │   └── Task.js
│   ├── routes/
│   │   ├── authRoutes.js
│   │   └── taskRoutes.js
│   ├── .env
│   ├── server.js
│   └── package.json
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Auth/
│   │   │   ├── Dashboard/
│   │   │   ├── TaskList/
│   │   │   └── TaskForm/
│   │   ├── context/
│   │   │   └── AuthContext.js
│   │   ├── services/
│   │   │   └── api.js
│   │   ├── App.js
│   │   └── index.js
│   ├── .env
│   └── package.json
└── README.md
```

## 💻 Sample Code

### Backend - Task Model (models/Task.js)

```javascript
const mongoose = require('mongoose');

const taskSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Task title is required'],
    trim: true,
    maxlength: [100, 'Title cannot exceed 100 characters']
  },
  description: {
    type: String,
    trim: true,
    maxlength: [500, 'Description cannot exceed 500 characters']
  },
  status: {
    type: String,
    enum: ['todo', 'in-progress', 'completed'],
    default: 'todo'
  },
  priority: {
    type: String,
    enum: ['low', 'medium', 'high'],
    default: 'medium'
  },
  dueDate: {
    type: Date
  },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  }
}, {
  timestamps: true
});

module.exports = mongoose.model('Task', taskSchema);
```

### Backend - Task Controller (controllers/taskController.js)

```javascript
const Task = require('../models/Task');

// Get all tasks for authenticated user
exports.getTasks = async (req, res) => {
  try {
    const tasks = await Task.find({ user: req.user.id })
      .sort({ createdAt: -1 });
    res.json(tasks);
  } catch (error) {
    res.status(500).json({ message: 'Server error', error: error.message });
  }
};

// Create new task
exports.createTask = async (req, res) => {
  try {
    const { title, description, status, priority, dueDate } = req.body;
    
    const task = new Task({
      title,
      description,
      status,
      priority,
      dueDate,
      user: req.user.id
    });
    
    const savedTask = await task.save();
    res.status(201).json(savedTask);
  } catch (error) {
    res.status(400).json({ message: 'Error creating task', error: error.message });
  }
};

// Update task
exports.updateTask = async (req, res) => {
  try {
    const task = await Task.findOneAndUpdate(
      { _id: req.params.id, user: req.user.id },
      req.body,
      { new: true, runValidators: true }
    );
    
    if (!task) {
      return res.status(404).json({ message: 'Task not found' });
    }
    
    res.json(task);
  } catch (error) {
    res.status(400).json({ message: 'Error updating task', error: error.message });
  }
};

// Delete task
exports.deleteTask = async (req, res) => {
  try {
    const task = await Task.findOneAndDelete({
      _id: req.params.id,
      user: req.user.id
    });
    
    if (!task) {
      return res.status(404).json({ message: 'Task not found' });
    }
    
    res.json({ message: 'Task deleted successfully' });
  } catch (error) {
    res.status(500).json({ message: 'Server error', error: error.message });
  }
};
```

### Frontend - Task Component (components/TaskList/TaskItem.jsx)

```javascript
import React from 'react';
import './TaskItem.css';

const TaskItem = ({ task, onUpdate, onDelete }) => {
  const handleStatusChange = (newStatus) => {
    onUpdate(task._id, { ...task, status: newStatus });
  };

  const getPriorityClass = () => {
    return `priority-${task.priority}`;
  };

  const getStatusClass = () => {
    return `status-${task.status}`;
  };

  return (
    <div className={`task-item ${getStatusClass()} ${getPriorityClass()}`}>
      <div className="task-header">
        <h3>{task.title}</h3>
        <span className="priority-badge">{task.priority}</span>
      </div>
      
      <p className="task-description">{task.description}</p>
      
      <div className="task-meta">
        {task.dueDate && (
          <span className="due-date">
            Due: {new Date(task.dueDate).toLocaleDateString()}
          </span>
        )}
      </div>
      
      <div className="task-actions">
        <select 
          value={task.status} 
          onChange={(e) => handleStatusChange(e.target.value)}
          className="status-select"
        >
          <option value="todo">To Do</option>
          <option value="in-progress">In Progress</option>
          <option value="completed">Completed</option>
        </select>
        
        <button 
          onClick={() => onDelete(task._id)} 
          className="delete-btn"
        >
          Delete
        </button>
      </div>
    </div>
  );
};

export default TaskItem;
```

### Frontend - API Service (services/api.js)

```javascript
import axios from 'axios';

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000/api';

// Create axios instance
const api = axios.create({
  baseURL: API_URL,
  headers: {
    'Content-Type': 'application/json'
  }
});

// Add token to requests
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Auth APIs
export const authAPI = {
  register: (userData) => api.post('/auth/register', userData),
  login: (credentials) => api.post('/auth/login', credentials),
  getCurrentUser: () => api.get('/auth/me')
};

// Task APIs
export const taskAPI = {
  getAllTasks: () => api.get('/tasks'),
  createTask: (taskData) => api.post('/tasks', taskData),
  updateTask: (id, taskData) => api.put(`/tasks/${id}`, taskData),
  deleteTask: (id) => api.delete(`/tasks/${id}`)
};

export default api;
```

## 🚀 API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login user
- `GET /api/auth/me` - Get current user

### Tasks
- `GET /api/tasks` - Get all tasks for authenticated user
- `POST /api/tasks` - Create new task
- `PUT /api/tasks/:id` - Update task
- `DELETE /api/tasks/:id` - Delete task

## 🎯 Use Cases

This project is perfect for:
- **Learning Full-Stack Development**: Understand how frontend and backend work together
- **Portfolio Projects**: Showcase your skills to potential employers
- **Interview Preparation**: Discuss architecture, design patterns, and best practices
- **Starting Point**: Use as a template for building similar applications
- **Teaching**: Great example for teaching modern web development

## 🔐 Security Features

- Password hashing with bcrypt
- JWT-based authentication
- Protected API routes
- Input validation and sanitization
- Environment variable management
- CORS configuration

## 🌟 Future Enhancements

- [ ] Add task categories/tags
- [ ] Implement drag-and-drop functionality
- [ ] Add email notifications for due dates
- [ ] Implement task sharing between users
- [ ] Add file attachments to tasks
- [ ] Dark mode support
- [ ] Export tasks to CSV/PDF
- [ ] Add task analytics dashboard

## 📝 License

This project is licensed under the MIT License - feel free to use it for learning and portfolio purposes.

## 👨‍💻 Author

**Rajeshwar Chowhan**
- GitHub: [@rajeshwarchowhan1992](https://github.com/rajeshwarchowhan1992)

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the issues page.

## ⭐ Show your support

Give a ⭐️ if this project helped you!

---

**Note**: This is a demonstration project showcasing full-stack development skills. It includes sample code and best practices for building modern web applications with React and Node.js/Express.
