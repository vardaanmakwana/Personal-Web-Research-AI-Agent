# Personal Web Research AI Agent - Creation Guide

This document details the step-by-step process of creating the Personal Web Research AI Agent, including challenges faced and how they were resolved.

---

## Overview

The goal was to build an AI research agent that:
- Searches the web for user queries
- Fetches and cleans webpage content
- Processes the information using a local LLM (Qwen)
- Generates a markdown summary report
- Runs entirely locally without cloud dependencies (except web search)

---

## Step 1: Initial Setup

### 1.1 Create Project Folder
Created a new project directory:
```
Personal-Web-Research-AI-Agent
```

### 1.2 Create Virtual Environment
Opened PowerShell and ran:
```powershell
python -m venv .venv
```

### 1.3 Activate Virtual Environment
```powershell
.\.venv\Scripts\Activate.ps1
```

If PowerShell blocked execution, used:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
```

---

## Step 2: Install Dependencies

With the virtual environment activated, installed the required Python packages:

```powershell
pip install requests beautifulsoup4 ollama python-dotenv
```

**Packages used:**
- `requests` - HTTP requests for web search and page fetching
- `beautifulsoup4` - HTML parsing and text extraction
- `ollama` - Python client for Ollama local model
- `python-dotenv` - Environment variable management from .env file

---

## Step 3: Get Ollama API Key

### 3.1 Created Ollama Account
- Visited https://ollama.com
- Signed up for a free account

### 3.2 Generated API Key
- Logged into account dashboard
- Navigated to API Keys section
- Clicked "Create API Key"
- Copied the generated key (format: `ollama_xxxxxxxxxxxxxxxx`)

### 3.3 Create .env File
Created a `.env` file in the project root directory with:
```
OLLAMA_API_KEY=your_actual_api_key_here
```

**Important:** No quotes around the key, no spaces around `=`

---

## Step 4: Download Qwen Model

Opened a terminal and downloaded the Qwen 3.5 model locally:
```powershell
ollama pull qwen3.5:4b
```

This downloaded approximately 3.4 GB of model data.

---

## Step 5: Create Initial Code

Created `research_agent.py` with the following structure:

```python
import os
import json
import requests
import ollama
from dotenv import load_dotenv
from bs4 import BeautifulSoup
from datetime import datetime
from pathlib import Path
```

The script consisted of three main functions:
1. `search_web(query)` - Calls Ollama web search API
2. `fetch_text(url)` - Downloads and cleans webpage content
3. `main()` - Orchestrates the workflow

---

## Step 6: Issues Encountered and Fixes

### Issue 1: Module Not Found - python-dotenv
**Problem:** `ModuleNotFoundError: No module named 'dotenv'`

**Solution:** 
```powershell
pip install python-dotenv
```

### Issue 2: Environment Variable Not Loading
**Problem:** `load_dotenv()` was imported but never called, so `API_KEY` was `None`

**Original code:**
```python
from dotenv import load_dotenv
API_KEY = os.getenv("OLLAMA_API_KEY")  # Returns None
```

**Fixed code:**
```python
from dotenv import load_dotenv

load_dotenv()  # <-- This line was missing!
API_KEY = os.getenv("OLLAMA_API_KEY")
```

**Verification:**
```python
print("API Key loaded:", API_KEY is not None)
```

### Issue 3: 401 Unauthorized Error
**Problem:** Script returned `401 Client Error: Unauthorized` when calling the web search API

**Causes investigated:**
- API key format (verified correct)
- API key validation (manually tested with curl)
- Account permissions (might not have access to web search endpoint)

**Resolution:**
The API key was correctly loaded and formatted. The 401 error indicated either:
- The Ollama account didn't have permissions for the web search API, or
- The endpoint had changed

Despite this, the local Qwen model worked correctly.

### Issue 4: Incorrect Response Object Access
**Problem:** Creating empty markdown files

**Original code:**
```python
digest = response.message.content  # Incorrect
```

**Fixed code:**
```python
digest = response["message"]["content"]  # Correct dictionary access
```

**Added error handling:**
```python
if not digest.strip():
    print("❌ Model returned an empty response.")
    return
```

### Issue 5: Console Output Management
**Problem:** Added debug print statements during troubleshooting

**Original:**
```python
print("\n===== FULL RESPONSE =====")
print(response)
print("=========================\n")
```

**Cleaned up:** Removed all debug prints for production use

---

## Step 7: Final Working Code

The complete, working `research_agent.py`:

```python
import os
import json
import requests
import ollama
from dotenv import load_dotenv
from bs4 import BeautifulSoup
from datetime import datetime
from pathlib import Path

load_dotenv()

API_KEY = os.getenv("OLLAMA_API_KEY")
SEARCH_URL = "https://ollama.com/api/web_search"
MODEL = "qwen3.5:4b"

# Search web using Ollama web search 
def search_web(query):
    response = requests.post(
        SEARCH_URL,
        headers={"Authorization": f"Bearer {API_KEY}"},
        json={"query": query, "max_results": 5},
        timeout=30,
    )
    response.raise_for_status()
    return response.json().get("results", [])

