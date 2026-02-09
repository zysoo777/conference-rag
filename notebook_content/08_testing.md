# Part 8: Test Your RAG System! (10 min)

🎉 **Everything is set up!** Let's test the complete system.

## Step 8a: Test from Frontend

1. Go to your deployed site: `https://YOUR-USERNAME.github.io/my-conference-rag/`
2. Make sure you're logged in
3. Ask a question: **"How can I find peace during difficult times?"**
4. Watch the magic happen!

**What's happening behind the scenes:**
```
Your Question
    ↓
Edge Function: embed-question
    ↓ (OpenAI embedding)
Vector Search in pgvector
    ↓ (top 20 sentences)
Group by talk_id, rank
    ↓ (top 3 talks)
Edge Function: generate-answer
    ↓ (GPT-4 with context)
Final Answer! ✨
```

## Step 8b: Test from Colab

```python
# @title 🧪 Test RAG Pipeline End-to-End

def test_rag_system(question):
    """Test the complete RAG pipeline"""
    print(f"Question: {question}\n")
    
    # Step 1: Get embedding for question
    print("1️⃣ Getting embedding for question...")
    embed_response = requests.post(
        f"{SUPABASE_URL}/functions/v1/embed-question",
        headers={
            "Authorization": f"Bearer {SUPABASE_ANON_KEY}",
            "Content-Type": "application/json"
        },
        json={"question": question}
    )
    embedding = embed_response.json()['embedding']
    print(f"   ✅ Got {len(embedding)}-dimensional embedding\n")
    
    # Step 2: Search for similar sentences
    print("2️⃣ Searching for similar sentences...")
    search_result = supabase_admin.rpc('match_sentences', {
        'query_embedding': embedding,
        'match_threshold': 0.6,
        'match_count': 20
    }).execute()
    
    sentences = search_result.data
    print(f"   ✅ Found {len(sentences)} similar sentences\n")
    
    # Step 3: Group by talk and rank
    print("3️⃣ Ranking talks by relevance...")
    from collections import defaultdict
    talk_sentences = defaultdict(list)
    
    for sent in sentences:
        talk_sentences[sent['talk_id']].append(sent)
    
    # Sort talks by number of matching sentences
    ranked_talks = sorted(
        talk_sentences.items(),
        key=lambda x: len(x[1]),
        reverse=True
    )[:3]  # Top 3 talks
    
    print(f"   ✅ Top 3 relevant talks:\n")
    context_talks = []
    for i, (talk_id, sents) in enumerate(ranked_talks, 1):
        # Get full talk text
        full_talk_result = supabase_admin.table('sentence_embeddings') \
            .select('title, speaker, text') \
            .eq('talk_id', talk_id) \
            .execute()
        
        talk_sentences_texts = [s['text'] for s in full_talk_result.data]
        full_text = ' '.join(talk_sentences_texts)
        
        context_talks.append({
            'title': sents[0]['title'],
            'speaker': sents[0]['speaker'],
            'text': full_text
        })
        
        print(f"      {i}. \"{sents[0]['title']}\" by {sents[0]['speaker']}")
        print(f"         ({len(sents)} matching sentences)\n")
    
    # Step 4: Generate answer
    print("4️⃣ Generating answer with GPT-4...")
    answer_response = requests.post(
        f"{SUPABASE_URL}/functions/v1/generate-answer",
        headers={
            "Authorization": f"Bearer {SUPABASE_ANON_KEY}",
            "Content-Type": "application/json"
        },
        json={
            "question": question,
            "context_talks": context_talks
        }
    )
    answer = answer_response.json()['answer']
    
    print(f"   ✅ Generated answer!\n")
    print("="*60)
    print("ANSWER:")
    print("="*60)
    print(answer)
    print("="*60)
    
    return answer

# Test questions
test_questions = [
    "How can I strengthen my faith?",
    "What does the church teach about prayer?",
    "How can I find peace during trials?"
]

print("Testing RAG system with sample questions...\n")
print("="*60)

for q in test_questions:
    test_rag_system(q)
    print("\n" + "="*60 + "\n")
```

## ✅ CHECKPOINT 5: Final Verification

```python
# Final system check

print("🎉 FINAL SYSTEM CHECK\n")
print("="*60)

checks = {
    "Database has data": False,
    "Vector search works": False,
    "Embed function works": False,
    "Answer function works": False
}

# Check 1: Database
try:
    result = supabase_admin.table('sentence_embeddings').select('id', count='exact').limit(1).execute()
    if result.count > 0:
        checks["Database has data"] = True
except:
    pass

# Check 2: Vector search
try:
    result = supabase_admin.rpc('match_sentences', {
        'query_embedding': embeddings[0],
        'match_count': 5
    }).execute()
    if result.data:
        checks["Vector search works"] = True
except:
    pass

# Check 3: Embed function
try:
    response = requests.post(
        f"{SUPABASE_URL}/functions/v1/embed-question",
        headers={"Authorization": f"Bearer {SUPABASE_ANON_KEY}", "Content-Type": "application/json"},
        json={"question": "test"}
    )
    if response.ok:
        checks["Embed function works"] = True
except:
    pass

# Check 4: Answer function
try:
    response = requests.post(
        f"{SUPABASE_URL}/functions/v1/generate-answer",
        headers={"Authorization": f"Bearer {SUPABASE_ANON_KEY}", "Content-Type": "application/json"},
        json={"question": "test", "context_talks": [{"title": "Test", "speaker": "Test", "text": "Test"}]}
    )
    if response.ok:
        checks["Answer function works"] = True
except:
    pass

# Print results
for check, passed in checks.items():
    status = "✅" if passed else "❌"
    print(f"{status} {check}")

all_passed = all(checks.values())
print("\n" + "="*60)
if all_passed:
    print("🎉 ALL SYSTEMS GO! Your RAG application is ready!")
    print("\nNext: Visit your deployed site and try asking questions!")
else:
    print("⚠️ Some checks failed. Review the steps above.")
print("="*60)
```
