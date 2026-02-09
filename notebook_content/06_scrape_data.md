# Part 6: Scrape Conference Data (20 min)

Now let's get the actual data! We'll scrape 5 years of conference talks from the Church's website.

## Step 6a: Install Dependencies

```python
# @title 📦 Install Scraping Libraries

!pip install -q beautifulsoup4 requests pandas tqdm

print("✅ Libraries installed!")
```

## Step 6b: Scrape Conference Talks

```python
# @title 🌐 Scrape Conference Talks (5 years)

import requests
from bs4 import BeautifulSoup
import pandas as pd
import re
from tqdm.auto import tqdm
from concurrent.futures import ThreadPoolExecutor, as_completed

# How many years to scrape
YEARS_TO_SCRAPE = 5
START_YEAR = 2025 - YEARS_TO_SCRAPE
END_YEAR = 2025

def setup_session():
    """Create session with retries"""
    session = requests.Session()
    session.headers.update({
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    })
    return session

def get_conference_urls(start_year, end_year):
    """Generate URLs for conferences"""
    base_url = 'https://www.churchofjesuschrist.org/study/general-conference/{year}/{month}?lang=eng'
    return [(base_url.format(year=year, month=month), str(year), month)
            for year in range(start_year, end_year + 1)
            for month in ['04', '10']]

def get_talk_urls(conference_url, year, month, session):
    """Fetch talk URLs from a conference page"""
    try:
        response = session.get(conference_url, timeout=10)
        response.raise_for_status()
    except:
        return []
    
    soup = BeautifulSoup(response.text, 'html.parser')
    talk_urls = []
    seen_urls = set()
    
    # Session slugs to exclude
    session_slugs = [
        'saturday-morning', 'saturday-afternoon', 'sunday-morning', 'sunday-afternoon',
        'priesthood-session', 'women-session', 'womens-session', 'session', 'video'
    ]
    
    for link in soup.select('div.talk-list a[href*="/study/general-conference/"]'):
        href = link.get('href')
        if not href or 'lang=eng' not in href:
            continue
        
        canonical_url = 'https://www.churchofjesuschrist.org' + href
        if canonical_url in seen_urls:
            continue
        seen_urls.add(canonical_url)
        
        # Skip session videos
        if any(slug in canonical_url.lower() for slug in session_slugs):
            continue
        
        talk_urls.append(canonical_url)
    
    return talk_urls

def scrape_talk(talk_url, session):
    """Scrape a single talk"""
    try:
        response = session.get(talk_url, timeout=10)
        response.raise_for_status()
    except:
        return None
    
    soup = BeautifulSoup(response.text, 'html.parser')
    
    def clean_text(text):
        if not text:
            return text
        return text.strip()
    
    title = clean_text(soup.find("h1").text) if soup.find("h1") else "No Title"
    speaker_tag = soup.find("p", {"class": "author-name"})
    speaker = clean_text(speaker_tag.text) if speaker_tag else "Unknown"
    
    calling_tag = soup.find("p", {"class": "author-role"})
    calling = clean_text(calling_tag.text) if calling_tag else ""
    
    content_div = soup.find("div", {"class": "body-block"})
    if not content_div:
        return None
    
    content = " ".join(clean_text(p.text) for p in content_div.find_all("p"))
    
    year_match = re.search(r'/(\d{4})/', talk_url)
    year = int(year_match.group(1)) if year_match else None
    season = "April" if "/04/" in talk_url else "October"
    
    return {
        "title": title,
        "speaker": speaker,
        "calling": calling,
        "year": year,
        "season": season,
        "url": talk_url,
        "text": content
    }

# Main scraping logic
print(f"📰 Scraping {YEARS_TO_SCRAPE} years of conference talks ({START_YEAR}-{END_YEAR})...\n")

session = setup_session()
conference_urls = get_conference_urls(START_YEAR, END_YEAR)

# Get all talk URLs
print("Finding talk URLs...")
all_talk_urls = []
for conf_url, year, month in tqdm(conference_urls):
    talk_urls = get_talk_urls(conf_url, year, month, session)
    all_talk_urls.extend(talk_urls)

print(f"Found {len(all_talk_urls)} talks\n")

# Scrape talks in parallel
print("Scraping talk content...")
talks_data = []
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(scrape_talk, url, session): url for url in all_talk_urls}
    for future in tqdm(as_completed(futures), total=len(all_talk_urls)):
        talk = future.result()
        if talk:
            talks_data.append(talk)

talks_df = pd.DataFrame(talks_data)

print(f"\n✅ Scraped {len(talks_df)} talks successfully!")
print(f"   Years: {talks_df['year'].min()} - {talks_df['year'].max()}")
print(f"   Total words: {talks_df['text'].str.split().str.len().sum():,}")

# Preview
print("\nSample talks:")
print(talks_df[['year', 'season', 'title', 'speaker']].head(10))
```

### 💡 Learning Checkpoint

The scraper:
1. Finds all conference URLs for the year range
2. Extracts talk URLs (excluding session videos)
3. Scrapes each talk in parallel (10 at a time)
4. Cleans and structures the data

This is real **web scraping** - a valuable data engineering skill!
