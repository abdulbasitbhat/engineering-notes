# Pushing Project to GitHub

## First Time Setup

```powershell
# 1. Go to project root
cd D:\Projects\Dynamic-middleware-connector

# 2. Initialize git
git init

# 3. Generate .gitignore for .NET (excludes bin/, obj/, and other generated files)
dotnet new gitignore

# 4. Stage all files
git add .

# 5. First commit
git commit -m "Initial commit — Parsers library and ConnectorApp setup"

# 6. Point to your GitHub repo (create the repo on github.com first, empty, no README)
git remote add origin https://github.com/abdulbasitbhat/Dynamic-middleware-connector.git

# 7. Rename branch to main and push
git branch -M main
git push -u origin main
```

---

## Every Commit After This

```powershell
git add .
git commit -m "your message here"
git push
```
