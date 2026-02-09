# Part 7: Generate Embeddings & Import Data (25 min)

Now we'll convert the text to embeddings and import everything to Supabase.

## Step 7a: Split Talks into Sentences

```python
# @title ✂️ Split Talks into Sentences

import uuid
import re

def split_into_sentences(text):
    """Split text into sentences (simple approach)"""
    # Split on period followed by space and capital letter
    sentences = re.split(r'\\. (?=[A-Z])', text)
    # Clean up
    sentences = [s.strip() + '.' if not s.endswith('.') else s.strip() for s in sentences]
    return [s for s in sentences if len(s) > 20]  # Filter very short sentences

# Create sentence records
sentence_records = []

for _, talk in tqdm(talks_df.iterrows(), total=len(talks_df), desc="Splitting into sentences"):
    talk_id = str(uuid.uuid4())
    sentences = split_into_sentences(talk['text'])
    
    for i, sentence in enumerate(sentences, 1):
        sentence_records.append({
            'talk_id': talk_id,
            'title': talk['title'],
            'speaker': talk['speaker'],
            'calling': talk['calling'],
            'year': talk['year'],
            'season': talk['season'],
            'url': talk['url'],
            'sentence_num': i,
            'text': sentence
        })

sentences_df = pd.DataFrame(sentence_records)

print(f"\n✅ Split {len(talks_df)} talks into {len(sentences_df):,} sentences")
print(f"   Average sentences per talk: {len(sentences_df) / len(talks_df):.1f}")
print(f"   Average sentence length: {sentences_df['text'].str.len().mean():.0f} characters")
```

## Step 7b: Generate Embeddings

```python
# @title 🧠 Generate OpenAI Embeddings (this may take 10-15 minutes)

import openai
import time
from openai import OpenAI

# Initialize OpenAI client
client = OpenAI(api_key=OPENAI_API_KEY)

def get_embedding_batch(texts, model="text-embedding-3-small"):
    """Get embeddings for a batch of texts"""
    try:
        response = client.embeddings.create(
            model=model,
            input=texts
        )
        return [item.embedding for item in response.data]
    except Exception as e:
        print(f"Error: {e}")
        return None

# Process in batches to avoid rate limits
BATCH_SIZE = 100
embeddings = []
failed_indices = []

print(f"Generating embeddings for {len(sentences_df):,} sentences...")
print(f"Batch size: {BATCH_SIZE}\n")

for i in tqdm(range(0, len(sentences_df), BATCH_SIZE)):
    batch_texts = sentences_df['text'].iloc[i:i+BATCH_SIZE].tolist()
    
    batch_embeddings = get_embedding_batch(batch_texts)
    
    if batch_embeddings:
        embeddings.extend(batch_embeddings)
    else:
        failed_indices.extend(range(i, min(i+BATCH_SIZE, len(sentences_df))))
        # Add empty embeddings as placeholder
        embeddings.extend([None] * len(batch_texts))
    
    # Rate limiting: OpenAI allows ~3000 requests/min
    time.sleep(0.1)

# Add embeddings to dataframe
sentences_df['embedding'] = embeddings

# Remove failed embeddings
sentences_df = sentences_df[sentences_df['embedding'].notna()]

print(f"\n✅ Generated {len(sentences_df):,} embeddings")
if failed_indices:
    print(f"   ⚠️ {len(failed_indices)} failed (removed)")

# Estimate cost
total_tokens = sentences_df['text'].str.split().str.len().sum()
cost = (total_tokens / 1_000_000) * 0.020  # $0.020 per 1M tokens
print(f"\n💰 Estimated cost: ${cost:.2f}")
```

## Step 7c: Import to Supabase

```python
# @title 💾 Import Data to Supabase

# Convert to list of dicts for insertion
records = sentences_df.to_dict('records')

# Convert embeddings to lists (from numpy arrays if needed)
for record in records:
    if hasattr(record['embedding'], 'tolist'):
        record['embedding'] = record['embedding'].tolist()

print(f"Importing {len(records):,} sentence embeddings to Supabase...")
print("This may take 5-10 minutes...\n")

# Insert in batches
BATCH_SIZE = 100
success_count = 0
error_count = 0

for i in tqdm(range(0, len(records), BATCH_SIZE)):
    batch = records[i:i+BATCH_SIZE]
    
    try:
        result = supabase_admin.table('sentence_embeddings').insert(batch).execute()
        success_count += len(batch)
    except Exception as e:
        print(f"\nError inserting batch {i//BATCH_SIZE + 1}: {e}")
        error_count += len(batch)
        continue
    
    # Small delay to avoid overwhelming Supabase
    time.sleep(0.1)

print(f"\n✅ Import complete!")
print(f"   Success: {success_count:,} sentences")
if error_count > 0:
    print(f"   Errors: {error_count:,} sentences")
```

## Step 7d: Verify Import

```python
# ✅ CHECKPOINT 4: Verify Data Import

# Check row count
result = supabase_admin.table('sentence_embeddings').select('id', count='exact').limit(1).execute()
row_count = result.count or 0

print(f"✅ Database contains {row_count:,} sentence embeddings")

# Test vector search
if row_count > 0:
    # Get an embedding from our data
    test_embedding = embeddings[0]
    
    # Try the match_sentences function
    result = supabase_admin.rpc('match_sentences', {
        'query_embedding': test_embedding,
        'match_threshold': 0.7,
        'match_count': 5
    }).execute()
    
    if result.data:
        print(f"\n✅ Vector search working!")
        print(f"   Found {len(result.data)} similar sentences")
        print(f"\nTop match:")
        print(f"   Title: {result.data[0]['title']}")
        print(f"   Speaker: {result.data[0]['speaker']}")
        print(f"   Text: {result.data[0]['text'][:100]}...")
        print(f"   Similarity: {result.data[0]['similarity']:.3f}")
    else:
        print("⚠️ No results from vector search (this might be normal)")
else:
    print("❌ No data in database! Check import step above.")
```

### 💡 Learning Checkpoint

**What just happened?**

1. **Sentence splitting**: ~400 talks → ~80,000 sentences
2. **Embedding generation**: Each sentence → 1,536-dimensional vector
3. **Vector database**: Stored in pgvector for fast similarity search

**Why sentence-level?**
- Research shows sentences preserve semantic meaning
- Higher precision for specific queries
- Can aggregate by talk for context

This is the **core of RAG**: converting text to searchable vectors!
