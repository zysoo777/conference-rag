# Part 2: Supabase Project Setup (10 min)

## Step 2a: Create a Supabase Project

1. Go to [https://supabase.com](https://supabase.com)
2. Sign up / Sign in
3. Click **"New Project"**
4. Fill in:
   - **Name**: `conference-rag` (or anything)
   - **Database Password**: Choose a strong password (save it!)
   - **Region**: Choose closest to you
5. Click **"Create new project"** (takes ~2 minutes)

## Step 2b: Get Your Credentials

Once the project is created:

1. Go to **Settings** (gear icon) → **API**
2. You'll need these values:
   - **Project URL**: `https://xyzabc123.supabase.co`
   - **anon public** key: Long string starting with `eyJ...`
   - **service_role** key: Long string starting with `eyJ...` (click "Reveal")

3. Extract your **Project Reference ID** from the URL:
   - Example: `https://xyzabc123.supabase.co` → Reference ID is `xyzabc123`

4. Get a **Personal Access Token**:
   - Go to [https://supabase.com/dashboard/account/tokens](https://supabase.com/dashboard/account/tokens)
   - Click "Generate new token"
   - Name: "Conference RAG Setup"
   - Copy the token (starts with `sbp_`)

5. Get an **OpenAI API Key**:
   - Go to [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)
   - Click "Create new secret key"
   - Copy the key (starts with `sk-`)

Now add these to **Colab Secrets** 🔑

## Step 2c: Load Credentials

```python
# @title 🔐 Load Your Credentials from Colab Secrets

# To add secrets in Colab:
# 1. Click the 🔑 key icon in the left sidebar
# 2. Add each secret below (click "+ Add new secret")
# 3. Toggle "Notebook access" ON for each

from google.colab import userdata
import os

# Required secrets:
# - SUPABASE_URL
# - SUPABASE_ANON_KEY
# - SUPABASE_SERVICE_KEY
# - SUPABASE_PROJECT_REF
# - SUPABASE_ACCESS_TOKEN
# - OPENAI_API_KEY

try:
    SUPABASE_URL = userdata.get('SUPABASE_URL')
    SUPABASE_ANON_KEY = userdata.get('SUPABASE_ANON_KEY')
    SUPABASE_SERVICE_KEY = userdata.get('SUPABASE_SERVICE_KEY')
    SUPABASE_PROJECT_REF = userdata.get('SUPABASE_PROJECT_REF')
    SUPABASE_ACCESS_TOKEN = userdata.get('SUPABASE_ACCESS_TOKEN')
    OPENAI_API_KEY = userdata.get('OPENAI_API_KEY')
    
    # Set environment variable for Supabase CLI
    os.environ['SUPABASE_ACCESS_TOKEN'] = SUPABASE_ACCESS_TOKEN
    
    print("✅ All credentials loaded!")
    print(f"   Project: {SUPABASE_URL}")
    print(f"   OpenAI Key: {OPENAI_API_KEY[:8]}...")
except Exception as e:
    print(f"❌ Error: {e}")
    print("\\nAdd credentials to Colab Secrets (🔑 icon)")
    raise
```
