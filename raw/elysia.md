---
url: https://github.com/weaviate/elysia
date_fetched: 2026-07-05
backfilled: true
---

⚠️ Elysia is in beta!If you encounter any issues, please open an issue on GitHub.


Elysia is an agentic platform designed to use tools in a decision tree. A decision agent decides which tools to use dynamically based on its environment and context. You can use custom tools or use the pre-built tools designed to retrieve your data in a Weaviate cluster.

💡 Don't forget to check out the Github Repository for the Frontend!


Installation is as simple as:

`pip install elysia-ai`Run the app via

`elysia start`Then head to `localhost:8000` in a browser, navigate to the settings page, add your required API keys, Weaviate cloud cluster details and specify your models. Optionally use `--port` to specify which port Elysia will be run on.

Alternatively, we have created a demo version of Elysia (rate-limited, fixed datasets) to experiment with. Find it at: https://elysia.weaviate.io/

To use Elysia, you need to either set up your models and API keys in your `.env` file, or specify them in the config. See the setup page to get started.

Elysia can be used very simply:

```
from elysia import tool, Tree
tree = Tree()
@tool(tree=tree)
async def add(x: int, y: int) -> int:
    return x + y
tree("What is the sum of 9009 and 6006?")
```
Elysia is pre-configured to be capable of connecting to and interacting with your Weaviate clusters!

```
import elysia
tree = elysia.Tree()
response, objects = tree(
    "What are the 10 most expensive items in the Ecommerce collection?",
    collection_names = ["Ecommerce"]
)
```
This will use the built-in open source *query* tool or *aggregate* tool to interact with your Weaviate collections. To get started connecting to Weaviate, see the setting up page in the docs.

Elysia requires Python 3.12:

Optionally create a virtual environment via

```
python3.12 -m venv .venv
source .venv/bin/activate
```
Then simply run

`pip install elysia-ai`to install straight away!

To get the latest development version, you can clone the github repo by running

`git clone https://github.com/weaviate/elysia`move to the working directory via

`cd elysia`Create a virtual environment with Python (version 3.10 - 3.12)

```
python3.12 -m venv .venv
source .venv/bin/activate
```
and then install Elysia via pip

`pip install -e .`Done! You can now use the Elysia python package

To use Elysia with Weaviate, i.e. for agentic searching and retrieval, you need to either have a *locally running* instance of Weaviate, or access to a *Weaviate cloud cluster* via an api key and URL. This can be specific in the app directly (see above image), or by creating a `.env` file with

```
WCD_URL=...
WCD_API_KEY=...
WEAVIATE_IS_LOCAL=... # True or False
```
Elysia will automatically detect these when running locally, and this will be the default Weaviate cluster for all users logging into the Elysia app. But these can be configured on a user-by-user basis through the config.

Whichever vectoriser you use for your Weaviate collection you will need to specify your corresponding API key, e.g.

```
OPENAI_API_KEY=...
```
These will automatically be added to the headers for the Weaviate client.

Same for whichever model you choose for the LLM in Elysia, so if you are using GPT-4o, for example, specify an `OPENAI_API_KEY`.

Elysia's recommended config is to use OpenRouter to give easy access to a variety of models. So this requires

```
OPENROUTER_API_KEY=...
```
Elysia is architectured as a modern web application with a full-featured frontend for a responsive, real-time interface and a FastAPI backend serving both the web interface and API. The core logic is written in pure Python – what we call "blood, sweat, and tears" custom logic – with DSPy handling LLM interactions.

Unlike simple agentic platforms which have access to all possible tools at runtime, Elysia has a pre-defined web of possible nodes, each with a corresponding action. Each node in the tree is orchestrated by a decision agent with global context awareness about its environment and its available options. The decision agent evaluates its environment, available actions, past actions and future actions to strategize the best tool to use.

Read more about how we built Elysia in this blog.

**Weaviate** is proud to offer this open source project for the community. While we strive to address issues as fast as we can, please understand that it may not be maintained with the same rigor as production software. We welcome and encourage community contributions to help keep it running smoothly. Your support in fixing open issues quickly is greatly appreciated.

See the full contributor guidelines to see how you can get started contributing to Elysia!

**How do I use Elysia with my own data?**

You can connect to your own Weaviate cloud cluster, which will automatically identify any collections that exist in the cluster.

Collections require being *preprocessed* for Elysia. In the app, you just click the 'analyze' button in the Data tab. In Python you can do:

```
from elysia.preprocessing.collection import preprocess
preprocess(collection_names=["YourCollectionName"])
```
**Can I run Elysia completely locally? (Locally running Weaviate, local models)**

Yes!

You can connect to a locally running Weaviate instance in Docker, and connect to Ollama for locally running language models. See the setup page to get started.

**Help! My local model isn't working with Elysia. It's timing out or there are errors.**

Elysia works with quite long context, so some smaller models will struggle with this - it will either take a very long time to complete or the model will error to output the correct structured response.

For a complete guide and troubleshooting, see this page of the documentation.

**How do I clear all my Elysia data?**

Everything Elysia doesn't store locally will be a collection in your Weaviate cluster. You can delete any collections that start with `ELYSIA_` to reset all your Elysia data.

For example, in Python:

```
from elysia.util.client import ClientManager()
with ClientManager().connect_to_client() as client:
    for collection_name in client.collections.list_all():
        if collection_name.startswith("ELYSIA_"):
            client.collections.delete(collection_name)
```
**Can I contribute to Elysia?**

Elysia is **fully open source**, so yes of course you can! Clone and create a new branch of Elysia via

```
git clone https://github.com/weaviate/elysia
git checkout -b <branch_name>
```
Make your changes, push them to your branch, go to GitHub and submit a pull request.

**Where is the best place I can start contributing?**

There are no 'huge' new features we are planning for Elysia (for the moment). You could start with creating a new tool, or multiple new tools to create a custom workflow for something specific. Look for pain points you experience from your user journey and find what exactly is causing these. Then try to fix them or create an alternative way of doing things!
