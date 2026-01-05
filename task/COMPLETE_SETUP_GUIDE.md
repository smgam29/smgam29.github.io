# Complete Setup Guide: Claude Chat + Claude Code + GitHub Pages Task Board

**Total setup time: ~20 minutes**

This guide will walk you through creating a fully automated task management system where:
- You manage tasks naturally in Claude chat
- Claude Code automatically syncs changes to GitHub
- Your task board is always accessible at a permanent URL

---

## Part 1: Install Claude Code (5 minutes)

### Step 1: Check if you have Node.js

Open your terminal and run:
```bash
node --version
```

**If you see a version number (v18.x or higher):** ✅ You're good! Skip to Step 2.

**If you see "command not found":** You need to install Node.js:
- Go to https://nodejs.org
- Download the "LTS" version (left button)
- Run the installer
- Restart your terminal
- Run `node --version` again to confirm

### Step 2: Install Claude Code

In your terminal, run:
```bash
npm install -g @anthropic-ai/claude-code
```

Wait for installation to complete (1-2 minutes).

### Step 3: Authenticate Claude Code

Run:
```bash
claude
```

This will:
- Open your browser for authentication
- Ask you to sign in with your Claude account
- Return you to the terminal when done

**Test it works:** Type "hello" and press Enter. Claude should respond!

Type `/exit` to quit for now.

✅ **Claude Code is installed!**

---

## Part 2: Set Up GitHub Authentication (10 minutes)

### Step 1: Generate SSH Key (if you don't have one)

**Check if you already have a key:**
```bash
ls ~/.ssh/id_ed25519.pub
```

**If you see "No such file":** Create a new key:
```bash
ssh-keygen -t ed25519 -C "your.email@company.com"
```

Press Enter three times (accept defaults, no passphrase for simplicity).

### Step 2: Copy Your Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

This shows your public key. **Copy the entire output** (starts with `ssh-ed25519`).

### Step 3: Add Key to GitHub

1. Go to https://github.com/settings/keys
2. Click "New SSH key"
3. Title: "My Computer" (or any name)
4. Paste your public key
5. Click "Add SSH key"

### Step 4: Test the Connection

```bash
ssh -T git@github.com
```

You should see: "Hi [your-username]! You've successfully authenticated"

✅ **GitHub authentication complete!**

---

## Part 3: Create Your GitHub Pages Task Board (5 minutes)

### Step 1: Create GitHub Repository

1. Go to https://github.com/new
2. Repository name: `task-board`
3. Select "**Public**" (required for free GitHub Pages)
4. ✅ Check "Add a README file"
5. Click "Create repository"

### Step 2: Upload Your Task Board Files

I've created these files for you:
- `index.html` (the task board interface)
- `tasks.json` (your 9 current tasks)
- `.claude/commands/sync-tasks.md` (automation script)
- `.claude/settings.json` (safety permissions)

**Upload them to GitHub:**

1. Download the `github-task-board` folder I created
2. In your GitHub repository, click "Add file" → "Upload files"
3. Drag these files from the folder:
   - `index.html`
   - `tasks.json`
4. Scroll down and click "Commit changes"

### Step 3: Enable GitHub Pages

1. In your repository, click "Settings" tab
2. Scroll down to "Pages" in the left sidebar
3. Under "Source": Select "Deploy from a branch"
4. Under "Branch": Select "main" and "/ (root)"
5. Click "Save"

**Wait 1-2 minutes**, then refresh the page. You'll see:
> "Your site is live at https://YOUR-USERNAME.github.io/task-board/"

✅ **Bookmark this URL!** This is your permanent task board.

### Step 4: Clone Repository to Your Computer

In your terminal, navigate to where you want to keep the repo:
```bash
cd ~/Documents  # or wherever you prefer
git clone git@github.com:YOUR-USERNAME/task-board.git
cd task-board
```

Replace `YOUR-USERNAME` with your actual GitHub username.

### Step 5: Add Claude Code Configuration

Copy the `.claude` folder from the files I created:

**Option A: If you downloaded the folder:**
```bash
cp -r ~/Downloads/github-task-board/.claude ~/Documents/task-board/
```

**Option B: Create manually:**
```bash
mkdir -p .claude/commands
```

Then create these two files:

**File 1: `.claude/commands/sync-tasks.md`**
(Copy content from the sync-tasks.md file I created)

**File 2: `.claude/settings.json`**
(Copy content from the settings.json file I created)

Commit these configuration files:
```bash
git add .claude
git commit -m "Add Claude Code configuration"
git push
```

✅ **Setup complete!**

---

## Part 4: The Daily Workflow (30 seconds per update)

### Updating Tasks Through Claude Chat (This Chat!)

**You:** "Add a task to review the marketing plan by Friday"

**Me:** Captures task, adds metadata, identifies dependencies, proactively researches/drafts materials

**Me:** "✅ Task added! Here's your updated tasks.json file"

**You:** Download the tasks.json file I provide

### Syncing to GitHub with Claude Code

**Option A: Interactive (Safe, good for learning)**

1. Open terminal in your task-board folder
2. Copy the downloaded tasks.json to the folder
3. Run: `claude`
4. Say: "The tasks.json file has been updated. Please commit and push it to GitHub."
5. Review the changes Claude shows you
6. Confirm when prompted
7. GitHub Pages refreshes automatically!

**Option B: Use the Slash Command (Fastest)**

1. Open terminal in your task-board folder
2. Copy tasks.json to the folder
3. Run: `claude`
4. Type: `/sync-tasks`
5. Done! Claude handles everything automatically.

**Option C: One-Line Command (Expert mode)**

```bash
cd ~/Documents/task-board
cp ~/Downloads/tasks.json .
claude -p "Commit and push tasks.json with a descriptive message" --allowedTools "Bash(git *)"
```

All done in one command!

---

## Alternative: Simpler "Copy-Paste" Approach

If Claude Code setup feels like too much right now, here's an even simpler workflow:

1. **You:** Update tasks in this chat
2. **Me:** Generate updated tasks.json
3. **You:** Go to GitHub → click tasks.json → click Edit (✏️) → Paste new content → Commit
4. **GitHub:** Auto-refreshes your board in ~30 seconds

This works great and requires zero local setup!

---

## Troubleshooting

**"Permission denied (publickey)" when pushing:**
- Your SSH key isn't set up correctly
- Re-run: `ssh -T git@github.com` to test
- Make sure you added the key to GitHub (Part 2, Step 3)

**Claude Code can't find git:**
- Make sure git is installed: `git --version`
- If not: Download from https://git-scm.com

**GitHub Pages not updating:**
- Check Settings → Pages shows "Your site is live"
- Hard refresh browser: Cmd+Shift+R (Mac) or Ctrl+Shift+R (Windows)
- Wait 2-3 minutes for first deployment

**tasks.json changes don't show on board:**
- Hard refresh: Ctrl+Shift+R
- Check browser console (F12) for errors
- Verify JSON is valid at https://jsonlint.com

---

## What Would You Like To Do?

a) Do the full setup now (Claude Code + GitHub Pages) - I'll walk you through each step  
b) Just use GitHub Pages for now (manual updates) - simpler, still works great  
c) Keep everything in this chat only (no external hosting) - I'll track conversationally  
d) Think about it and decide later

Which approach feels right for you?
