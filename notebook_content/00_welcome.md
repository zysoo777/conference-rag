# 🎓 Conference RAG - Complete Setup Guide

Welcome! In this notebook, you'll build a **production-ready Retrieval Augmented Generation (RAG) application** that lets users ask questions about conference talks using semantic search and AI-generated answers.

## What You'll Build

A full-stack web application with:
- ✅ User authentication (Supabase magic links)
- ✅ Vector embeddings & semantic search (pgvector)
- ✅ Server-side API key management (Edge Functions)
- ✅ Row Level Security (RLS)
- ✅ Deployed on GitHub Pages

## Architecture

```
┌─────────────┐
│   Browser   │  Student asks question
│  (GitHub    │
│   Pages)    │
└──────┬──────┘
       │
       ├─── Supabase Auth (magic link)
       │
       ├─── Edge Function: embed-question
       │         ↓ OpenAI API (server-side key 🔒)
       │         ↓ Returns embedding vector
       │
       ├─── Supabase Database (pgvector)
       │         ↓ Vector similarity search
       │         ↓ Returns top matching sentences
       │
       └─── Edge Function: generate-answer
                 ↓ OpenAI GPT-4 (server-side key 🔒)
                 ↓ Returns final answer
```

## Learning Objectives

You'll learn:
1. **Vector Embeddings** - How to represent text as numbers
2. **Semantic Search** - Finding similar content without exact keyword matches
3. **RAG Architecture** - Combining retrieval + generation
4. **Server-side Security** - Protecting API keys with Edge Functions
5. **Row Level Security** - User-specific data isolation
6. **Production Deployment** - Real-world application architecture

## Time Estimate
⏱️ **~85 minutes** (grab a coffee!)

## Cost Estimate
💰 **~$0.60** in OpenAI API usage (for 5 years of conference talks)

Let's get started! 🚀
