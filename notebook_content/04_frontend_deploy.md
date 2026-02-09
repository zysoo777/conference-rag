# Part 4: Frontend Deployment (15 min)

Now let's get your frontend app online! This is where students will actually use the RAG system.

## Step 4a: Update config.js

In your GitHub repository, edit the `config.js` file:

1. Go to your repository on GitHub
2. Click on `config.js`
3. Click the pencil icon (✏️) to edit
4. Replace the placeholder values:

```javascript
const SUPABASE_CONFIG = {
    url: 'YOUR_SUPABASE_URL',      // Replace with your actual URL
    anonKey: 'YOUR_ANON_KEY'       // Replace with your actual anon key
};
```

5. Click "Commit changes"

## Step 4b: Deploy to GitHub Pages

1. Go to your repository **Settings**
2. Click **Pages** in the left sidebar
3. Under "Source":
   - Select **Deploy from a branch**
   - Branch: **main** (or **master**)
   - Folder: **/ (root)**
4. Click **Save**
5. Wait ~2 minutes for deployment

Your site will be at: `https://YOUR-USERNAME.github.io/my-conference-rag/`

## Step 4c: Configure Auth Redirect

Copy your deployed URL and add it to Supabase:

1. Go to Supabase Dashboard → **Authentication** → **URL Configuration**
2. Under "Redirect URLs", click **Add URL**
3. Paste: `https://YOUR-USERNAME.github.io/my-conference-rag/`
4. Click **Save**

## Step 4d: Test Login

1. Visit your deployed site
2. Enter your email
3. Click "Sign In with Magic Link"
4. Check your inbox
5. Click the magic link
6. You should be logged in! ✅

**Expected behavior**: You can log in, but asking questions will fail (we haven't deployed Edge Functions yet).

## ✅ Checkpoint 2

```python
# Verify your deployment

print("🌐 Check list:")
print("")
print("1. ✅ config.js updated with your credentials?")
print("2. ✅ Site deployed to GitHub Pages?")
print("3. ✅ Redirect URL added to Supabase?")
print("4. ✅ Successfully logged in?")
print("")
print("If yes to all, continue! If not, review the steps above.")
print("")
print("Your deployed URL should be:")
print(f"https://YOUR-USERNAME.github.io/REPO-NAME/")
```

### 💡 Learning Checkpoint

**Why can't we ask questions yet?**

The frontend is trying to call Edge Functions that don't exist yet:
1. `embed-question` - converts question to vector
2. `generate-answer` - calls GPT-4 for final answer

We'll deploy those next!
