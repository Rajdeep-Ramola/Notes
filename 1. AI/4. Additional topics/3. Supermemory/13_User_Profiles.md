# 👤 User Profiles (API)

> Guide to fetching user profiles and building LLM prompts using Supermemory profile data.

---

## 📑 Table of Contents

1. [Profile + Search](#1-profile--search)
2. [Building Prompts](#2-building-prompts)

---

## 1. Profile + Search

Getting profile and search results in one call by using the `q` parameter:

```python
result = client.profile(
    container_tag="user_123",
    q="deployment errors"
)

# Profile data
facts = result.profile.static
context = result.profile.dynamic

# Search results
memories = result.search_results.results if result.search_results else []
```

---

## 2. Building Prompts

Injecting profile into an LLM's system prompt:

```python
async def chat(user_id: str, message: str):
    # Fetch user profile from Supermemory
    response = await client.profile(containerTag=user_id)
    profile = response["profile"]

    system_prompt = f"""You are assisting a user.

ABOUT THE USER:

{chr(10).join(profile.get("static", [])) or "No profile yet."}

CURRENT CONTEXT:

{chr(10).join(profile.get("dynamic", [])) or "No recent activity."}

Personalize responses to their expertise and preferences.
"""

    return await llm.chat(
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": message},
        ]
    )
```
