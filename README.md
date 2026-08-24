# Week 9: Dynamic Portfolio with Context & API Integration
## Transform Your Portfolio into a Data-Driven Application

### 🎯 Learning Objectives
By completing this project, you will:
- Implement useEffect for data fetching with proper cleanup
- Use Context API to manage global theme state across components
- Handle async/await and promises for API communication
- Implement loading and error states professionally
- Prevent race conditions with effect cleanup
- Share state globally without prop drilling

### 📚 Prerequisites
Before starting, you should have:
- Completed Week 9 lecture on React Effects, Context API & Data Fetching
- Reviewed prep work chapters: React Front To Back Chapters 6-7 (useEffect, Context & APIs)
- Your code from Week 8 React State project (recommended)
- Understanding of useState, component decomposition, and event handling

### 🏗️ Project Overview
This week, you'll elevate your portfolio from a static React application to a dynamic, data-driven platform. You'll fetch project data from a REST API, implement a global theme system using Context API, and handle all the edge cases that come with asynchronous operations - loading states, error handling, and race condition prevention.

### 🚀 Getting Started

**First:** Accept the GitHub Classroom assignment link provided by your instructor. This will create a new repository with the Week 9 starter code.

1. **Clone your new repository:**
   ```bash
   git clone [your-github-classroom-repo-url]
   cd [your-repo-name]
   ```

2. **Choose your path:**

#### Option A: Continue from Week 8 (Recommended)
If you completed the Week 8 project and want to build upon it:

1. **Copy your Week 8 src folder** into the starter repository
2. **Keep the starter's package.json** - it includes json-server for our mock API
3. **Install dependencies:**
   ```bash
   npm install
   ```
4. **Start both the dev server AND the mock API:**
   ```bash
   npm run dev:all
   ```
   This starts both Vite (port 5173) and json-server (port 3001)

#### Option B: Fresh Start
If starting fresh, use the provided starter files:

1. **Install dependencies:**
   ```bash
   npm install
   ```
2. **Start both servers:**
   ```bash
   npm run dev:all
   ```
3. **Open your browser** to `http://localhost:5173`

### 📝 Step-by-Step Instructions

#### Part 1: Understanding the Architecture (10 minutes)

**What we're building:**
- A **Theme Context** that manages dark/light mode globally
- **Dynamic project loading** from a REST API
- **Professional error handling** with user-friendly messages
- **Loading states** while data fetches

**The data flow:**
```
API (json-server) → useEffect fetch → Context state → Components
                         ↓
                    Loading/Error states
```

**Key files you'll work with:**
- `src/context/ThemeContext.jsx` - Global theme management
- `src/context/ProjectsContext.jsx` - Projects data and API logic
- `src/components/Projects.jsx` - Display with loading/error states
- `src/data/db.json` - Mock API database

#### Part 2: Setting Up the Theme Context (15 minutes)

**Step 1: Create the Theme Context**

Create `src/context/ThemeContext.jsx`:

```jsx
import { createContext, useState, useContext, useEffect } from 'react';

// Create the context
const ThemeContext = createContext();

// Custom hook for using theme
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

// Provider component
export const ThemeProvider = ({ children }) => {
  // TODO: Initialize theme from localStorage or default to 'light'
  const [theme, setTheme] = useState('light');

  // TODO: Add useEffect to apply theme class to document.body
  // and save to localStorage when theme changes

  // TODO: Create toggleTheme function

  const value = {
    theme,
    toggleTheme: () => {} // TODO: Implement
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

**What's happening here:**
- Context holds global state that any component can access
- No more passing theme through every component level
- useEffect syncs theme with DOM and localStorage
- Custom hook provides clean API for components

**Common mistakes to avoid:**
- Forgetting to wrap app with Provider
- Not checking localStorage on initial load
- Missing cleanup in useEffect

**Step 2: Wrap your app with the Theme Provider**

In `src/main.jsx`:
```jsx
import { ThemeProvider } from './context/ThemeContext';

// Wrap your App component
<ThemeProvider>
  <App />
</ThemeProvider>
```

**Step 3: Use theme in your Navbar**

Update `src/components/Navbar.jsx` to add a theme toggle button that uses the context.

#### Part 3: Creating the Projects Context with API Integration (20 minutes)

**Step 1: Create the Projects Context**

Create `src/context/ProjectsContext.jsx`:

```jsx
import { createContext, useState, useContext, useEffect } from 'react';

const ProjectsContext = createContext();

export const useProjects = () => {
  const context = useContext(ProjectsContext);
  if (!context) {
    throw new Error('useProjects must be used within ProjectsProvider');
  }
  return context;
};

export const ProjectsProvider = ({ children }) => {
  const [projects, setProjects] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // TODO: Implement useEffect to fetch projects
  useEffect(() => {
    // TODO: Create async function to fetch data
    // TODO: Handle loading states
    // TODO: Handle errors properly
    // TODO: Implement cleanup for race conditions
  }, []);

  // TODO: Create addProject function for POST requests
  // TODO: Create updateProject function for PUT requests
  // TODO: Create deleteProject function for DELETE requests

  const value = {
    projects,
    loading,
    error,
    // Add your functions here
  };

  return (
    <ProjectsContext.Provider value={value}>
      {children}
    </ProjectsContext.Provider>
  );
};
```

**What's happening here:**
- Projects are fetched once when context mounts
- Loading/error states provide user feedback
- Cleanup prevents race conditions if component unmounts
- CRUD operations update both API and local state

**Step 2: Implement the fetch with async/await**

```jsx
useEffect(() => {
  let isCancelled = false;

  const fetchProjects = async () => {
    try {
      setLoading(true);
      setError(null);

      const response = await fetch('http://localhost:3001/projects');

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();

      // Only update if component is still mounted
      if (!isCancelled) {
        setProjects(data);
        setLoading(false);
      }
    } catch (err) {
      if (!isCancelled) {
        setError(err.message);
        setLoading(false);
      }
    }
  };

  fetchProjects();

  // Cleanup function
  return () => {
    isCancelled = true;
  };
}, []);
```

**Race condition prevention:**
- The `isCancelled` flag prevents state updates after unmount
- This prevents the "Can't update unmounted component" error
- Critical for production applications

#### Part 4: Updating Projects Component for Dynamic Data (15 minutes)

**Step 1: Use the Projects Context**

Update `src/components/Projects.jsx`:

```jsx
import { useProjects } from '../context/ProjectsContext';
import ProjectCard from './ProjectCard';
import LoadingSpinner from './LoadingSpinner';
import ErrorMessage from './ErrorMessage';

