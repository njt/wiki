---
url: https://developers.openai.com/api/docs/guides/structured-outputs
date_fetched: 2026-07-05
backfilled: true
---

Ensure text responses from the model adhere to a JSON schema you define.

JSON is one of the most widely used formats in the world for applications to exchange data.

Structured Outputs is a feature that ensures the model will always generate responses that adhere to your supplied JSON Schema, so you don’t need to worry about the model omitting a required key, or hallucinating an invalid enum value.

Some benefits of Structured Outputs include:

Reliable type-safety: No need to validate or retry incorrectly formatted responses

Explicit refusals: Safety-based model refusals are now programmatically detectable

Simpler prompting: No need for strongly worded prompts to achieve consistent formatting

In addition to supporting JSON Schema in the REST API, the OpenAI SDKs for Python and JavaScript also make it easy to define object schemas using Pydantic and Zod respectively. Below, you can see how to extract information from unstructured text that conforms to a schema defined in code.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
from openai import OpenAI
from pydantic import BaseModel
client = OpenAI()
classCalendarEvent(BaseModel): name: str date: str participants: list[str]
response = client.responses.parse(
model="gpt-5.5",
input=[
{"role": "system", "content": "Extract the event information."},
{
"role": "user",
"content": "Alice and Bob are going to a science fair on Friday.",
},
],
text_format=CalendarEvent,
)
event = response.output_parsed

Supported models

Structured Outputs is available in our latest large language models, starting with GPT-4o. For new projects, start with gpt-5.5. Older models like gpt-4-turbo and earlier may use JSON mode instead.

When to use Structured Outputs via function calling vs via
response_format

When to use Structured Outputs via function calling vs via
text.format

Structured Outputs is available in two forms in the OpenAI API:

Function calling is useful when you are building an application that bridges the models and functionality of your application.

For example, you can give the model access to functions that query a database in order to build an AI assistant that can help users with their orders, or functions that can interact with the UI.

Conversely, Structured Outputs via response_format are more suitable when you want to indicate a structured schema for use when the model responds to the user, rather than when the model calls a tool.

For example, if you are building a math tutoring application, you might want the assistant to respond to your user using a specific JSON Schema so that you can generate a UI that displays different parts of the model’s output in distinct ways.

Put simply:

If you are connecting the model to tools, functions, data, etc. in your
system, then you should use function calling - If you want to structure the
model’s output when it responds to the user, then you should use a structured
response_format

If you are connecting the model to tools, functions, data, etc. in your
system, then you should use function calling - If you want to structure the
model’s output when it responds to the user, then you should use a structured
text.format

The remainder of this guide will focus on non-function calling use cases in
the Chat Completions API. To learn more about how to use Structured Outputs
with function calling, check out the

The remainder of this guide will focus on non-function calling use cases in
the Responses API. To learn more about how to use Structured Outputs with
function calling, check out the

Structured Outputs is the evolution of JSON mode. While both ensure valid JSON is produced, only Structured Outputs ensure schema adherence. Both Structured Outputs and JSON mode are supported in the Responses API, Chat Completions API, Assistants API, Fine-tuning API and Batch API.

We recommend always using Structured Outputs instead of JSON mode when possible.

However, Structured Outputs with response_format: {type: "json_schema", ...} is only supported with the gpt-4o-mini, gpt-4o-mini-2024-07-18, and gpt-4o-2024-08-06 model snapshots and later.

{ "steps": [ { "explanation": "Start with the equation 8x + 7 = -23.", "output": "8x + 7 = -23" }, { "explanation": "Subtract 7 from both sides to isolate the term with the variable.", "output": "8x = -23 - 7" }, { "explanation": "Simplify the right side of the equation.", "output": "8x = -30" }, { "explanation": "Divide both sides by 8 to solve for x.", "output": "x = -30 / 8" }, { "explanation": "Simplify the fraction.", "output": "x = -15 / 4" } ], "final_answer": "x = -15 / 4"}

Structured data extraction

Structured data extraction

You can define structured fields to extract from unstructured input data, such as research papers.

