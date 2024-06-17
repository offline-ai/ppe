# Programmable Prompt Engine Specification(Draft)

The [offline-ai/cli](https://github.com/offline-ai/cli) is an implementation of the [Programmable Prompt Engine Specification](https://github.com/offline-ai/ppe).

This is a specification for a programmable prompt engine.

## Quick Start

Welcome to the streamlined guide for getting started quickly with your AI-powered scripting experience. This guide focuses on making the process of creating and executing interactive scripts more intuitive and straightforward. Let's dive in!

### Essential Tips

* Script Return Value: The script's final command's output determines its return value.
* Auto-Execution: Scripts ending with prompts but no explicit `$AI` call will automatically execute `$AI` at the end, configurable via `autoRunLLMIfPromptAvailable`.
* Output Mode: Scripts default to streaming output, but not all LLM backends support this feature.

### Structuring Dialogue

Each line represents a conversation turn, attributed to either `system`,`assistant`, `user`, or implied `user` if not stated:

```yaml
system: "You're an AI assistant."
"user: What's 10 plus 18?"
```

A triple dash (`---`) or asterisks (`***`) initiates a new dialogue, resetting context:

```yaml
system: "You're an AI."
---
"user: What's 10 plus 18?"
$AI                 # Executes the AI
$print: "?=result"  # Prints AI response
---                 # New dialogue starts here
"user: What's 10 plus 12?"
$AI
```

### Input & Output Customization

To build reusable prompt templates, utilize [Front Matter](https://jekyllrb.com/docs/front-matter/) at the file's top:

```yaml
---
input:
  - source_lang     # Language of input; "auto" by default for auto-detection
  - source_text: {required: true}  # Mandatory content for translation
  - target_lang: {required: true}  # Target language
output:
  type: "object"
  properties:
    target_text:
      type: "string"
    source_text:
      type: "string"
    source_lang:
      type: "string"
    target_lang:
      type: "string"
  required: ["target_text", "source_text", "source_lang", "target_lang"]
source_text: "Text to translate."  # Default input value
target_lang: "Chinese"             # Default target language
---
```

This section defines required inputs and structures expected outputs using [JSON Schema](https://json-schema.org/).

### Message Templates

Leverage [Jinja2](https://en.wikipedia.org/wiki/Jinja_(template_engine))-like templating within prompts and parameters. Templates can be pre-defined or generated dynamically during script execution.

* Templates are rendered when `$AI` is called unless prefixed with `#` for immediate formatting.
* Data sources for templates follow this hierarchy: `function arguments` > `prompt` object > `parameters`.

### Advanced AI Substitutions

Use double square brackets `[[` `]]` for sophisticated AI-driven variable substitution within messages. For instance:

```yaml
assistant: "Here's a joke: [[JOKE]] Enjoy!"
$echo: "?=prompt.JOKE"  # Accesses the AI-generated joke stored in prompt
```

This mechanism allows for dynamic content insertion based on AI responses.

By following these simplified steps, you can efficiently create and manage interactive scripts that leverage AI capabilities seamlessly. Remember, practice and experimentation are key to mastering the nuances of this powerful toolset.

