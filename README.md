# Building Research Agents for Tech Insights 🚀

*Salam, that's Raneem! 👋*

Ever tried asking ChatGPT to "scout all of tech for me and summarize trends based on what I think I'd find interesting"? Yeah, you probably got something pretty generic - maybe it searched a few websites, grabbed some news articles, and called it a day. That's because ChatGPT is built for general use cases, not for the deep, personalized tech scouting we actually need.

<img width="1400" height="782" alt="image" src="https://github.com/user-attachments/assets/41d45768-b14a-41dd-85a7-b9f1c3fbc104" />

This project shows you how to build a specialized agent that can actually scout the entire tech landscape, aggregate millions of texts, filter data based on your specific persona, and surface patterns and themes you can actually act on. No more endless scrolling through forums and social media - let the agent do the heavy lifting for you.

<img width="1400" height="615" alt="image" src="https://github.com/user-attachments/assets/c43eeb51-cfff-4e10-b1a4-319f41884678" />

## What Makes This Different? 🎯

The magic happens through three key components:
- A unique data source that goes way beyond surface-level content
- A controlled workflow that ensures quality and relevance  
- Smart prompt chaining techniques that build insights systematically

<img width="1400" height="1211" alt="image" src="https://github.com/user-attachments/assets/99e8440a-b9fa-472b-9c83-64abe495038b" />

By caching data strategically, we keep costs down to just a few cents per report while delivering enterprise-level insights.

<img width="1400" height="669" alt="image" src="https://github.com/user-attachments/assets/940bd917-1870-49df-aa7c-7c0967ef967b" />


## Architecture Philosophy 🏗️

If you're new to building agents, this might not look "groundbreaking" at first glance. But here's the thing - if you want to build something that actually works in production, you need solid software engineering principles behind your AI applications. LLMs might be getting smarter, but they still need proper guidance and guardrails.

For workflows with clear paths (like this one), structured "workflow-like" systems work best. Save the dynamic approaches for when you have humans in the loop.

The secret sauce here is the data moat - without a quality data source, this workflow couldn't outperform ChatGPT. That's where the real competitive advantage lies.

## Data Pipeline: The Foundation 📊

Most people get this wrong when working with LLM systems - they think AI can magically process and aggregate data entirely on its own. We're not there yet in terms of reliability, so we need clean data pipelines just like any other system.

### How Our Data Source Works

The system ingests thousands of texts from tech forums and websites daily, using small NLP models to:
- Break down main keywords
- Categorize content
- Analyze sentiment
- Track trending keywords across categories over time

<img width="1400" height="775" alt="image" src="https://github.com/user-attachments/assets/9b45e9cd-c60f-42ab-af1c-4b198912d4f7" />

### The "Facts" Extraction Process

I added a special endpoint that collects "facts" for each keyword:

1. Receives a keyword and time period
2. Sorts comments/posts by engagement
3. Processes texts in chunks with smaller models
4. Decides which "facts" to keep based on relevance

<img width="1400" height="825" alt="image" src="https://github.com/user-attachments/assets/f337486d-8f52-440f-a1e7-86890f843564" />

The final LLM step summarizes the most important facts while keeping source citations intact.

<img width="1400" height="494" alt="image" src="https://github.com/user-attachments/assets/67870986-af23-40da-9d35-baf1e833cc1c" />

This mimics LlamaIndex's citation engine through prompt chaining. First run takes up to 30 seconds, but caching makes repeat requests lightning-fast (few milliseconds).

## Model Selection Strategy 🤖

Here's something everyone's thinking about right now - when to use small vs. large models.

As we chain more LLM calls together, costs add up fast. So use smaller models whenever possible:

**Great for small models:**
- Citing and grouping sources
- Routing decisions  
- Parsing natural language into structured data

**Reserve larger LLMs for:**
- Finding patterns in very large texts
- Human communication
- Complex reasoning tasks

In this workflow, costs stay minimal because:
- Data is cached
- Most tasks use smaller models
- Only final steps require large LLM calls

## Agent Architecture Deep Dive 🔍