Extracting data from research papers using Structured Outputs

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
import OpenAI from"openai";
import { z } from"zod";
import { zodResponseFormat } from"openai/helpers/zod";
const openai = new OpenAI();
const ResearchPaperExtraction = z.object({
title: z.string(),
authors: z.array(z.string()),
abstract: z.string(),
keywords: z.array(z.string()),
});
const completion = await openai.chat.completions.parse({
model: "gpt-5.5",
messages: [
{ role: "system", content: "You are an expert at structured data extraction. You will be given unstructured text from a research paper and should convert it into the given structure." },
{ role: "user", content: "..." },
],
response_format: zodResponseFormat(ResearchPaperExtraction, "research_paper_extraction"),
});
const research_paper = completion.choices[0].message.parsed;

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
from pydantic import BaseModel
from openai import OpenAI
client = OpenAI()
classResearchPaperExtraction(BaseModel): title: str authors: list[str]
abstract: str keywords: list[str]
completion = client.chat.completions.parse(
model="gpt-5.5",
messages=[
{"role": "system", "content": "You are an expert at structured data extraction. You will be given unstructured text from a research paper and should convert it into the given structure."},
{"role": "user", "content": "..."}
],
response_format=ResearchPaperExtraction,
)
research_paper = completion.choices[0].message.parsed

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
curl https://api.openai.com/v1/chat/completions \
-H "Authorization: Bearer $OPENAI_API_KEY" \
-H "Content-Type: application/json" \
-d '{
"model": "gpt-5.5",
"messages": [
{
"role": "system",
"content": "You are an expert at structured data extraction. You will be given unstructured text from a research paper and should convert it into the given structure."
},
{
"role": "user",
"content": "..."
}
],
"response_format": {
"type": "json_schema",
"json_schema": {
"name": "research_paper_extraction",
"schema": {
"type": "object",
"properties": {
"title": { "type": "string" },
"authors": {
"type": "array",
"items": { "type": "string" }
},
"abstract": { "type": "string" },
"keywords": {
"type": "array",
"items": { "type": "string" }
}
},
"required": ["title", "authors", "abstract", "keywords"],
"additionalProperties": false
},
"strict": true
}
}
}'

Extracting data from research papers using Structured Outputs

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
import OpenAI from"openai";
import { zodTextFormat } from"openai/helpers/zod";
import { z } from"zod";
const openai = new OpenAI();
const ResearchPaperExtraction = z.object({
title: z.string(),
authors: z.array(z.string()),
abstract: z.string(),
keywords: z.array(z.string()),
});
const response = await openai.responses.parse({
model: "gpt-5.5",
input: [
{
role: "system",
content:
"You are an expert at structured data extraction. You will be given unstructured text from a research paper and should convert it into the given structure.",
},
{ role: "user", content: "..." },
],
text: {
format: zodTextFormat(ResearchPaperExtraction, "research_paper_extraction"),
},
});
const research_paper = response.output_parsed;

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
from openai import OpenAI
from pydantic import BaseModel
client = OpenAI()
classResearchPaperExtraction(BaseModel): title: str authors: list[str]
abstract: str keywords: list[str]
response = client.responses.parse(
model="gpt-5.5",
input=[
{
"role": "system",
"content": "You are an expert at structured data extraction. You will be given unstructured text from a research paper and should convert it into the given structure.",
},
{"role": "user", "content": "..."},
],
text_format=ResearchPaperExtraction,
)
research_paper = response.output_parsed

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
curl https://api.openai.com/v1/responses \
-H "Authorization: Bearer $OPENAI_API_KEY" \
-H "Content-Type: application/json" \
-d '{
"model": "gpt-5.5",
"input": [
{
"role": "system",
"content": "You are an expert at structured data extraction. You will be given unstructured text from a research paper and should convert it into the given structure."
},
{
"role": "user",
"content": "..."
}
],
"text": {
"format": {
"type": "json_schema",
"name": "research_paper_extraction",
"schema": {
"type": "object",
"properties": {
"title": { "type": "string" },
"authors": {
"type": "array",
"items": { "type": "string" }
},
"abstract": { "type": "string" },
"keywords": {
"type": "array",
"items": { "type": "string" }
}
},
"required": ["title", "authors", "abstract", "keywords"],
"additionalProperties": false
},
"strict": true
}
}
}'

Example response

{ "title": "Application of Quantum Algorithms in Interstellar Navigation: A New Frontier", "authors": ["Dr. Stella Voyager", "Dr. Nova Star", "Dr. Lyra Hunter"], "abstract": "This paper investigates the utilization of quantum algorithms to improve interstellar navigation systems. By leveraging quantum superposition and entanglement, our proposed navigation system can calculate optimal travel paths through space-time anomalies more efficiently than classical methods. Experimental simulations suggest a significant reduction in travel time and fuel consumption for interstellar missions.", "keywords": [ "Quantum algorithms", "interstellar navigation", "space-time anomalies", "quantum superposition", "quantum entanglement", "space travel" ]}

