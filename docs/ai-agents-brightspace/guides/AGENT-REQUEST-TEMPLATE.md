# How to Request an Agent from a Coding Agent

> **This section is for humans.** The prompt template below (after the horizontal rule) is what you copy and give to the coding agent.

## Instructions for Designers

1. **Open a coding agent session** (e.g. Cursor, Copilot Chat, or similar) in this repository.
2. **Copy everything below the `---` line.**
3. **Fill in all fields** marked with `[square brackets]`. Replace the bracket and its contents entirely with your answer.
4. **Paste it as your first message** to the coding agent.

The coding agent will read `AGENT-GUIDE.md` to understand the framework before writing any code. You do not need to explain the framework to it.

### Tips for filling out the template

- **Agent name**: Use lowercase with hyphens (kebab-case), e.g. `accessibility-review`, `quiz-validator`.
- **Inputs from Brightspace**: Think about what data the LLM needs to do its job. Does it need to read the course content? The gradebook? A JSON file you've uploaded?
- **Tool categories**: These control which Brightspace features the LLM can call. Only list what the agent actually needs. Categories: `content`, `course`, `discussions`, `grades`, `quizzes`, `dropbox`, `rubrics`, `other`.
- **Output format**: Describe the structure of what the agent should produce. The more specific, the better.
- **Prompt/instructions for the LLM**: Write this as if you were giving instructions to a person. The coding agent will turn this into the LLM system prompt.

---

## Prompt to paste into your coding agent

I want to implement a new Brightspace agent plugin in this repository.
Read `AGENT-GUIDE.md` for all framework conventions before starting.

**Agent name:** [kebab-case name, e.g. `accessibility-review`]

**What it does:**
[One or two paragraphs describing the agent's purpose. What problem does it solve? What does a user do with it?]

**Inputs it needs from Brightspace:**
[List the data sources the agent needs. Examples:]
- [Reads course content (modules, topics, files)]
- [Reads the gradebook structure]
- [Reads a JSON file from course files at `data/myData.json`]
- [Reads discussion forums and posts]

**What the LLM should analyze or produce:**
[Describe what the AI model should do with those inputs. Examples:]
- [Compare the course structure to a template and flag differences]
- [Evaluate each quiz question against accessibility standards]
- [Generate a structured list of recommendations]

**Output / report format:**
[Describe how the output should be structured. Examples:]
- [A report with a summary paragraph and a list of findings, each with a label, severity (ok/warning/issue), and detail text in markdown]
- [A list of recommendations grouped by category, saved to course files as a JSON file]
- [A pass/fail result per rubric criterion with supporting evidence]

**Tool categories the LLM needs:**
[Choose from: `content`, `course`, `discussions`, `grades`, `quizzes`, `dropbox`, `rubrics`, `other`. List only what the agent needs.]

**Any custom data the agent needs:**
[Describe any JSON files, catalogs, or static data the agent reads from course files or that are bundled with the agent. Example: "a JSON file at `data/masterCourses.json` with a list of master course URLs — see the `template-fidelity` agent for an example of this pattern." If none, write "None."]

**Prompt / instructions for the LLM:**
[Write in plain language what the LLM prompt should tell the AI model to do. Include any rules, priorities, output constraints, or tone. Example: "Review each content module for alignment with the provided rubric. For each topic, determine whether it meets, partially meets, or does not meet each criterion. Be specific and cite the topic name. Focus on factual gaps — do not penalize for style choices."]

**UI notes:**
[Optional. Describe any UI requirements beyond the standard layout (a selector for the user to choose an input, a list of saved reports, a progress indicator during analysis). If the standard MainView + run button + report list pattern is fine, write "Standard layout."]