The agent runs in Discord (though that's not the focus here). The process splits into two main parts:

### Part 1: Profile Setup

<img width="1400" height="620" alt="image" src="https://github.com/user-attachments/assets/188cdea4-e4e8-4f6a-8e8a-742408f5db42" />

Since I know the data source intimately, I built an extensive system prompt that helps the LLM translate user inputs into fetchable data parameters:

```python
PROMPT_PROFILE_NOTES = """
You are tasked with defining a user persona based on the user's profile summary.

Your job is to:
1. Pick a short personality description for the user.
2. Select the most relevant categories (major and minor).
3. Choose keywords the user should track, strictly following the rules below (max 6).
4. Decide on time period (based only on what the user asks for).
5. Decide whether the user prefers concise or detailed summaries.

Step 1. Personality
- Write a short description of how we should think about the user.
- Examples:
  - CMO for non-technical product → "non-technical, skip jargon, focus on product keywords."
  - CEO → "only include highly relevant keywords, no technical overload, straight to the point."
  - Developer → "technical, interested in detailed developer conversation and technical terms."
[...]
"""
```

The output follows a strict schema:

```python
class ProfileNotesResponse(BaseModel):
    personality: str
    major_categories: List[str]
    minor_categories: List[str]
    keywords: List[str]
    time_period: str
    concise_summaries: bool
```

Without domain knowledge of the API, an LLM wouldn't figure this out alone. I always use structured JSON outputs for validation - if validation fails, we re-run automatically.

### Part 2: Report Generation

When users trigger `/news`, here's what happens:

1. **Fetch user profile** - Get stored context for relevant data fetching
2. **Get trending keywords** - Based on categories and time period (default: weekly)

<img width="1400" height="553" alt="image" src="https://github.com/user-attachments/assets/44a91138-35cc-4133-bed5-1a2bb94bd40c" />

3. **Filter relevance** - Though I could add an LLM step here to filter irrelevant keywords
4. **Fetch cached facts** - Use parallel calls for speed (first-time requests take longer)
5. **Process and deduplicate** - Combine data, remove duplicates, parse citations
6. **Generate themes** - First LLM finds 5-7 themes ranked by relevance

<img width="1400" height="701" alt="image" src="https://github.com/user-attachments/assets/e934fa7c-8098-44e0-88c7-43080b36ef2a" />

7. **Create final report** - Second LLM generates summaries and title using themes + original data

The final step uses a reasoning model like GPT-5 for best results, though you could swap for something faster.

<img width="800" height="535" alt="image" src="https://github.com/user-attachments/assets/0485ff29-0ded-43cd-a659-dd9589025882" />

Total process time: a few minutes, depending on cached data availability.

## Key Engineering Insights 💡

Every agent is different, so this isn't a universal blueprint. But notice the software engineering level required - LLMs don't eliminate the need for good software and data engineers.

I'm mostly using LLMs to translate natural language into JSON, then moving that through the system programmatically. It's not what people imagine when they think "AI applications," but it's the most reliable approach for production systems.

Free-moving agents work better when humans are in the loop, but for automated workflows like this, structure wins every time.

## Getting Started 🚀

### Prerequisites
- Python 3.8+
- MongoDB (optional, for storing profiles)
- Discord bot token (if using Discord interface)
- Access to your preferred LLM APIs

### Installation

```bash
git clone https://github.com/ilsilfverskiold/ai-personalized-tech-reports-discord.git
cd ai-personalized-tech-reports-discord
pip install -r requirements.txt
```

### Configuration

1. Set up your environment variables
2. Configure your data source endpoints
3. Set up Discord bot (if using Discord interface)
4. Initialize your database

### Usage

Check out the detailed implementation in the source code - I've focused this README on architecture and concepts rather than step-by-step coding details.

## Future Improvements 🔮

I have plans to enhance this further, but I'm always open to feedback! Some ideas:
- Content generation capabilities
- More data sources
- Improved filtering algorithms
- Better caching strategies

## Contributing 🤝

Feel free to fork this project and make it your own! Whether you want to:
- Add new data sources
- Improve the agent logic
- Build different interfaces
- Create specialized versions for other domains

The architecture is flexible enough to support various adaptations.

## What You've Learned 📚

Hopefully, this gives you insight into:
- Building production-ready AI agents
- The importance of data preparation
- When to use different model sizes
- Structured vs. free-form agent approaches
- Practical prompt chaining techniques

Remember - the real competitive advantage comes from your data moat and engineering discipline, not just the LLM itself.

---

*Built with ❤️ by Raneem*

*Want to build something similar? Start with a solid data foundation and work your way up!*