UI generation

UI Generation

You can generate valid HTML by representing it as recursive data structures with constraints, like enums.

How to use Structured Outputs with response_format

You can use Structured Outputs with the new SDK helper to parse the model’s output into your desired format, or you can specify the JSON schema directly.

Note: for fine tuned models, the first request you make with any schema
will have additional latency as our API processes the schema, but subsequent
requests with the same schema will not have additional latency. Other models
do not have this limitation.

SDK objects

Step 1: Define your object

First you must define an object or data structure to represent the JSON Schema that the model should be constrained to follow. See the examples at the top of this guide for reference.

While Structured Outputs supports much of JSON Schema, some features are unavailable either for performance or technical reasons. See here for more details.

To maximize the quality of model generations, we recommend the following:

Name keys clearly and intuitively

Create clear titles and descriptions for important keys in your structure

Create and use evals to determine the structure that works best for your use case

Step 2: Supply your object in the API call

You can use the parse method to automatically parse the JSON response into the object you defined.

Under the hood, the SDK takes care of supplying the JSON schema corresponding to your data structure, and then parsing the response as an object.

python

1
2
3
4
5
6
7
8
completion = client.chat.completions.parse(
model="gpt-5.5",
messages=[
{"role": "system", "content": "You are a helpful math tutor. Guide the user through the solution step by step."},
{"role": "user", "content": "how can I solve 8x + 7 = -23"},
],
response_format=MathResponse,
)

1
2
3
4
5
6
7
8
const completion = await openai.chat.completions.parse({
model: "gpt-5.5",
messages: [
{ role: "system", content: "You are a helpful math tutor. Guide the user through the solution step by step." },
{ role: "user", content: "how can I solve 8x + 7 = -23" },
],
response_format: zodResponseFormat(MathResponse, "math_response"),
});

Step 3: Handle edge cases

In some cases, the model might not generate a valid response that matches the provided JSON schema.

This can happen in the case of a refusal, if the model refuses to answer for safety reasons, or if for example you reach a max tokens limit and the response is incomplete.

First you must design the JSON Schema that the model should be constrained to follow. See the examples at the top of this guide for reference.

While Structured Outputs supports much of JSON Schema, some features are unavailable either for performance or technical reasons. See here for more details.

Tips for your JSON Schema

To maximize the quality of model generations, we recommend the following:

Name keys clearly and intuitively

Create clear titles and descriptions for important keys in your structure

Create and use evals to determine the structure that works best for your use case

Note: the first request you make with any schema will have additional latency as our API processes the schema, but subsequent requests with the same schema will not have additional latency.

Step 3: Handle edge cases

In some cases, the model might not generate a valid response that matches the provided JSON schema.

This can happen in the case of a refusal, if the model refuses to answer for safety reasons, or if for example you reach a max tokens limit and the response is incomplete.

Step 4: Use the generated structured data in a type-safe way

Typically, when using Structured Outputs you will have a type or class in the type-system of your programming language representing the JSON Schema as an object.

Once you have confirmed that you have received the JSON guaranteed to match the schema you requested, you can now safely parse it to the corresponding type.

For example:

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
from pydantic import BaseModel, ValidationError
from typing importList# Define types that match the JSON Schema using pydantic modelsclassStep(BaseModel): explanation: str output: strclassSolution(BaseModel): steps: List[Step]
final_answer: str...
try: # Parse and validate the response content solution = Solution.parse_raw(response.choices[0].message.content)
print(solution)
except ValidationError as e: # Handle validation errorsprint(e.json())

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
// Here we specify types in TypeScript that exactly match the JSON Schema we provided when calling the OpenAI API. Note that these _must_ be kept in sync.type Step = {
explanation: string;
output: string;
};
type Solution = {
steps: Step[];
final_answer: string;
};
...
// Now so long as the JSON Schema we created was exactly equivalent to our TypeScript types, this is type-safeconst solution = JSON.parse(response.choices[0].message.content) as Solution;

How to use Structured Outputs with text.format

Step 1: Define your schema

First you must design the JSON Schema that the model should be constrained to follow. See the examples at the top of this guide for reference.

