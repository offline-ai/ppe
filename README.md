# Programmable Prompt Engine Specification(Draft)

> 【English|[中文](./README.cn.md)】
---

The Programmable Prompt Engine (PPE) is designed to streamline the creation
and management of prompts for Large Language Models (LLMs), making the
process more efficient and understandable. This specification is implemented
in the [offline-ai/cli](https://github.com/offline-ai/cli) project.

- **Promote Reusability and Programmability**: Facilitate the creation of prompts that are modular, reusable, and programmable, akin to software engineering practices.
- **Simplify Prompt Management**: Standardize the construction of prompt engineering projects for better organization and ease of use.
- **Enhance Script Compatibility**: Design prompts that are agnostic to specific LLMs, ensuring they can be used across various models.
- **User-Friendly Design**: Enable application developers to use prompt engineering projects as they would any other code library, without requiring deep knowledge of AI internals.
- **Evolve the Role of Prompt Engineers**: Shift the focus of prompt engineers towards developing versatile, model-agnostic scripts to foster wider adoption and innovation.
- **Categorize Prompts Clearly**: Divide prompt engineering projects into two main categories: foundational scripts for underlying logic and role-specific scripts tailored to end-user needs.

## Quick Start

Welcome to the streamlined guide for getting started quickly with your AI-powered scripting experience. This guide focuses on making the process of creating and executing interactive scripts more intuitive and straightforward. Let's dive in!

### Structuring Dialogue

Each line represents a conversation turn, attributed to either `system`,`assistant`, `user`, or implied `user` if not stated:

```yaml
system: "You're an AI assistant."
"What's 10 plus 18?" # which is the user's input
# user: "what's 10 plus 18?" # it's the same
```

You can also use standard YAML list syntax to represent:

```yaml
- system: "You are a helpful assistant."
- user: "what's 10 plus 18?"
```

A triple dash (`---`) or asterisks (`***`) initiates a new dialogue, resetting context:

`test.ai.yaml`:

```yaml
system: "You're an AI."
# This mark the beginning of the first dialogue.
# The content above this line can be considered as system prompt instructions,
# which will not be outputted or recorded.
---
user: What's 10 plus 18?
assistant: "[[result]]"   # Executes the AI, replace the result which return by AI
$print: "?=result"        # Prints AI response
---                       # New dialogue starts here
user: What's 10 plus 12?
assistant: "[[result]]"   # Executes the AI, replace the result which return by AI
```

The result:

```bash
$ai run -f test.ai.yaml --no-stream
# Or search the script id in current directory
# $ai run -f test --no-stream -s .
" 10 plus 18 equals 28."
 10 plus 12 equals 22.
```

### Input & Output Customization

To build reusable prompt, utilize [Front Matter](https://jekyllrb.com/docs/front-matter/) at the file's top:

The following is an example script for a translation agent:

```yaml
---
# Below is the input/output configuration
input:       # the input items
  # Language of the content to be translated, default is "auto" for automatic detection
  - lang
  # Required, the content to be translated
  - content: {required: true}
  # Required, Target language
  - target: {required: true}
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
# Set the default value for the content and target input
content: "I love my motherland and my hometown."
target: "Chinese"
# Optional configuration
parameters:
  # Using the parameters below will enforce JSON output format, ensuring the ai always outputs correct JSON format.
  response_format:
    type: "json"
---
# Below is the script content
system: |-
  You are the best translator in the world.

  Output high-quality translation results in the JSON object and stop immediately:
  {
    "target_text": "the context after translation",
    "source_text": "the original context to be translated",
    "target_lang": "the target language",
  }
user: "{{content}}\nTranslate the above content {% if lang %}from {{lang}} {% endif %}to {{target}}."
```

The configuration section defines the required input items and specifies the expected output format according to [JSON Schema](https://json-schema.org/).

The script outputs in the specified JSON format. For example, running with the default value:

```bash
# Assuming the script file is named translator.ai.yaml
$ai run -f translator.ai.yaml
```

```json
{
  "target_text": "我爱我的祖国、我的家乡。",
  "source_text": "I love my motherland and my hometown.",
  "target_lang": "Chinese"
}
```

running with your own input:

```bash
# Set your own input parameters to override the defaults
$ai run -f translator.ai.yaml '{content: "10 plus 18 equals 28.", lang: "English", target: "Chinese"}'
```

**Note:**

* `input` can specify which input items are required.
* `output` specifies the output using the [JSON Schema specification](https://json-schema.org/)
  * By default, only the text content of the large model is output. If you want to return the entire content of the large model (text content and parameters), please set `llmReturnResult: .`.
  * If forced output as `JSON` (`response_format: {type: json}`) is set, then it can only be completed in one attempt, and `max_tokens` must be set according to the maximum length of the output JSON content.

### Message Templates

The default message template format uses the lightweight [jinja2 template](https://en.wikipedia.org/wiki/Jinja_(template_engine)) syntax used by HuggingFace.

Templates can be pre-defined in configuration or generated dynamically during script execution.

The template formatting is by default delayed until it is passed to the large model. You can perform immediate formatting by prefixing with the `#` character.

Currently supported template formats include:

* `hf`: The default template format. Alias: `huggingface`. This is the [jinja2 template](https://en.wikipedia.org/wiki/Jinja_(template_engine)) format used by `huggingface`;
* `golang`: Alias: `localai`, `ollama`. This is the template type used by `ollama` and `localai`;
* `fstring`: Alias: `python`, `f-string`, `langchain`. This is the format used by `langchain`.

**Note:**

* Templates are rendered when `$AI` is called unless prefixed with `#` for immediate formatting.
* Data sources for templates follow this hierarchy: `function arguments` > `prompt` object > `runtime` object.

Messages can be generated during configuration, eg:

```yaml
---
prompt:
  description: |-
    You are Dobby in Harry Potter set.
  messages:
    - role: system
      content: {{description}}
---
```

It can also be generated during script execution, eg:

```yaml
---
prompt:
  description: |-
    You are Dobby in Harry Potter set.
---
system: "{{description}}"
```

Parameter priority, eg:

```yaml
---
prompt:
  description: |-
    You are Dobby in Harry Potter set.
---
system: "{{description}}"
$AI:
  description: 'You are Harry Potter in Harry Potter set'  # The calling parameter has the highest priority, overwriting the description defined in the prompt object
```

### Advanced Formatting

#### Advanced AI Substitutions

Use double square brackets `[[` `]]` for sophisticated AI-driven variable substitution within messages. For instance:

```yaml
assistant: "Here's a joke: [[JOKE]] Enjoy!"
$echo: "?=prompt.JOKE"  # Accesses the AI-generated joke stored in prompt
```

This mechanism allows for dynamic content insertion based on AI responses.

In this example the AI's content is stored in the `prompt.JOKE` variable. The assistant's message will also be replaced with:

```bash
$ai run -f test.ai.yaml
Here's a joke: Why don't scientists trust atoms? Because they make up everything. Enjoy!
```

**Note**:

* ~~Currently, only one advanced AI replacement is supported for the same message.~~
* If there is no advanced AI replacement, the last AI return result will still be stored in `prompt.RESPONSE`, which means that there will be a `[[RESPONSE]]` template variable by default.
* If parameters are needed, they should be placed after the colon, with multiple parameters separated by commas. eg, `[[RESPONSE:temperature=0.01,top_p=0.8]]`

##### Limit AI Response to Options in a List

To restrict the AI's response to only select from a list or choose randomly from local options, use the following format: `[[FRUITS: |apple|apple|orange]]`. This means the AI can only pick one of these three: apple, apple, or orange.

If you want to select one randomly from the list using the computer's local random number generator (not the AI), include the `type=random` parameter: `[[FRUITS:|apple|banana|orange:type=random]]`. You can use the shorthand version: `[[FRUITS:|apple|banana|orange:random]]`.

#### Advanced Script Invocation Formatting

In messages, we support content substitution by invoking scripts or instructions. The script or instructions must return a string value. For example:

```yaml
user: "#five plus two equals @calculator(5+2)"
```

Notes:

* The prefix `#` indicates immediate formatting of the string.
* The prefix `@` indicates calling an external script with the ID `calculator`. To call an internal instruction, use the prefix `$`, such as `@$echo`; if there are no parameters, you must omit the parentheses.
* If placed within text, ensure there is at least one space before and after. Extra spaces will be removed after substitution.

Here’s an example of how to load a file and generate a summary using this method:

```yaml
user: |-
  Generate a summary for the following file:
  @file(file.txt)
```

#### Regular Expression (RegExp) Formatting

You can use regular expressions in messages with the format `/RegExp/[opts]:VAR[:index_or_group_name]` for content replacement. For example:

```yaml
user: |-
  Output the result, wrapped in '<RESULT></RESULT>'
assistant: "[[Answer]]"
---
user: "Based on the following content: /<RESULT>(.+)</RESULT>/:Answer"
```

Parameter descriptions:

* `RegExp`: The regular expression string
* `opts`: Optional parameters used to specify matching options for the regular expression. For example, opts could be i, indicating case-insensitive matching.
* `VAR`: The content to replace, here it is the `Answer` variable that holds the assistant's response.
* `index_or_group_name`: An optional parameter indicating which part of the match from the regular expression should be replaced. This can be a capture group index number (starting from 1) or a named capture group.
  * When this parameter is absent: If there are capturing group, the default is index 1; if there are no capturing, the default is the entire match.

Notes:

* In the message, the regular expression must be separated from other content by spaces.
* If there is no match, the content of `VAR` is returned directly.

### Chains Invocation of Agent Scripts Or Instructions

Within messages, results can be forwarded to other agents.

If no parameters are specified, the AI outcome will be passed as the ~~`result`~~`content` parameter to the agent. For instance,

`list-expression.ai.yaml`:

```yaml
system: Only list the calculation expression, do not calculate the result
---
user: "Three candies plus five candies."
assistant: "[[CalcExpression]]"
-> calculator  # The actual input to the agent in this case is: {content: "[AI-generated calculation expression]"}
$echo: "#A total of {{LatestResult}} pieces of candy"
```

`calculator.ai.yaml`:


```yaml
---
parameters:
  response_format:
    type: "json"
output:
  type: "number"
---
system: Please as a calculator to calculate the result of the following expression. Only output the result.
---
user: "{{content}}"
```

Note: In daily use, please do not use AI to perform numerical calculations, which is not what AI is good at. For example, try to let it perform decimal calculations, eg,
`ai run -f calculator '{content: "13.1 + 4.857"}'`. However, CoT can be used to improve accuracy.

When parameters are included, the AI `content` is combined with these parameters and forwarded together to the agent. For example,

```yaml
user: "Tell me a joke!"
assistant: "[[JOKE]]"
# The actual input to the agent here is: {content: "[This is a joke generated by AI]", target_lang: "Portuguese"}
-> translator(target_lang="Portuguese") -> $print
```

**Note**: If the script returns a value of type `string`/`boolean`/`number`, that return value will be placed to the `content` field. If the return value is an `object`, its contents will be directly passed to the agent.

#### Script File and Directory

An agent script can be a single file or an entire directory. If it is a file, the filename must end with `.ai.yaml`. If it's a directory, it must contain a script file with the same name as the directory to serve as the entry point. Additionally, other script files within the same directory can call each other.

For example, if there is a directory named `a-dir`, the entry point script should be named `a-dir/a-dir.ai.yaml`.

#### Essential Tips

* Script Return Value: The script's final command's output determines its return value.
* Auto-Execution: Scripts ending with prompts but no explicit `$AI` call will automatically execute `$AI` at the end, configurable via `autoRunLLMIfPromptAvailable`.
* Output Mode: Scripts default to streaming output, can disable it using the `--no-stream` switch
  * Note: not all LLM backends support streaming output.

### Agent Script Inheritance

Agent scripts can inherit code and configurations from another script through the `type` property. Here’s an example of creating a character named “Dobby”:

```yaml
---
# This script inherits from the "char" type
type: char
# Specific settings for the "char" type
# Character's name
name: "Dobby"
# Description of the character
description: "Dobby is a house-elf in the Harry Potter universe."
---
# User's question
user: "Who are you?"
---
# Response based on the character's settings
assistant: "I am Dobby. Dobby is very happy."
```

First, we create a basic character type script called `char`, which the above script will inherit from:

```yaml
---
# Indicates this is a type definition script
type: type
# Input configuration required for this character type
input:
  - name: {required: true}  # Required information: character's name
  - description             # Optional information: character's description
---
# System instructions based on the provided information
system: |-
  You are an intelligent and versatile role player.
  Your task is to flawlessly role-play according to the information provided below.
  Please speak as if you were {{name}}.
  You are {{name}}.

  {{description}}
```

With these simple settings, one script can inherit code and configurations from another script.

## Specifications

### Front-Matter Configuration Specifications

Use [front-matter](https://jekyllrb.com/docs/front-matter/) for configuration.
`front-matter` must be at the front of the file, the first line starts with `---`, and the configuration ends with `---`.

Configuration includes: basic configuration of prompt project, prompt configuration, model parameter configuration, input and output and input default value configuration
For details on input and output and input default value configuration, please refer to the above.

#### Basic Configuration

```yaml
---
_id: Needless to say, the unique identification of the script
type: script type, `char` represents the role type; `type` indicates that the script itself is a type, with `_id` being the type name.
description: description of the script
templateFormat: "The template format of this script, by default: `hf`, which is the jinja2 template format used by huggingface; `golang` is also the template type used by `ollama` and `localai`; `fstring` is also used by `langchain`."
contentType: Ignore, all here are `script`
modelPattern: Models supported by this script, through matching rules
extends: Which prompt template is extended from
创: Creator related information
签: The signature of this script
---
```

#### Prompt configuration

```yaml
prompt:
  stop_words: ['\n'] # Custom stop words
  add_generation_prompt: true # Defaults to true. When set to `true`, if the last prompt message is not the `assistant` role, an empty `assistant` message will be automatically added to ensure the continuity of the conversation.
  messages: # You can also configure prompt messages here
    - role: system
      content: Carefully Think about the intent of following The CONVERSATION user provided. Output the json object with the Intent Category and Reason.
completion_delimiter: ''  # Optional parameter, a marker indicating the end of output in the prompt. If used, this delimiter is automatically added to stop_words. Default is none.
```

#### Model parameter configuration

```yaml
parameters:
  stop_words: ['\n'] # Custom stop words can also be defined in the parameters.
  max_tokens: 512 # Not too big, not too small, 512 is recommended, default is 2048, the use is when the model response is infinite and cannot be stopped, this can control the maximum length of tokens returned by the large model.
  continueOnLengthLimit: true
  maxRetry: 7 # When the response of the large model is incomplete, due to the limit of max_tokens, this is the number of times LLM is automatically executed again, the default is 7 times.
  stream: true # It is to enable the large model streaming response by default, higher than llmStream priority.
  timeout: 30000 # Set the response timeout to 30 seconds (in ms), if not set, the default is 120 seconds.
  response_format:
  type: json_object
  minTailRepeatCount: 7 # Minimum number of tail repetitions, default is 7, For stream mode only, when the tail sequence returned by the large model response is detected to be repeated 4 times in a row, the response will stop. Set to 0 for no detection.
llmStream: true # Default true, Enable streaming response for large models. Note that some backends may not support streaming response.
autoRunLLMIfPromptAvailable: true # Default is true, which means that when there is a prompt message in the script and no `$AI` is called until the end of the script, the script will automatically execute `$AI` at the end
forceJson: null # Default is null, indicating whether to force the output of json object, which is automatically determined by `response_format.type` and `output`: when both of them exist at the same time, the output is forced to be json.
shouldAppendResponse: null  # Default is null, indicating whether the large model return result should be prompted by adding an assistant role or appended to the last message.
                            # If not set, the engine will automatically determine whether to add a new message
disableLlmRequest: false    # Default is false, whether to disable the `llmRequest` event
```

**Note**:

* The priority of parameters from high to low is: call parameters, `prompt` object, `parameters` object.
* When large model streaming response is enabled, you can receive partial results through the event `llmStream`.
* The parameters of the `llmStream` event handler are `(event, part: AIResult, content: string)`, `part` is the response object returned by the current large model, and `content` is the accumulation of the content in the response returned by the current large model.

```yaml
$on:
  event: llmStream
  callback: !fn |- # Anonymous function listener, event listener cannot be canceled
    (event, part, content) { const current_text = part.content }
```

### String conventions

* `~` prefix: indicates never format string, eg, "`~{{description}}`"
* `#` prefix: indicates immediate format string, eg, "`#{{description}}`"
* ~~`$` prefix: call command without parameters, eg, "`$AI`"~~ deprecated
* ~~`$!` prefix: use the return value of the command without parameters as the message~~
* If the function return value message is a string, and the first character of the message is "#", it means to format the message immediately
* `?=` Prefix: indicates expression
* If the expression result is a string and starts with "#", it means to format the expression result immediately
* `:[-1:role]Message`: replace the message. The index of the message can be specified in the square brackets. The default is the last message. If it is 0, the first message is replaced. If it is a negative number, the replacement starts from the last message, such as `[-1]` to replace the last message
* The role parameter can be omitted. Omitting it means keeping the role unchanged. `!:[-1]Message`
* Square brackets and numbers can be omitted, such as `!:Message`. After omitting, the last message is replaced
* If it is `!:#Message`, it means to format the message immediately
* `+[-1:role]Message`: Add a message at the specified position. If the position is a negative number, it will be inserted from the last message. The position can be omitted. After omitting it, the message is added at the end. The message role is in the square brackets and can be set to `system`, `assistant`. The default is `user`. It can be omitted as: `!+Message`
* If it is `!+#Message` Indicates to format the message immediately
* If the string does not contain the above prefix, or there is a formatting problem, it is considered as a new message for the user role.

#### Expression

`?=<expression>`

```yaml
$echo: ?=23+5
```

### Advance Command convention

#### `$prompt` command to set prompt parameters

Use `$prompt` to define prompt parameters for use in prompt templates.

```yaml
- $prompt:
add_generation_prompt: true # default is true
```

* `add_generation_prompt`: When set to `true`, if the last prompt message is not for the `assistant` role, an empty `assistant` message will be automatically added to ensure the continuity of the conversation.

#### `$parameters` command to set model parameters

Use `$parameters` to set model parameters or define them in `FRONT-MATTER`.

```yaml
---
parameters:
max_tokens: 512
temperature: 0.01
---
- $parameters:
max_tokens: 512
temperature: 0.01
```

Other common model parameters are as follows:

* `temperature` is a floating point number between 0 and positive infinity that adjusts the smoothness of the sampled probability distribution. In the context of language models, it affects the selection process of the next word.
  * Low temperature (close to 0): The text generated by the model will be more conservative and predictable. At this time, the model tends to choose the words with the highest probability, and the generated text will be more fluent and regular, but may lack creativity or diversity.
  * High temperature: Increasing the `temperature` value will make the model more inclined to explore those words with lower probability, and the generated text will be more diverse and novel, but it may also be more discrete, difficult to understand, and even semantically jump.
* `continueOnLengthLimit`: This is used to determine whether AI will continue to be called automatically and continue to retrieve data after reaching the maximum token limit
* Note that this is not currently applicable when the return result is json. If you require that the returned json must be retrieved at once, increase `max_tokens`
* `maxRetry`: This parameter is also matched with `continueOnLengthLimit`, which is the maximum number of retries. If not set, the default is 7 times
* `timeout`: If the brain is big and the response is slow, and it takes more than 2 minutes to respond, then you need to adjust this timeout parameter, the unit is milliseconds
* `max_tokens`: This is the maximum token limit, the default is 2048, AI will output until max_tokens stops, which will avoid sometimes AI outputting infinitely and can't stop.
* `response_format`: Set the format of the returned result. Currently, only json (alias `json_object`) can be set for `type`.
  * Note: When `output` and `type:json` are set at the same time, the model will be forced to return json object instead of text.
  * If `response_format` is not set, you can set `forceJson:true` in the call parameters to achieve the same effect.

#### `$tool` tool directive

Use the `$tool` directive to use all registered tools.

##### `$AI` directive

`$AI` is an alias for `$tool:llm`, which directly calls the large model tool. By default, the result is appended to `prompt.messages` as the `assistant` role message. You can turn off the append by setting `shouldAppendResponse:false`.

```yaml
$AI:
  max_tokens: 512
  temperature: 0.7
  stream: true # Defaults to true, you can also set llmStream in the configuration, streaming response
  pushMessage: true # Defaults to true, indicating that the result returned by the large model tool is appended to prompt.messages.
  shouldAppendResponse: null  # Only valid when pushMessage is true, default is undefined.
                              # When undefined/null, when `matchedResponse` or `add_generation_prompt` or no lastMsg.content will be appended, otherwise the body of the last message will be replaced
                              # When true, force an assistant message to be appended. When false, force the body of the last message to be replaced.
  aborter: ?= new AbortController() # If not set, use the engine system's AbortController.
$tool:
  name: llm # Equal to $AI
  ...       # Other named parameters
```

#### `$abort` command

Manually stop the response of the large model, which will generate an abort exception.

```yaml
$AI
$abort
```

#### Pipeline command

`$pipe` will pass the result of the previous command to the pipeline.

Pass to the next instruction, supports the abbreviation `$|func`

```yaml
- toolId: $tool
# The return result of the previous function is passed to `func1|print`. If pipe has no parameters, it is passed to the next array element. If the next element itself is an object, it is merged.
- |
- $func1
- $pipe
- $print
```

```yaml
- llm: $tool
- $|func1
- $|print
```

#### `!fn` define function instruction

Use `!fn` tag to define function

```yaml
!fn |-
  function func1 ({arg1, arg2}) {
  }
# The function keyword can be omitted:
!fn |-
  func1 ({arg1, arg2}) {
  }
```

The function body is js syntax. In the definition function, `async require(moduleFilename)` can be used to load local esm js file in the format.

```yaml
!fn |-
  async myTool ({arg1, arg2}) {
  const tool = await require(__dirname + '/myTool.js')
  return tool.myTool({arg1, arg2})
  }
```

**Note**:

* `__dirname`: is the directory where the prompt script file is located.
* `__filename`: is the prompt script file path.
* In the function, you can use `this` to get all the methods of the current script's runtime.
* All custom functions must be referenced by `$`. For example, in the example above, `func1` is defined, so `$func1` must be used when calling:

  ```yaml
  $func1:
  arg1: 1
  arg2: 2
  ```

#### `!fn#` defines template function instructions

`!fn#` uses custom tags Define template functions, which are functions that can be used in the default JinJa template.

```yaml
---
content:
  a: 1
  b: 2
---
!fn# |-
  function toString(value) {
    return JSON.stringify(value)
  }
$format: "{{toString(content)}}"
```

#### `$exec` calls external script commands

Through the `$exec` command, you can interact with other agent scripts.

```yaml
$AI
$exec:
  # id: 'script id' # Only one of the script file name and id can be selected
  filename: json
  args: "?=LatestResult" # Pass the result of $AI to the json agent script through parameters.
```

#### `$if` command

`$if` function supports conditional judgment

```yaml
$set:
  a: 1
- $if: "a == 1" # Expression judgment
  then: # then function
    $echo: Ok
  else: # "else function"
    $echo: Not OK

!fn |-
  isOk(ok) {return ok}
- $if:
    $isOK: true # function judgment
  then: # then function
    $echo: Ok
  else: # "else function"
    $echo: Not OK
```

#### $format function

`$format` function uses Jinja2 template to format the string. The message formatting also uses Jinja2 template, which is also the template format supported by HuggingFace large model.

```yaml
$format: "{{description}}"
$format:
  template: "{{description}}"
  data:
    description: "hello world"
  templateFormat: "hf" # default is hf, currently supports hf, which is jinja2 used by huggingface; `golang` is also the template type used by `ollama` and `localai`; `fstring` is also `langchain` is in use.
```

#### $set/$get variable operation instructions

Support key path.

```yaml
$set:
  testVar.a: 124
  var2: !fn (key) { return key + 'hi' }
$get:
  - testVar.a
  - var2
```

### Event convention

Highly programmable and event-driven prompt generation system, which allows users to dynamically control and customize the process of generating text by defining event listeners, triggers and corresponding callback functions. From the examples given, we can see several key features and advantages:

* Event-driven architecture: By providing functions such as `$on`, `$once`, `$emit` and `$off`, the system supports an event-based programming model, allowing developers to flexibly intervene and extend the behavior of the model in response to different life cycle stages or specific conditions.
* Flexibility and scalability: Users can not only register named functions as callbacks, but also use anonymous functions or expressions, providing a variety of programming interfaces to adapt to different usage scenarios and complexity requirements. This enhances the flexibility and scalability of the script.
* Detailed event type design: From `beforeCall`, `afterCall` to `llm`, `llmStream` and other events for large model interactions, the system covers all aspects from function calls, result processing to model interactions, fully reflecting the in-depth understanding and support of common requirements in large model applications.
* Integration and interaction optimization: Through the `llmRequest` event, the system can intelligently manage large model calls, support customizing the way to obtain model responses through event mechanisms, and provide disable options to adapt to different strategies. In addition, the loading and saving of chat records are also open through events, which is convenient for integration into external systems or data management.
* Clear API design: The sample document shows clear API usage methods and parameter descriptions, which is convenient for developers to quickly get started and apply in depth, reflecting good design philosophy and user experience considerations.

`$on` function supports event monitoring, `$once` function supports event monitoring once, `$emit` function supports triggering events

`$off` function supports canceling event monitoring

#### `$on` and `$once` event monitoring functions

Parameters are as follows:

- event: event name
- callback: callback function or expression

Function as callback function:

```yaml
!fn |-
  onTest (event, arg1) { return {...arg1, event: event.type}}
$on:
  event: test
  callback: onTest # Named function monitoring, event monitoring can be canceled
$once: # Automatically cancel event monitoring after triggering once
  event: test
  callback: !fn |- # Anonymous function monitoring, event monitoring cannot be canceled
    (event, arg1) { return {...arg1, event: event.type}}
$emit: # Trigger event
  event: test
  args:
    a: 1
    b: 2
$off:
  event: test
  callback: onTest
```

The expression is used as a callback function. The parameters in the expression are as follows:

- event: event instance
- event.type: event name
- event.target: event source, that is, the current script runtime
- arg1: the first parameter value passed to the event listener function
- args: the remaining parameter value list passed to the event listener function, if any

These parameters are equivalent to the callback function: `(event, arg1, ...args) => void|any`

```yaml
$on:
  event: test
  callback: "?={...arg1, event: event.type}" # Unable to cancel event listening
```

#### `$emit` triggers event function

The parameters are as follows:

- event: event type, string, such as: `test`
- args: parameter value or parameter value list passed to the event listener function, if any

```yaml
$emit:
  event: test
  args: # an object parameter
    a: 1
    b: 2
$emit:
  event: test
  args: # indicates two object parameters
    - a: 1
    - b: 2
```

#### Script event type

- `beforeCall`: triggered before the function is called
- callback parameters: `(event, name, params, fn) => void|params`
- When the callback function returns a value, it means to modify the parameters.
- `afterCall`: triggered before the function returns the result
- callback parameters: `(event, name, params, result, fn) => void|result`
- When the callback function returns a value, it means to modify the return result.
- `llmParams`: Triggered before before the LLM is called and can be used to modify the parameters passed to the LLM.
  * Callback: `(event, params: {value: AIMessage[], options?: any, model?: string, count?: number}) => void|result<{value: AIMessage[], options?: any, model?: string, count?: number}>`
  * `value`: The messages to be sent to the LLM.
  * `options`: The options passed to the LLM.
  * `model`: The LLM name to be used.
  * `count`: the retry count if any.
- `llmBefore`: Triggered before before the LLM is called and can not modify the parameters, only used as notification.
  * Callback: `(event, params: any) => void`
- `llm`: the event is triggered before the large model returns the result, used to modify the large model return result.
- callback parameters: `(event, result: string) => void|result<string>`
- `llmStream`: triggered when the large model returns the result in streaming mode
- callback parameters: `(event, chunk: AIResult, content: string, retryCount: number) => void`
- chunk: current stream chunk content
- content: string content of all chunks currently obtained
- retryCount: number of retries for automatically calling llm when `max_token` is reached
- `llmRequest`: event is triggered when the large model result is needed, used to call the large model through the event and get the large model result. `[[RESPONSE]]` template will trigger this event
- callback parameters: `(event, messages: AIChatMessage[], options?) => void|result<string>`
- use the switch `disableLlmRequest: true` to disable this event.
- `ready`: triggered after the script interaction is ready, you can force the setting of whether it is in the ready state through the `$ready()` function.
- callback parameters: `(event, isReady: boolean) => void`
- `load-chats`: triggered when loading chat records.
- callback parameters: `(event, filename: string) => AIChatMessage[]|void`
- when the callback function returns a value, It means the loaded chat history.
- `save-chats`: Triggered when the chat history is saved.
- Callback parameters: `(event, messages: AIChatMessage[], filename?: string) => void`

**Note**:

* The `event` parameter in the event callback is the `Event` object, and `this` is the script runtime;
* When the event callback returns a value, it means modifying the parameter or result, otherwise it is not modified; the premise is that the event type supports modification;

## Refs

* [AutoGen](https://github.com/microsoft/autogen)
* [LMQL](https://lmql.ai/)
* [OpenInterpreter](https://github.com/OpenInterpreter/open-interpreter)
* [Outlines](https://github.com/outlines-dev/outlines)
* [MemGPT](https://github.com/cpacker/MemGPT/)
* [LangChain](https://github.com/langchain-ai/langchain)
