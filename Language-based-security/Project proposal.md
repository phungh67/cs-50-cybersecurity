
Topic: prompt-injection, which was originally a way to trick the chat-bot or LLM agents to do some disturbing commands (echo Hello Worlds, completely ignore user's inputs...) to execute harmful shell instructions. With the rapidly development of AI assistants (Claude, Copilot, Gemini,...) and the form of guiding rules (via `instructions.md`, `.cursorule` or other files within the working space), the attack surfaces are broader than ever.

There are three types:
- **Coding Assistants Vulnerable**: taking the advantages of modifying instructions to force these agents to execute shell commands illegally.
- **RAG**: poisoned the documents, especially in enterprise environment (Office 365 applications), when AI tried to summary the contents, it also compromised the software.
- **Web-agentic tool**: Hidden text on a website to steals user session tokens or redirects transactions.

# 1. Threat model

First, we will consider that capabilities of the attackers, we assume that it would be infeasible to directly modify the agent's system prompt (the answer for user's commands), intercept the communication between user and agent or completely control the host machine further that what agents exposed.

Nevertheless, there are still several things that adversaries can do:
- **Content injector**: place malicious content in the repositories, publish documents, web pages, but of course cannot access private repositories or authenticated systems (because it's just "content").
- **Tool publisher**: in this case, they even bring their own compromised agents into official marketplaces, it can be varies from MCP servers, skills or extensions.
- **Network attacker**: this is one of basic way, because of the "fake" agent, they can play man-in-the-middle attack or even use DNS manipulation for redirect attack.

Next, after successfully conducted attack, it is meat to be achieve one or more:
- Data exfiltration: stealing source code, sensitive api keys, environment variables,...
- Code injection: inserting backdoor, malware, supply chain attacks,...
- Privilege escalation: gaining escalated access within the system or expanding to other services.
- Denial of service attack.
- Persistence: establishing on going access through configuration changes or installed backdoor.

To avoid these things, at least, one of these boundaries should be met:
- User-agent: instructions from users are in the highest priority, can not be overlapped or overwritten by external content.
- Agent-tool: tool-response should not be treated as safe, it must be treated as unsafe, not executable instructions.
- Tool-tool: tool should not be able to influence or hijack other tool's behavior.
- Session: old session and new session are completely separated.

Currently, almost 73% of tested platforms cannot meet at least one boundary.

# 2. Attack dimensions

## 2.1. Delivery vector