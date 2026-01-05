---
allowed-tools: Read, Write, Bash(git *)
description: Sync latest tasks.json to GitHub task board repository and push changes
---

# Sync Tasks to GitHub

1. Check if we're in the task-board repository
   - If not, ask the user for the path to their task-board repo
   
2. Look for tasks.json in the current directory or Downloads folder
   - Check ./tasks.json
   - Check ~/Downloads/tasks.json
   - If found in Downloads, copy it to the repo
   
3. If tasks.json exists and has been updated:
   - Show the user a summary of changes (number of tasks, any new/updated items)
   - Stage the file: git add tasks.json
   - Create a descriptive commit message based on changes (e.g., "Update tasks: added 2 new tasks, marked 1 complete")
   - Commit: git commit -m "[generated message]"
   - Push to GitHub: git push origin main
   - Confirm success and remind user GitHub Pages will refresh in ~1 minute
   
4. If no tasks.json found:
   - Ask user where to find it
   - Provide guidance on downloading from Claude chat

Remember: This command should be run from inside the task-board repository directory, or provide the path when prompted.
