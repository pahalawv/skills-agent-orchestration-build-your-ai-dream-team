# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate the work on Mona's
Project Pulse dashboard.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 4.7 (copilot) | Coordinates the specialists, assigns file scopes and phases, manages dependencies, and verifies the integrated result. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the repository, documentation, dependencies, and edge cases, then creates an implementation plan. | `.github/agents/planner.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Defines the dashboard's UX, accessibility, information hierarchy, responsive behavior, and visual styling. | `.github/agents/designer.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements assigned code and support configuration, then validates the changes. | `.github/agents/coder.agent.md` |
