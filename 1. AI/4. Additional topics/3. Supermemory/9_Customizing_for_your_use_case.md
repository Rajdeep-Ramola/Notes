# ⚙️ Customizing for Your Use Case

> Tell Supermemory what content matters during ingestion using filter prompts to help filter and prioritize what gets indexed.

---

## 📑 Table of Contents

1. [Filter Prompts](#1-filter-prompts)

---

## 1. Filter Prompts

Filter prompts tell Supermemory what content matters during ingestion. They help filter and prioritize what gets indexed.

```javascript
// Example: Brand guidelines assistant
await client.settings.update({
  shouldLLMFilter: true,
  filterPrompt: `You are ingesting content for Brand.ai's brand guidelines system.

Index:
- Official brand values and mission statements
- Approved tone of voice guidelines
- Logo usage and visual identity docs
- Approved messaging and taglines

Skip:
- Draft documents and work-in-progress
- Outdated brand materials (pre-2024)
- Internal discussions about brand changes
- Competitor analysis docs`
});
```