const Projects = () => {
  const { projects, loading, error } = useProjects();
  const [filter, setFilter] = useState('all');

  // Handle loading state
  if (loading) {
    return (
      <section id="projects" className="projects">
        <div className="container">
          <h2>Projects</h2>
          <LoadingSpinner />
        </div>
      </section>
    );
  }

  // Handle error state
  if (error) {
    return (
      <section id="projects" className="projects">
        <div className="container">
          <h2>Projects</h2>
          <ErrorMessage message={error} onRetry={() => window.location.reload()} />
        </div>
      </section>
    );
  }

  // Filter projects based on selected category
  const filteredProjects = filter === 'all'
    ? projects
    : projects.filter(p => p.category === filter);

  // TODO: Render the filtered projects
};
```

**Step 2: Create Loading and Error Components**

These provide professional user feedback during async operations.

#### Part 5: Implementing CRUD Operations (20 minutes)

**Step 1: Add a new project form**

Create a form component that uses the `addProject` function from context:

```jsx
const ProjectForm = () => {
  const { addProject } = useProjects();
  const [formData, setFormData] = useState({
    title: '',
    description: '',
    category: 'web',
    imageUrl: '',
    githubUrl: '',
    liveUrl: ''
  });

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
      await addProject(formData);
      // Clear form or show success message
      setFormData({...}); // Reset
    } catch (error) {
      // Handle error - maybe show toast notification
      console.error('Failed to add project:', error);
    }
  };

  // TODO: Implement form JSX
};
```

**Step 2: Implement the addProject function in context**

```jsx
const addProject = async (projectData) => {
  try {
    const response = await fetch('http://localhost:3001/projects', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        ...projectData,
        id: Date.now(), // Simple ID generation
        createdAt: new Date().toISOString()
      })
    });

    if (!response.ok) {
      throw new Error('Failed to add project');
    }

    const newProject = await response.json();

    // Optimistically update local state
    setProjects(prev => [...prev, newProject]);

    return newProject;
  } catch (error) {
    // Re-throw to let component handle it
    throw error;
  }
};
```

#### Part 6: Advanced useEffect Patterns (15 minutes)

**Implementing search with debouncing:**

```jsx
const ProjectSearch = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedTerm, setDebouncedTerm] = useState('');
  const { projects } = useProjects();

  // Debounce search term
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedTerm(searchTerm);
    }, 500); // Wait 500ms after user stops typing

    return () => clearTimeout(timer); // Cleanup
  }, [searchTerm]);

  // Filter projects based on debounced term
  const searchResults = projects.filter(project =>
    project.title.toLowerCase().includes(debouncedTerm.toLowerCase()) ||
    project.description.toLowerCase().includes(debouncedTerm.toLowerCase())
  );

  // TODO: Render search input and results
};
```

**What's happening:**
- Debouncing prevents API calls on every keystroke
- Cleanup function cancels pending timers
- Improves performance and user experience

### 🎨 Customization Ideas
- Add animations when projects load (staggered fade-in)
- Implement infinite scroll for projects
- Add project categories with icon badges
- Create a detailed project modal with image gallery
- Add "Add to Favorites" with localStorage persistence

### 🏆 Challenge Extensions
- **Intermediate:** Implement pagination for projects list
- **Advanced:** Add real-time updates using WebSockets
- **Graduate (253A):** Implement optimistic UI updates with rollback on error

### 🐛 Troubleshooting Guide

**"Cannot read properties of undefined"**
- Make sure you wrapped your app with both Context Providers
- Check that you're using the custom hooks (useTheme, useProjects) correctly

**"Network request failed"**
- Ensure json-server is running (`npm run server` or `npm run dev:all`)
- Check that you're using the correct port (3001)

**Theme not persisting after refresh:**
- Check localStorage implementation in ThemeContext
- Ensure you're reading from localStorage on initial state

**Race condition warnings:**
- Always implement cleanup functions in useEffect
- Use the cancelled flag pattern for async operations

### 📖 Resources & References
- [React useEffect Documentation](https://react.dev/reference/react/useEffect)
- [Context API Guide](https://react.dev/learn/passing-data-deeply-with-context)
- [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [JSON Server Documentation](https://github.com/typicode/json-server)

### ✅ Project Checklist
- [ ] Theme Context created and working
- [ ] Theme toggle in navbar functional
- [ ] Theme persists in localStorage
- [ ] Projects Context fetches from API
- [ ] Loading spinner shows during fetch
- [ ] Error handling displays user-friendly messages
- [ ] Projects display dynamically from API
- [ ] At least one CRUD operation implemented
- [ ] useEffect cleanup prevents race conditions
- [ ] No console errors or warnings
- [ ] Code is well-commented
- [ ] All functionality tested and working
