# re:Invent 2026 Session Submission

## Title
**Cost-Optimized AI Coding Agents: Running Claude Code with 38 Models on Amazon Bedrock**

## Session Type
Builders' Session (BLD) — 60 min, hands-on

## Abstract (300 words max)

AI coding agents like Claude Code are transforming software development, but running them exclusively on frontier models is expensive at scale — $15/M output tokens for routine tasks like test generation or boilerplate refactoring. What if you could route 70% of those tasks to models that cost 5-20x less, with no loss in quality?

In this session, we demonstrate a production-ready architecture that extends Claude Code to leverage 38+ non-Anthropic models via Amazon Bedrock Mantle — including DeepSeek V3, Qwen Coder, Kimi K2.5, Mistral, and NVIDIA Nemotron — all through a single LiteLLM proxy that translates between the Anthropic Messages API and OpenAI Chat Completions.

We present benchmark results from a 5-task coding evaluation (bug fixes, test writing, feature implementation, refactoring, circular import resolution) across 5 models at different price points. Our findings show that Qwen Coder 30B ($0.15/M input) scores 4.2/5 on code quality — 93% of Claude Sonnet's 4.5/5 — at 20x lower cost. Kimi K2.5 achieves 4.1/5 at 5x lower cost. Claude Opus serves as LLM-as-judge, evaluating actual generated code files on correctness, quality, completeness, and efficiency.

Attendees will:
- Set up a multi-model proxy for Claude Code in under 10 minutes
- Understand Bedrock Mantle's Chat Completions API and bearer token auth
- See live benchmarks routing tasks by complexity to cost-appropriate models
- Learn a task-routing strategy that achieves 55-70% cost reduction for engineering teams running AI coding agents at scale

This directly implements the model routing recommendations from Anthropic's "Serving a Trillion Tokens a Month" whitepaper, using native AWS infrastructure with zero external dependencies.

## Target Audience
Developers, ML engineers, and platform teams running AI coding assistants at scale who want to reduce inference costs without sacrificing quality.

## AWS Services Used
- Amazon Bedrock (Anthropic models — native)
- Amazon Bedrock Mantle (38 third-party models — Chat Completions API)
- Amazon EC2 (self-hosted alternative path with vLLM)

## Key Takeaways
1. Bedrock Mantle provides a unified OpenAI-compatible endpoint for 38 models from 12+ providers — no vendor-specific SDKs needed
2. A LiteLLM proxy bridges Claude Code's Anthropic API to Mantle's Chat Completions API with full tool-use and streaming support
3. Benchmark data proves cheaper models (5-20x less) match frontier model quality on routine coding tasks
4. Task-routing by complexity (frontier for architecture, budget for boilerplate) delivers 55-70% cost reduction at enterprise scale

## Speaker Bio
Prateek Shekhar — Solutions Architect, AWS. Building production architectures for AI coding agents with multi-model inference on Amazon Bedrock. Maintainer of open-source multi-model Claude Code integrations.

## Demo/Hands-on Plan
1. **Setup (5 min)**: Launch proxy with `setup-proxy.sh`, show 38 models healthy
2. **Single-model demo (5 min)**: Run Claude Code with Qwen Coder 30B fixing a bug live
3. **Benchmark (10 min)**: Run 5-task suite across 3 price tiers, show pass/fail + latency
4. **Judge (5 min)**: Claude Opus evaluates code quality across models
5. **Cost analysis (5 min)**: Show cost per task, extrapolate to team-scale savings
6. **Architecture deep-dive (15 min)**: Mantle auth, proxy config, task routing strategies
7. **Q&A (15 min)**

## Supporting Materials
- GitHub: github.com/shekharprateek/claude-code-multi-model-bedrock
- Benchmark results: 5 models × 5 tasks, deterministic + LLM-as-judge scoring
- References: Anthropic "Serving a Trillion Tokens a Month" whitepaper [14], [35]