While Structured Outputs supports much of JSON Schema, some features are unavailable either for performance or technical reasons. See here for more details.

Tips for your JSON Schema

To maximize the quality of model generations, we recommend the following:

Name keys clearly and intuitively

Create clear titles and descriptions for important keys in your structure

Create and use evals to determine the structure that works best for your use case

Note: the first request you make with any schema will have additional latency as our API processes the schema, but subsequent requests with the same schema will not have additional latency.

Step 3: Handle edge cases

In some cases, the model might not generate a valid response that matches the provided JSON schema.

This can happen in the case of a refusal, if the model refuses to answer for safety reasons, or if for example you reach a max tokens limit and the response is incomplete.

Step 4: Use the generated structured data in a type-safe way

Typically, when using Structured Outputs you will have a type or class in the type-system of your programming language representing the JSON Schema as an object.

Once you have confirmed that you have received the JSON guaranteed to match the schema you requested, you can now safely parse it to the corresponding type.

For example:

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
from pydantic import BaseModel, ValidationError
from typing importList# Define types that match the JSON Schema using pydantic modelsclassStep(BaseModel): explanation: str output: strclassSolution(BaseModel): steps: List[Step]
final_answer: str...
try: # Parse and validate the response content solution = Solution.parse_raw(response.choices[0].message.content)
print(solution)
except ValidationError as e: # Handle validation errorsprint(e.json())

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
// Here we specify types in TypeScript that exactly match the JSON Schema we provided when calling the OpenAI API. Note that these _must_ be kept in sync.type Step = {
explanation: string;
output: string;
};
type Solution = {
steps: Step[];
final_answer: string;
};
...
// Now so long as the JSON Schema we created was exactly equivalent to our TypeScript types, this is type-safeconst solution = JSON.parse(response.choices[0].message.content) as Solution;

Refusals with Structured Outputs

When using Structured Outputs with user-generated input, OpenAI models may occasionally refuse to fulfill the request for safety reasons. Since a refusal does not necessarily follow the schema you have supplied in response_format, the API response will include a new field called refusal to indicate that the model refused to fulfill the request.

When the refusal property appears in your output object, you might present the refusal in your UI, or include conditional logic in code that consumes the response to handle the case of a refused request.

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
classStep(BaseModel): explanation: str output: strclassMathReasoning(BaseModel): steps: list[Step]
final_answer: strcompletion = client.chat.completions.parse(
model="gpt-5.5",
messages=[
{"role": "system", "content": "You are a helpful math tutor. Guide the user through the solution step by step."},
{"role": "user", "content": "how can I solve 8x + 7 = -23"},
],
response_format=MathReasoning,
)
math_reasoning = completion.choices[0].message
# If the model refuses to respond, you will get a refusal messageif math_reasoning.refusal:
print(math_reasoning.refusal)
else:
print(math_reasoning.parsed)

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
const Step = z.object({
explanation: z.string(),
output: z.string(),
});
const MathReasoning = z.object({
steps: z.array(Step),
final_answer: z.string(),
});
const completion = await openai.chat.completions.parse({
model: "gpt-5.5",
messages: [
{ role: "system", content: "You are a helpful math tutor. Guide the user through the solution step by step." },
{ role: "user", content: "how can I solve 8x + 7 = -23" },
],
response_format: zodResponseFormat(MathReasoning, "math_reasoning"),
});
const math_reasoning = completion.choices[0].message;
// If the model refuses to respond, you will get a refusal messageif (math_reasoning.refusal) {
console.log(math_reasoning.refusal);
} else {
console.log(math_reasoning.parsed);
}

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
classStep(BaseModel): explanation: str output: strclassMathReasoning(BaseModel): steps: list[Step]
final_answer: strresponse = client.responses.parse(
model="gpt-5.5",
input=[
{"role": "system", "content": "You are a helpful math tutor. Guide the user through the solution step by step."},
{"role": "user", "content": "how can I solve 8x + 7 = -23"},
],
text_format=MathReasoning,
)
for output in response.output:
if output.type != "message":
continuefor item in output.content:
if item.type == "refusal":
# If the model refuses to respond, you will get a refusal messageprint(item.refusal)
continueifnot item.parsed:
raise Exception("Could not parse response")
print(item.parsed)

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
const Step = z.object({
explanation: z.string(),
output: z.string(),
});
const MathReasoning = z.object({
steps: z.array(Step),
final_answer: z.string(),
});
const response = await openai.responses.parse({
model: "gpt-5.5",
input: [
{ role: "system", content: "You are a helpful math tutor. Guide the user through the solution step by step." },
{ role: "user", content: "how can I solve 8x + 7 = -23" },
],
text: {
format: zodTextFormat(MathReasoning, "math_response"),
},
});
for (const output of response.output) {
if (output.type !== "message") {
continue;
}
for (const item of output.content) {
if (item.type == "refusal") {
// If the model refuses to respond, you will get a refusal messageconsole.log(item.refusal);
continue;
}
if (!item.parsed) {
thrownewError("Could not parse response");
}
console.log(item.parsed);
}
}

