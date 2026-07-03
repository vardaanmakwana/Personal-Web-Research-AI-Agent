# Personal Web Research AI Agent

An intelligent web research agent that automatically searches the web, fetches content, and generates AI-powered markdown summaries using a local language model.

## Features

- **Web Search Integration**: Searches the web using Ollama's web search API
- **Content Extraction**: Automatically fetches and cleans full page content from search results
- **Local AI Processing**: Uses Qwen model running locally via Ollama for privacy
- **Markdown Output**: Generates well-formatted markdown summaries
- **Auto-saved Results**: Saves digests with timestamped filenames

## Prerequisites

- Python 3.8+
- [Ollama](https://ollama.ai) installed and running
- Qwen model downloaded locally (`qwen2.5:4b` or similar)
- API key for Ollama web search functionality

## Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/Personal-Web-Research-AI-Agent.git
   cd Personal-Web-Research-AI-Agent
   ```

2. **Create a virtual environment**
   ```bash
   python -m venv .venv
   .venv\Scripts\activate  # On Windows
   # source .venv/bin/activate  # On macOS/Linux
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables**
   Create a `.env` file in the project root:
   ```env
   OLLAMA_API_KEY=your_api_key_here
   ```

## Configuration

- **MODEL**: Set the Ollama model in `research_agent.py` (default: `qwen2.5:4b`)
- **SEARCH_URL**: The Ollama web search endpoint (default: `https://ollama.com/api/web_search`)
- **Max Results**: Number of search results to fetch (default: 5)

## Usage

Run the agent from the command line:

```bash
python research_agent.py
```

Then enter your research prompt when prompted:

```
Enter your prompt: What are the latest developments in quantum computing?
```

The agent will:
1. Search the web for your query
2. Fetch full content from each result
3. Pass the content to the Qwen model for analysis
4. Generate a markdown summary
5. Save the result as `digest-YYYY-MM-DD_HH-MM-SS.md`

## Example Output

```
digest-2026-07-04_14-30-22.md
digest-2026-07-04_15-45-13.md
```

## Dependencies

- `requests` - HTTP requests for web search and page fetching
- `beautifulsoup4` - HTML parsing and content extraction
- `ollama` - Python client for Ollama
- `python-dotenv` - Environment variable management

## How It Works

1. **Web Search**: Submits user query to Ollama's web search API
2. **Content Fetching**: Downloads HTML and extracts clean text from each result
3. **Cleaning**: Removes scripts, styles, navigation, and footer elements
4. **AI Processing**: Sends aggregated data to local Qwen model
5. **Output**: Generates markdown summary and saves it locally

## Requirements

Ensure Ollama is running before executing the script:

```bash
ollama serve
```

Pull the required model if not already available:

```bash
ollama pull qwen2.5:4b
```

## Troubleshooting

- **Connection Error**: Ensure Ollama is running (`ollama serve`)
- **Empty Response**: Check if model is properly loaded and has sufficient context
- **API Key Issues**: Verify `OLLAMA_API_KEY` is correctly set in `.env`
- **Timeout Errors**: Increase timeout values in the code if fetching slow websites

## Author

Created by Vardaan Makwana as a personal research automation tool

---

**Note**: This agent uses only local resources for AI processing, ensuring privacy and offline capability.
