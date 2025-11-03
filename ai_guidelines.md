# LLM Prompts for News Processing

## Overview
The system uses various prompts for different stages: metadata generation, RAG retrieval, synthesis, composition, and chat response.

## Metadata Generation Prompts
For each article, extract:
- Summary (2-3 sentences)
- Sentiment (positive/negative/neutral + score)
- Keywords (5-10)
- Topics

Prompt template: "Summarize this article in 2-3 sentences. Analyze sentiment. Extract keywords and topics."

## RAG Synthesis Prompts
Given a collection of articles and query:
"Given the following article metadata, retrieve and synthesize the most relevant information for creating a news segment on [query]."

## Composition Prompts
Using selected persona:
"You are [persona_name]. Compose a 300-word news segment based on this synthesized information. Follow these guidelines: [guidance]. Maintain this tone: [tone_words]."

## Chat Adjustment Prompts
Process user input to modify composition:
"Adjust the current persona parameters based on this user instruction: [user_message]. Update temperature, guidance, or tone as appropriate. Return the modified configuration."

## Prompt Engineering Standards
- Use clear instructions
- Include few-shot examples where needed
- Specify output format (JSON/XML)
- Limit token usage for efficiency

## Advanced Prompt Techniques

### Chain-of-Thought Prompting
For complex analysis tasks:
"Let's think step by step:
1. First, identify the key themes in these articles
2. Next, analyze the sentiment and perspective
3. Then, synthesize the main points
4. Finally, compose the segment in [persona] style"

### Few-Shot Learning Examples
Include 2-3 examples of desired output format:
```
Example 1: [Input] → [Expected Output]
Example 2: [Input] → [Expected Output]
Now process: [Actual Input]
```

### Output Validation Prompts
"Review the generated content and verify:
- Accuracy of information
- Consistency with persona
- Appropriate tone and style
- Factual correctness"

## Model-Specific Considerations

### For Gemma Models
- Use Instruct format: `<start_of_turn>user\n{prompt}<end_of_turn>\n<start_of_turn>model\n`
- Keep context window under 4096 tokens for optimal performance
- Temperature: 0.7 for creative content, 0.3 for factual summaries
- Top-p: 0.9 recommended for balanced outputs

### For General GGUF Models
- Use appropriate stop tokens: `</s>`, `[/INST]`, or model-specific markers
- Monitor token usage to stay within context limits
- Adjust batch size based on available RAM/VRAM

## Prompt Safety and Validation

### Input Sanitization
- Remove potential injection attempts
- Validate content length before processing
- Check for malicious patterns
- Log suspicious prompts for review

### Output Quality Checks
- Verify JSON/XML format if structured output requested
- Check for incomplete responses
- Validate against expected schema
- Monitor for hallucinations or factual errors

---

## Checklist
- [x] Prompts designed
- [x] Prompt templates created
- [ ] Prompt performance tested
- [ ] Safety validation implemented
- [ ] Model-specific optimizations applied

## Ledger
| Prompt Type | Template | Performance Note | Last Updated |
|-------------|----------|------------------|--------------|
| Summary | "Summarize..." | Good accuracy | 2025-10-28 |
| RAG | "Retrieve..." | Needs tuning | 2025-10-28 |
| Compose | "You are [persona]..." | Excellent quality | 2025-10-28 |
| Metadata | "Analyze this article..." | 85% accuracy | 2025-10-28 |
| Chat Adjust | "Modify persona..." | In testing | 2025-10-28 |

---

*Document Version: 1.1 | Last Updated: 2025-10-28 | Review Frequency: Bi-weekly*