The API response from a refusal will look something like this:

If your application is using user-generated input, make sure your prompt includes instructions on how to handle situations where the input cannot result in a valid response.

The model will always try to adhere to the provided schema, which can result in hallucinations if the input is completely unrelated to the schema.

You could include language in your prompt to specify that you want to return empty parameters, or a specific sentence, if the model detects that the input is incompatible with the task.

Handling mistakes

Structured Outputs can still contain mistakes. If you see mistakes, try adjusting your instructions, providing examples in the system instructions, or splitting tasks into simpler subtasks. Refer to the prompt engineering guide for more guidance on how to tweak your inputs.

Avoid JSON schema divergence

To prevent your JSON Schema and corresponding types in your programming language from diverging, we strongly recommend using the native Pydantic/zod sdk support.

If you prefer to specify the JSON schema directly, you could add CI rules that flag when either the JSON schema or underlying data objects are edited, or add a CI step that auto-generates the JSON Schema from type definitions (or vice-versa).

Streaming

You can use streaming to process model responses or function call arguments as they are being generated, and parse them as structured data.

That way, you don’t have to wait for the entire response to complete before handling it.
This is particularly useful if you would like to display JSON fields one by one, or handle function call arguments as soon as they are available.

We recommend relying on the SDKs to handle streaming with Structured Outputs.

You can find an example of how to stream function call arguments without the SDK stream helper in the function calling guide.

Here is how you can stream a model response with the stream helper:

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
from typing importListfrom pydantic import BaseModel
from openai import OpenAI
classEntitiesModel(BaseModel): attributes: List[str]
colors: List[str]
animals: List[str]
client = OpenAI()
with client.beta.chat.completions.stream(
model="gpt-5.5",
messages=[
{"role": "system", "content": "Extract entities from the input text"},
{
"role": "user",
"content": "The quick brown fox jumps over the lazy dog with piercing blue eyes",
},
],
response_format=EntitiesModel,
) as stream:
for event in stream:
if event.type == "content.delta":
if event.parsed isnotNone: # Print the parsed data as JSONprint("content.delta parsed:", event.parsed)
elif event.type == "content.done":
print("content.done")
elif event.type == "error":
print("Error in stream:", event.error)
final_completion = stream.get_final_completion()
print("Final completion:", final_completion)

Root objects must not be anyOf and must be an object

Note that the root level object of a schema must be an object, and not use anyOf. A pattern that appears in Zod (as one example) is using a discriminated union, which produces an anyOf at the top level. So code such as the following won’t work:

To use Structured Outputs, all fields or function parameters must be specified as required.

json

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
{
"name": "get_weather",
"description": "Fetches the weather in the given location",
"strict": true,
"parameters": {
"type": "object",
"properties": {
"location": {
"type": "string",
"description": "The location to get the weather for" },
"unit": {
"type": "string",
"description": "The unit to return the temperature in",
"enum": ["F", "C"]
}
},
"additionalProperties": false,
"required": ["location", "unit"]
}
}

Although all fields must be required (and the model will return a value for each parameter), it is possible to emulate an optional parameter by using a union type with null.

json

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
{
"name": "get_weather",
"description": "Fetches the weather in the given location",
"strict": true,
"parameters": {
"type": "object",
"properties": {
"location": {
"type": "string",
"description": "The location to get the weather for" },
"unit": {
"type": ["string", "null"],
"description": "The unit to return the temperature in",
"enum": ["F", "C"]
}
},
"additionalProperties": false,
"required": [
"location", "unit" ]
}
}

Objects have limitations on nesting depth and size

A schema may have up to 5000 object properties total, with up to 10 levels of nesting.

Limitations on total string size

In a schema, total string length of all property names, definition names, enum values, and const values cannot exceed 120,000 characters.

