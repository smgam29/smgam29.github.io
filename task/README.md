# Task Board - GitHub Pages Setup

Your personal AI-assisted task management system, hosted for free on GitHub Pages.

## 🚀 Quick Setup (5 minutes)

### Step 1: Create GitHub Repository

1. Go to https://github.com/new
2. Repository name: `task-board` (or any name you prefer)
3. Choose "Public" (required for free GitHub Pages)
4. Check "Add a README file"
5. Click "Create repository"

### Step 2: Upload Files

1. In your new repository, click "Add file" → "Upload files"
2. Drag and drop these 2 files:
   - `index.html`
   - `tasks.json`
3. Scroll down and click "Commit changes"

### Step 3: Enable GitHub Pages

1. In your repository, click "Settings" tab
2. In the left sidebar, click "Pages"
3. Under "Source", select "Deploy from a branch"
4. Under "Branch", select "main" (or "master") and "/ (root)"
5. Click "Save"
6. Wait 1-2 minutes for deployment

### Step 4: Access Your Task Board

Your task board will be available at:
```
https://YOUR-USERNAME.github.io/task-board/
```

(Replace YOUR-USERNAME with your actual GitHub username)

Bookmark this URL for instant access!

---

## 📝 Updating Tasks

### Method 1: Edit on GitHub (Web Interface)

1. Go to your repository
2. Click on `tasks.json`
3. Click the pencil icon (✏️) to edit
4. Make your changes to the JSON
5. Scroll down and click "Commit changes"
6. Refresh your task board page - changes appear immediately!

### Method 2: Edit Locally (Advanced)

1. Clone the repository: `git clone https://github.com/YOUR-USERNAME/task-board.git`
2. Edit `tasks.json` in your favorite editor
3. Commit and push:
   ```bash
   git add tasks.json
   git commit -m "Update tasks"
   git push
   ```
4. Refresh your task board page

---

## 📋 Task JSON Format

Each task has this structure:

```json
{
  "id": "unique-id",
  "title": "Task title",
  "description": "Task description",
  "priority": "critical|high|medium|low",
  "dueDate": "2026-01-15",
  "tags": ["tag1", "tag2"],
  "status": "todo|in-progress|completed",
  "notes": "Additional notes",
  "createdAt": "2026-01-05T00:00:00.000Z",
  "completedAt": null
}
```

### Adding a New Task

1. Open `tasks.json`
2. Copy an existing task object
3. Paste it inside the `"tasks": [...]` array (remember to add a comma!)
4. Update the values (especially `id` - make it unique!)
5. Save and commit

### Updating Task Status

To mark a task as complete:
1. Find the task in `tasks.json`
2. Change `"status": "todo"` to `"status": "completed"`
3. Add completion date: `"completedAt": "2026-01-06T00:00:00.000Z"`
4. Save and commit

---

## 🎨 Features

- **Kanban Board**: Three columns (To Do, In Progress, Completed)
- **Stats Dashboard**: Total, In Progress, Completed, Overdue counts
- **Search**: Filter tasks by title, description, or tags
- **Filters**: Filter by status or priority
- **Responsive**: Works on desktop, tablet, and mobile
- **Fast**: Static files = instant loading
- **Free**: GitHub Pages is free forever

---

## 💡 Tips

1. **Bookmark the URL** for quick access from any device
2. **Keep task IDs unique** - use timestamps or sequential numbers
3. **Use tags liberally** - makes filtering easier
4. **Set due dates** - enables overdue tracking
5. **Keep notes concise** - displays better on cards

---

## 🔄 Syncing with Claude

When you update tasks in conversation with Claude:
1. Claude will tell you the task details
2. You can manually update `tasks.json` on GitHub
3. Or ask Claude to generate an updated `tasks.json` file you can upload

---

## 🆘 Troubleshooting

**Page shows 404:**
- Wait 2-3 minutes after enabling GitHub Pages
- Check Settings → Pages shows "Your site is live"
- Verify files are in the main branch root directory

**Tasks don't load:**
- Check browser console for errors (F12 → Console tab)
- Verify `tasks.json` is valid JSON (use JSONLint.com)
- Make sure both files are in the same directory

**Changes don't appear:**
- Hard refresh the page (Ctrl+Shift+R or Cmd+Shift+R)
- Clear browser cache
- Wait 30-60 seconds for GitHub Pages to rebuild

---

Enjoy your always-accessible task board! 🎉
