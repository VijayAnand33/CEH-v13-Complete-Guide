 # ShellGPT

 ## Overview

 ShellGPT (SGPT) is an AI-powered command-line assistant that integrates Large Language Models (LLMs) into the terminal. It assists users by generating shell commands, explaining technical concepts, automating repetitive tasks, and providing contextual guidance during cybersecurity assessments and system administration.

 ---

 ## Purpose

 The primary purpose of ShellGPT is to improve productivity by assisting users with command generation, troubleshooting, concept explanation, and workflow automation. During penetration testing, it serves as an intelligent assistant that supports learning and operational efficiency while still requiring human validation.

 ---

 ## Key Features

 - AI-powered command generation
 - Technical concept explanation
 - Shell command assistance
 - Terminal integration
 - Context-aware responses
 - Workflow automation support
 - Interactive learning

 ---

 ## Installation

 ### Python / pip

 ```bash
 pip install shell-gpt
 ```

 Alternatively using `pipx`:

 ```bash
 pipx install shell-gpt
 ```

 ### Configure API Key

 Set your OpenAI API key before using ShellGPT.

 Linux / macOS (bash/zsh):

 ```bash
 export OPENAI_API_KEY="<Your_API_Key>"
 ```

 Windows PowerShell:

 ```powershell
 setx OPENAI_API_KEY "<Your_API_Key>"
 ```

 ---

 ## Common Options / Commands

 | Command | Description |
 |--------|-------------|
 | `sgpt "<Prompt>"` | Send a natural-language prompt to ShellGPT and receive guidance, command suggestions, or explanations. |
 | `sgpt --shell "<Prompt>"` | Request shell command(s) tailored to the prompt (returns suggested commands). |
 | `sgpt --code "<Prompt>"` | Request code snippets or small scripts (e.g., Bash, Python). |
 | `sgpt --chat` | Start an interactive chat session with ShellGPT in the terminal. |
 | `sgpt --version` | Print the installed ShellGPT version. |
 | `sgpt --model <name>` | Specify the LLM model to use for the request (overrides default). |
 | `sgpt --temperature <0-1>` | Control response creativity (lower = deterministic, higher = creative). |

 ---

 ## Typical Workflow

 ```mermaid
 %%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#D0E8FF', 'primaryTextColor': '#092B4B', 'primaryBorderColor': '#1C6DD0', 'lineColor': '#2A7EE0', 'secondaryColor': '#EBF5FF', 'tertiaryColor': '#BBDFFF'}}}%%
 flowchart TD
	 A[Define Objective] --> B[Prompt ShellGPT]
	 B --> C[Receive Commands / Guidance]
	 C --> D[Review AI Response]
	 D --> E{Accept Suggestion?}
	 E -->|Yes| F[Execute Approved Commands]
	 E -->|No| G[Refine Prompt]
	 G --> B
	 F --> H[Analyze Results]
	 H --> I[Document Findings]
	 I --> J[End]

	 classDef start fill:#D6E9FF,stroke:#145DA0,stroke-width:2px,color:#092B4B;
	 classDef process fill:#FFFFFF,stroke:#1C6DD0,stroke-width:1.5px,color:#0D1B2A;
	 classDef decision fill:#FFF4CC,stroke:#E09F3E,color:#BF360C;
	 classDef endNode fill:#DCEED9,stroke:#2E7D32,stroke-width:2px,color:#1B5E20;

	 class A start;
	 class B,C,D,F,G,H,I process;
	 class E decision;
	 class J endNode;
 ```

 ---

 ### Verify Installation

 ```bash
 sgpt --version
 ```

 ---

 ## Basic Syntax

 ```bash
 sgpt "<Prompt>"
 ```

 Example:

 ```bash
 sgpt "Generate an Nmap command to detect service versions."
 ```

 ---

 ## CEH Practical Example

 During **Module 14 – Web Application Hacking**, ShellGPT was used as an AI assistant to support web application security testing. It helped generate security-related commands, explain penetration testing concepts, and assist with vulnerability assessment activities. The AI-generated responses were reviewed and validated before being applied during the assessment.

 ---

 ## Advantages

 - Improves productivity.
 - Reduces time spent searching for commands.
 - Assists with technical explanations.
 - Supports learning while performing practical tasks.
 - Integrates directly with the terminal.

 ---

 ## Limitations

 - AI-generated responses may be inaccurate or incomplete.
 - Requires internet connectivity and API access.
 - Should not replace professional judgment.
 - Generated commands should always be reviewed before execution.

 ---

 ## Best Practices

 - Review every AI-generated command before executing it.
 - Never assume AI responses are always correct.
 - Avoid sharing sensitive information in prompts.
 - Use ShellGPT as an assistant rather than a replacement for technical knowledge.
 - Validate all security findings manually.

 ---

 ## Used In

 - Module 02 – Footprinting and Reconnaissance
 - Module 03 – Scanning Networks
 - Module 04 – Enumeration
 - Module 06 – System Hacking
 - Module 13 – Hacking Web Servers
 - Module 14 – Web Application Hacking
 - Module 15 – SQL Injection

 ---

 ## Related Tools

 - Nmap
 - Burp Suite
 - Wapiti
 - cURL

 ---

 ## References

 - Official GitHub Repository: https://github.com/TheR1D/shell_gpt