Limitations on enum size

A schema may have up to 1000 enum values across all enum properties.

For a single enum property with string values, the total string length of all enum values cannot exceed 15,000 characters when there are more than 250 enum values.

additionalProperties: false must always be set in objects

additionalProperties controls whether it is allowable for an object to contain additional keys / values that were not defined in the JSON Schema.

Structured Outputs only supports generating specified keys / values, so we require developers to set additionalProperties: false to opt into Structured Outputs.

json

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
{
"name": "get_weather",
"description": "Fetches the weather in the given location",
"strict": true,
"schema": {
"type": "object",
"properties": {
"location": {
"type": "string",
"description": "The location to get the weather for" },
"unit": {
"type": "string",
"description": "The unit to return the temperature in",
"enum": ["F", "C"]
}
},
"additionalProperties": false,
"required": [
"location", "unit" ]
}
}

Key ordering

When using Structured Outputs, outputs will be produced in the same order as the ordering of keys in the schema.

JSON mode is a more basic version of the Structured Outputs feature. While
JSON mode ensures that model output is valid JSON, Structured Outputs reliably
matches the model’s output to the schema you specify. We recommend you use
Structured Outputs if it is supported for your use case.

When JSON mode is turned on, the model’s output is ensured to be valid JSON, except for in some edge cases that you should detect and handle appropriately.

To turn on JSON mode with the Chat Completions, set the response_format to { "type": "json_object" }. If you are using function calling, JSON mode is always turned on.

To turn on JSON mode with the Responses API you can set the text.format to { "type": "json_object" }. If you are using function calling, JSON mode is always turned on.

Important notes:

When using JSON mode, you must always instruct the model to produce JSON via some message in the conversation, for example via your system message. If you don’t include an explicit instruction to generate JSON, the model may generate an unending stream of whitespace and the request may run continually until it reaches the token limit. To help ensure you don’t forget, the API will throw an error if the string “JSON” does not appear somewhere in the context.

JSON mode will not guarantee the output matches any specific schema, only that it is valid and parses without errors. You should use Structured Outputs to ensure it matches your schema, or if that is not possible, you should use a validation library and potentially retries to ensure that the output matches your desired schema.

Your application must detect and handle the edge cases that can result in the model output not being a complete JSON object (see below)

