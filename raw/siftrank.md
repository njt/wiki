---
url: https://github.com/noperator/siftrank
date_fetched: 2026-07-05
backfilled: true
---

Use LLMs for document ranking.

Got a bunch of data? Want to throw it at an LLM to find the most "interesting" stuff? If you simply YOLO your data into a ChatGPT session, you'll run into problems:

- Nondeterminism: Doesn't always respond with the same result
- Limited context: Can't pass in all the data at once, need to break it up
- Output contraints: Sometimes doesn't return all the data you asked it to review
- Scoring subjectivity: Struggles to assign a consistent numeric score to an individual item

`siftrank` is an implementation of the **Sift**Rank document ranking algorithm that uses LLMs to efficiently find the items in any dataset that are most relevant to a given prompt:

- **S**tochastic: Randomly samples the dataset into small batches.
- **I**nflective: Looks for a natural inflection point in the scores that distinguishes particularly relevant items from the rest.
- **F**ixed: Caps the maximum number of LLM calls so the computational complexity remains linear in the worst case.
- **T**rial: Repeatedly compares batched items until the relevance scores stabilize.

Use LLMs to rank anything. No fine-tuning. No domain-specific models. Just an off-the-shelf model and your ranking prompt. Typically runs in seconds and costs pennies.

```
go install github.com/noperator/siftrank/cmd/siftrank@latest
```
Either set an `OPENAI_API_KEY` environment variable, or create a named configuration profile in  `~/.config/siftrank/config.yaml` or `./config.yaml` containing the API key. The `default` profile auto-loads when `-P` is not specified, and CLI flags always take precedence over profile values. Use `*_cmd` to retrieve secrets from a password manager or other command rather than storing them in plaintext. See `config-example.yaml` for all available options.

```
default: nano
profiles:
  nano:
    api_key_cmd: op read op://myvault/openai-api-key/credential
    model: gpt-5-nano-2025-08-07
    effort: minimal
```
```
siftrank -h
Options:
      --config-file string   path to config file (overrides discovery)
  -f, --file string          input file (required)
  -m, --model string         OpenAI model name (default "gpt-4o-mini")
  -o, --output string        JSON output file
  -P, --profile string       use a named profile from the config file
  -p, --prompt string        initial prompt (prefix with @ to use a file)
  -r, --relevance            post-process each item by providing relevance justification (skips round 1)
Visualization:
      --no-minimap   disable minimap panel in watch mode
      --watch        enable live terminal visualization (logs suppressed unless --log is specified)
Debug:
  -d, --debug          enable debug logging
      --dry-run        log API calls without making them
      --log string     write logs to file instead of stderr
      --trace string   trace file path for streaming trial execution state (JSON Lines format)
Advanced:
  -u, --base-url string         OpenAI API base URL (for compatible APIs like vLLM)
  -b, --batch-size int          number of items per batch (default 10)
  -c, --concurrency int         max concurrent LLM calls across all trials (default 50)
  -e, --effort string           reasoning effort level: none, minimal, low, medium, high
      --elbow-method string     elbow detection method: curvature (default), perpendicular (default "curvature")
      --elbow-tolerance float   elbow position tolerance (0.05 = 5%) (default 0.05)
      --encoding string         tokenizer encoding (default "o200k_base")
      --json                    force JSON parsing regardless of file extension
      --max-trials int          maximum number of ranking trials (default 50)
      --min-trials int          minimum trials before checking convergence (default 5)
      --no-converge             disable early stopping based on convergence
      --ratio float             refinement ratio (0.0-1.0, e.g. 0.5 = top 50%) (default 0.5)
      --stable-trials int       stable trials required for convergence (default 5)
      --template string         template for each object (prefix with @ to use a file) (default "{{.Data}}")
      --tokens int              max tokens per batch (default 128000)
```
Compares 100 sentences in 7 seconds.

```
siftrank \
    -f testdata/sentences.txt \
    -p 'Rank each of these items according to their relevancy to the concept of "time".' |
    jq -r '.[:10] | map(.value)[]' |
    nl
   1  The train arrived exactly on time.
   2  The old clock chimed twelve times.
   3  The clock ticked steadily on the wall.
   4  The bell rang, signaling the end of class.
   5  The rooster crowed at the break of dawn.
   6  She climbed to the top of the hill to watch the sunset.
   7  He watched as the leaves fell one by one.
   8  The stars twinkled brightly in the clear night sky.
   9  He spotted a shooting star while stargazing.
  10  She opened the curtains to let in the morning light.
```
## Advanced usage

If the input file is a JSON document, it will be read as an array of objects and each object will be used for ranking.

For instance, two objects would be loaded and ranked from this document:

```
[
  {
    "path": "/foo",
    "code": "bar"
  },
  {
    "path": "/baz",
    "code": "nope"
  }
]
```
It is possible to include each element from the input file in a template using the Go template syntax via the `--template "template string"` (or `--template @file.tpl`) argument.

For text input files, each line can be referenced in the template with the `Data` variable:

```
Anything you want with {{ .Data }}
```
For JSON input files, each object in the array can be referenced directly. For instance, elements of the previous JSON example can be referenced in the template code like so:

```
# {{ .path }}
{{ .code }}
```
Note in the following example that the resulting `value` key contains the actual value being presented for ranking (as described by the template), while the `object` key contains the entire original object from the input file for easy reference.

```
# Create some test JSON data.
seq 9 |
    paste -d @ - - - |
    parallel 'echo {} | tr @ "\n" | jo -a | jo nums=:/dev/stdin' |
    jo -a |
    tee input.json
[{"nums":[1,2,3]},{"nums":[4,5,6]},{"nums":[7,8,9]}]
# Use template to extract the first element of the nums array in each input object.
siftrank \
	-f input.json \
	-p 'Which is biggest?' \
	--template '{{ index .nums 0 }}' \
	--max-trials 1 |
	jq -c '.[]'
{"key":"eQJpm-Qs","value":"7","object":{"nums":[7,8,9]},"score":0,"exposure":1,"rank":1}
{"key":"SyJ3d9Td","value":"4","object":{"nums":[4,5,6]},"score":2,"exposure":1,"rank":2}
{"key":"a4ayc_80","value":"1","object":{"nums":[1,2,3]},"score":3,"exposure":1,"rank":3}
```
I released the prototype of this tool, Raink, while at Bishop Fox. See the original presentation, blog post and CLI tool.

- O(N) the Money: Scaling Vulnerability Research with LLMs
- Using LLMs to solve security problems
- Hard problems that reduce to document ranking
- Commentary: Critical Thinking - Bug Bounty Podcast
- Discussion: Hacker News
- Large Language Models are Effective Text Rankers with Pairwise Ranking Prompting

- add python bindings?
-  allow specifying an input *directory*(where each file is distinct object)
- clarify when prompt included in token estimate
- factor LLM calls out into a separate package
- run openai batch mode
- report cost + token usage
- add more examples, use cases
- account for reasoning tokens separately

## Completed

- add visualization
- support reasoning effort
- add blog link
- add parameter for refinement ratio
-  add ~~boolean~~refinement ratio flag
- alert if the incoming context window is super large
- automatically calculate optimal batch size?
- explore "tournament" sort vs complete exposure each time
- make sure that each randomized run is evenly split into groups so each one gets included/exposed
- parallelize openai calls for each run
- remove token limit threshold? potentially confusing/unnecessary
- save time by using shorter hash ids
- separate package and cli tool
- some batches near the end of a run (9?) are small for some reason
- support non-OpenAI models

This project is licensed under the MIT License.
