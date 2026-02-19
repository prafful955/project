# 🌿 Git Commands – Backend Developer Cheat Sheet

---

## 🔍 1. Check Status

git status

Shows:
- Modified files
- Untracked files
- Current branch

---

## ➕ 2. Add Files

### Add single file
git add filename.java

### Add all files
git add .

---

## 💾 3. Commit

git commit -m "message"

Example:
git commit -m "Added Kafka producer logic"

---

## 🚀 4. Push Code

git push origin main

Push to specific branch:
git push origin feature/kafka-integration

---

## 📥 5. Pull Latest Code

git pull origin main

---

## 🌱 6. Branch Commands

### Create new branch
git checkout -b feature/kafka

### Switch branch
git checkout main

### List branches
git branch

---

## 🔄 7. Merge Branch

Switch to main first:
git checkout main

Then merge:
git merge feature/kafka

---

## ❌ 8. Undo Changes

### Undo file changes
git checkout -- filename.java

### Undo last commit (keep changes)
git reset --soft HEAD~1

### Undo last commit (delete changes)
git reset --hard HEAD~1

---

## 🧠 9. View History

git log

One line format:
git log --oneline

---

## 🔥 10. Delete Branch

### Delete local branch
git branch -d feature/kafka

### Delete remote branch
git push origin --delete feature/kafka

---

# 💡 Important Concepts

Working Directory → Your files
Staging Area → git add
Repository → git commit
Origin → Remote repository (GitHub)

Basic Flow:
git add → git commit → git push