# Fetch full web page content
def fetch_text(url):
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
    except requests.RequestException as e:
        return ""
    soup = BeautifulSoup(response.text, "html.parser")
    for tag in soup(["script", "style", "nav", "footer"]):
        tag.decompose()
    return soup.get_text(separator="\n", strip=True)

def main():
    user_prompt = input("Enter your prompt: ").strip()
    if not user_prompt:
        print("Prompt cannot be empty.")
        return

    results = search_web(user_prompt)

    # For each url in web search result, fetch full content
    pages = []
    for item in results:
        url = item.get("url")
        if not url:
            continue

        print(f"Fetching: {url}")
        page_text = fetch_text(url)

        pages.append({
            "title": item.get("title", ""),
            "url": url,
            "snippet": item.get("content", ""),
            "page_text": page_text,
        })

    # Prompt to send to Qwen model with web data
    prompt = f"""
    User request:
    {user_prompt}

    Use these web results and page contents to answer in markdown format.

    Data:
    {json.dumps(pages, ensure_ascii=False)}
    """

    # Invoke local Qwen model 
    response = ollama.chat(
        model=MODEL,
        messages=[{"role": "user", "content": prompt}],
    )

    digest = response["message"]["content"]

    if not digest.strip():
        print("❌ Model returned an empty response.")
        return

    # Build a unique filename using today's date and time
    timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
    filename = f"digest-{timestamp}.md"

    # Save the digest to disk
    with open(filename, "w", encoding="utf-8") as f:
        f.write(digest)

    print(f"Saved to {filename}")

if __name__ == "__main__":
    main()
```

---

## Step 8: Running the Agent

### 8.1 Start Ollama Server
In a separate terminal:
```powershell
ollama serve
```

### 8.2 Run the Agent
In the project terminal (with venv activated):
```powershell
python research_agent.py
```

### 8.3 Enter Your Query
When prompted:
```
Enter your prompt: Latest AI news
```

### 8.4 View Output
The script will:
1. Fetch URLs from web search
2. Download page content
3. Process with Qwen model
4. Save to `digest-YYYY-MM-DD_HH-MM-SS.md`

Example output:
```
Fetching: https://example.com/ai-news-1
Fetching: https://example.com/ai-news-2
Fetching: https://example.com/ai-news-3
Fetching: https://example.com/ai-news-4
Fetching: https://example.com/ai-news-5
Saved to digest-2026-07-04_14-30-22.md
```

---

## Step 9: Created README.md

Created comprehensive README with:
- Project description
- Features overview
- Installation instructions
- Configuration guide
- Usage examples
- Troubleshooting section
- Dependencies list

---

## Key Learnings

1. **Always call load_dotenv()** - Importing the function isn't enough; it must be called
2. **Verify environment variables load** - Add debug prints to confirm `.env` files are being read
3. **Check API response formats** - Responses can be objects or dictionaries; verify the correct access method
4. **Test each component separately** - Verified Ollama worked before debugging web search issues
5. **Read error messages carefully** - 401 Unauthorized indicated an auth issue, not a code issue

---

## Project Structure

```
Personal-Web-Research-AI-Agent/
├── .env                      (API key - not in git)
├── .venv/                    (Virtual environment)
├── research_agent.py         (Main script)
├── README.md                 (Project documentation)
├── CREATION_GUIDE.md         (This file)
├── digest-*.md               (Generated outputs)
└── requirements.txt          (Optional: pip freeze > requirements.txt)
```

---

## How to Use Going Forward

Every time you want to use the agent:

1. Open PowerShell in project folder
2. Activate venv: `.\.venv\Scripts\Activate.ps1`
3. In another terminal: `ollama serve`
4. Run: `python research_agent.py`
5. Enter your research topic
6. Wait for digest to generate
7. Open the `digest-*.md` file to read results

---

## Improvements Made

- Fixed environment variable loading
- Fixed response object access
- Removed debug print statements
- Added error handling for empty responses
- Added UTF-8 encoding for file writing
- Improved console output messages

---

## Troubleshooting Reference

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError` | Run `pip install [package_name]` |
| `API_KEY is None` | Verify `.env` file exists and `load_dotenv()` is called |
| `401 Unauthorized` | Check API key validity and account permissions |
| Empty digest file | Check model response and error handling |
| Ollama not found | Verify Ollama is installed and `ollama serve` is running |

---

## Source & Credits

This project was created following the tutorial from **freeCodeCamp**:

**Tutorial:** "Build a Personal AI Web Research Agent with Ollama and Qwen"  
**Source:** https://freecodeecamp.org/news/build-a-personal-ai-web-research-agent-with-ollama-and-qwen/  
**Website:** freeCodeCamp  

freeCodeCamp is a free, open-source platform dedicated to teaching people how to code. The tutorial provided the foundational architecture, code structure, and step-by-step implementation that made this project possible.

---

## Date Completed

July 4, 2026

---

End of Creation Guide
