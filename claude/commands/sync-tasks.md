---
allowed-tools: Read, Write, Bash(git *)
description: Sync latest tasks.json to GitHub (task subfolder in smgam29.github.io repo)
---

# Sync Tasks to GitHub

1. Check if we're in the smgam29.github.io repository
   - If not, ask the user for the path to their repo
   
2. Navigate to the task subfolder (or verify we're already there)
   - cd task (if in repo root)
   
3. Look for tasks.json:
   - Check ./task/tasks.json (if in repo root)
   - Check ./tasks.json (if in task subfolder)
   - Check ~/Downloads/tasks.json
   - If found in Downloads, copy it to the task subfolder
   
4. If tasks.json exists and has been updated:
   - Show the user a summary of changes (number of tasks, any new/updated items)
   - Stage the file: git add task/tasks.json
   - Create a descriptive commit message based on changes (e.g., "Update tasks: added 2 new tasks, marked 1 complete")
   - Commit: git commit -m "[generated message]"
   - Push to GitHub: git push origin main
   - Confirm success and remind user GitHub Pages will refresh in ~1 minute
   
5. If no tasks.json found:
   - Ask user where to find it
   - Provide guidance on downloading from Claude chat

Remember: This command can be run from either the repo root or the task subfolder.