Handling edge cases

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
const we_did_not_specify_stop_tokens = true;
try {
const response = await openai.chat.completions.create({
model: "gpt-5.5",
messages: [
{
role: "system",
content: "You are a helpful assistant designed to output JSON.",
},
{ role: "user", content: "Who won the world series in 2020? Please respond in the format {winner: ...}" },
],
store: true,
response_format: { type: "json_object" },
});
// Check if the conversation was too long for the context window, resulting in incomplete JSONif (response.choices[0].finish_reason === "length") {
// your code should handle this error case }
// Check if the OpenAI safety system refused the request and generated a refusal insteadif (response.choices[0].message.refusal) {
// your code should handle this error case// In this case, the .content field will contain the explanation (if any) that the model generated for why it is refusingconsole.log(response.choices[0].message.refusal)
}
// Check if the model's output included restricted content, so the generation of JSON was halted and may be partialif (response.choices[0].finish_reason === "content_filter") {
// your code should handle this error case }
if (response.choices[0].finish_reason === "stop") {
// In this case the model has either successfully finished generating the JSON object according to your schema, or the model generated one of the tokens you provided as a "stop token"if (we_did_not_specify_stop_tokens) {
// If you didn't specify any stop tokens, then the generation is complete and the content key will contain the serialized JSON object// This will parse successfully and should now contain {"winner": "Los Angeles Dodgers"}console.log(JSON.parse(response.choices[0].message.content))
} else {
// Check if the response.choices[0].message.content ends with one of your stop tokens and handle appropriately }
}
} catch (e) {
// Your code should handle errors here, for example a network error calling the APIconsole.error(e)
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
we_did_not_specify_stop_tokens = Truetry:
response = client.chat.completions.create(
model="gpt-5.5",
messages=[
{"role": "system", "content": "You are a helpful assistant designed to output JSON."},
{"role": "user", "content": "Who won the world series in 2020? Please respond in the format {winner: ...}"}
],
response_format={"type": "json_object"}
)
# Check if the conversation was too long for the context window, resulting in incomplete JSONif response.choices[0].finish_reason == "length":
# your code should handle this error casepass# Check if the OpenAI safety system refused the request and generated a refusal insteadif response.choices[0].message.refusal:
# your code should handle this error case# In this case, the .content field will contain the explanation (if any) that the model generated for why it is refusingprint(response.choices[0].message.refusal)
# Check if the model's output included restricted content, so the generation of JSON was halted and may be partialif response.choices[0].finish_reason == "content_filter":
# your code should handle this error casepassif response.choices[0].finish_reason == "stop":
# In this case the model has either successfully finished generating the JSON object according to your schema, or the model generated one of the tokens you provided as a "stop token"if we_did_not_specify_stop_tokens:
# If you didn't specify any stop tokens, then the generation is complete and the content key will contain the serialized JSON object# This will parse successfully and should now contain "{"winner": "Los Angeles Dodgers"}"print(response.choices[0].message.content)
else:
# Check if the response.choices[0].message.content ends with one of your stop tokens and handle appropriatelypassexcept Exception as e:
# Your code should handle errors here, for example a network error calling the APIprint(e)

python

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
const we_did_not_specify_stop_tokens = true;
try {
const response = await openai.responses.create({
model: "gpt-5.5",
input: [
{
role: "system",
content: "You are a helpful assistant designed to output JSON.",
},
{ role: "user", content: "Who won the world series in 2020? Please respond in the format {winner: ...}" },
],
text: { format: { type: "json_object" } },
});
const message = response.output.find((item) => item.type === "message");
const messageContent = message?.content[0];
// Check if the conversation was too long for the context window, resulting in incomplete JSONif (response.status === "incomplete" && response.incomplete_details.reason === "max_output_tokens") {
// your code should handle this error case }
// Check if the OpenAI safety system refused the request and generated a refusal insteadif (messageContent?.type === "refusal") {
// your code should handle this error case// In this case, the .content field will contain the explanation (if any) that the model generated for why it is refusingconsole.log(messageContent.refusal)
}
// Check if the model's output included restricted content, so the generation of JSON was halted and may be partialif (response.status === "incomplete" && response.incomplete_details.reason === "content_filter") {
// your code should handle this error case }
if (response.status === "completed") {
// In this case the model has either successfully finished generating the JSON object according to your schema, or the model generated one of the tokens you provided as a "stop token"if (we_did_not_specify_stop_tokens) {
// If you didn't specify any stop tokens, then the generation is complete and the content key will contain the serialized JSON object// This will parse successfully and should now contain {"winner": "Los Angeles Dodgers"}console.log(JSON.parse(response.output_text))
} else {
// Check if the response.output_text ends with one of your stop tokens and handle appropriately }
}
} catch (e) {
// Your code should handle errors here, for example a network error calling the APIconsole.error(e)
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
we_did_not_specify_stop_tokens = Truetry:
response = client.responses.create(
model="gpt-5.5",
input=[
{"role": "system", "content": "You are a helpful assistant designed to output JSON."},
{"role": "user", "content": "Who won the world series in 2020? Please respond in the format {winner: ...}"}
],
text={"format": {"type": "json_object"}}
)
message = next((item for item in response.output if item.type == "message"), None)
message_content = message.content[0] if message and message.content elseNone# Check if the conversation was too long for the context window, resulting in incomplete JSONif response.status == "incomplete"and response.incomplete_details.reason == "max_output_tokens":
# your code should handle this error casepass# Check if the OpenAI safety system refused the request and generated a refusal insteadif message_content and message_content.type == "refusal":
# your code should handle this error case# In this case, the .content field will contain the explanation (if any) that the model generated for why it is refusingprint(message_content.refusal)
# Check if the model's output included restricted content, so the generation of JSON was halted and may be partialif response.status == "incomplete"and response.incomplete_details.reason == "content_filter":
# your code should handle this error casepassif response.status == "completed":
# In this case the model has either successfully finished generating the JSON object according to your schema, or the model generated one of the tokens you provided as a "stop token"if we_did_not_specify_stop_tokens:
# If you didn't specify any stop tokens, then the generation is complete and the content key will contain the serialized JSON object# This will parse successfully and should now contain "{"winner": "Los Angeles Dodgers"}"print(response.output_text)
else:
# Check if the response.output_text ends with one of your stop tokens and handle appropriatelypassexcept Exception as e:
# Your code should handle errors here, for example a network error calling the APIprint(e)

Resources

To learn more about Structured Outputs, we recommend browsing the following resources:
