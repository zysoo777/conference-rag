# Conference RAG - Setup Instructions

## Quick Start

The easiest way to set up this project is using our **Google Colab notebook** which walks you through every step.

### Option 1: Google Colab (Recommended) ⭐

1. **Open the setup notebook** in Google Colab:
   - Upload `setup.ipynb` to Google Colab
   - Or click: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR-ORG/conference-rag/blob/main/setup.ipynb)

2. **Follow the notebook** - It will guide you through:
   - Creating a Supabase project
   - Setting up the database
   - Deploying to GitHub Pages
   - Deploying Edge Functions
   - Scraping and importing conference talks
   - Testing your RAG system

3. **Time**: ~85 minutes  
   **Cost**: ~$0.60 in OpenAI API usage

### Option 2: Manual Setup

If you prefer to set things up manually, follow the documentation:

1. [SETUP.md](SETUP.md) - Complete setup instructions
2. [README.md](README.md) - Project overview and architecture

## What You'll Build

A production-ready RAG (Retrieval Augmented Generation) application that:

- ✅ Lets users ask questions about General Conference talks
- ✅ Uses semantic search to find relevant content
- ✅ Generates AI-powered answers using GPT-4
- ✅ Protects API keys with server-side Edge Functions
- ✅ Implements Row Level Security
- ✅ Deploys to GitHub Pages (free hosting!)

## Prerequisites

- Google account (for Colab)
- GitHub account (for repository and deployment)
- Supabase account (free tier: https://supabase.com)
- OpenAI API key (~$0.60 usage: https://platform.openai.com/api-keys)

## Learning Objectives

This project teaches you:

1. **Vector Embeddings** - Converting text to searchable numerical representations
2. **Semantic Search** - Finding similar content using vector similarity
3. **RAG Architecture** - Combining retrieval and generation for accurate AI responses
4. **Edge Functions** - Serverless compute for secure API key management
5. **Row Level Security** - Database-level access control
6. **Production Deployment** - Full-stack application deployment

## Getting Help

- **Issues with the notebook?** Check the troubleshooting section in `setup.ipynb`
- **General questions?** See [README.md](README.md)
- **Found a bug?** Open an issue on GitHub

## Next Steps

After completing setup:

1. Test your application
2. Try the optional extensions (in Part 9 of the notebook)
3. Share your project!

---

🎓 **This project is part of a course on web development and AI applications.**

💡 **Questions or feedback?** Open an issue or discussion on GitHub!
