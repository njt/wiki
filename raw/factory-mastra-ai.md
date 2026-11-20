---
url: https://factory.mastra.ai/
date_fetched: 2026-11-20
---

# Get started

Mastra Factory is an open-source software factory. Connect your repository and Factory turns issues into plans, implementations, and reviewed pull requests, following the stages, skills, and approvals your team sets up.

Factory runs entirely on Mastra platform by default, but you can also run the server locally with platform resources or host everything on infrastructure you manage. Authentication, storage, and sandboxes can each be configured independently.

Help me set up Mastra Factory locally with Mastra platform resources and connect an existing GitHub repository. Follow the setup guide at `https://factory.mastra.ai/` for detailed instructions.

- Ask me what to name the Factory project as an open-ended question. Don't suggest placeholder names like `my-factory`. Also ask for the repository to use for agent work. Keep the Factory project in a separate directory from that repository.
- Run `npm create factory@latest <project-name>`and help me complete the installer prompts. Pass any answer you already have as a flag (`--org`,`--region`) instead of waiting for the prompt. Pause when sign-in to Mastra platform is required.
- Follow the guide to configure credential encryption once for this project. Preserve the generated settings in `.env`.
- Enter the Factory project directory, run `npm run dev`, and open the local URL printed by the server.
- Guide me through connecting the GitHub repository, model provider, and issue intake. Help me create a small test issue and confirm Factory lists it.

New to the product? Watch the overview video or read What's Factory?

## Create the Factory project

Create the Factory project in its own directory, separate from the repository you want agents to change. Both directories keep independent configuration and Git histories. You'll connect the repository in a later step. It can be one you already have, and it doesn't need to be new or empty.

- Mastra platform
- Self-host

This setup runs the Factory Server on your computer while Mastra platform provides authentication, PostgreSQL, and cloud sandboxes.

From a directory outside your target repository, run:

- npm
- pnpm
- Yarn
- Bun

```
npm create factory@latest my-factory
cd my-factory
```
```
pnpm create factory my-factory
cd my-factory
```
```
yarn create factory my-factory
cd my-factory
```
```
bunx create-factory my-factory
cd my-factory
```
Follow the prompts to sign in to Mastra platform and select an organization and region. The installer creates the server files and installs dependencies. It then provisions the necessary platform resources. It writes local configuration to `.env`.

This setup skips any platform provisioning and requires you to configure your own authentication, storage, and sandbox providers.

From a directory outside your target repository, run:

- npm
- pnpm
- Yarn
- Bun

```
npm create factory@latest my-factory -- --no-platform
cd my-factory
```
```
pnpm create factory my-factory --no-platform
cd my-factory
```
```
yarn create factory my-factory --no-platform
cd my-factory
```
```
bunx create-factory my-factory --no-platform
cd my-factory
```
The installer creates the Factory Server without signing in to Mastra platform or provisioning remote resources.

Configure auth, storage, and sandboxes before continuing with the next steps and running your Factory. For a production deployment, follow the self-hosting requirements.

## Configure credential encryption

The installer doesn't generate a credential encryption key. Before connecting a model provider, check whether `.env` contains `FACTORY_CREDENTIAL_ENCRYPTION_KEY`. If it's missing, generate a key once for this project:

`openssl rand -base64 32`Save the output as `FACTORY_CREDENTIAL_ENCRYPTION_KEY` in `.env` and keep a protected backup. Losing or replacing it can prevent Factory from reading stored credentials. See Credential encryption.

## Start the Factory Server

- npm
- pnpm
- Yarn
- Bun

`npm run dev``pnpm run dev``yarn dev``bun run dev`Open the local Mastra Factory URL printed by the server. Sign in through Mastra platform for the default setup, or through the auth provider you configured.

## Connect your codebase and model

- Create your first Factory in the UI.
- On **Choose your codebase**, select the repository agents should change. If it's missing, use**Manage GitHub connection**, grant the App access to that repository (permissions overview), then return to Factory.
- Skip Linear or follow Linear configuration.
- Connect a model provider and choose the Factory model. See Models for personal and organization access.

## Select your issue intake

- Open **Settings → Work Intake**.
- Under **GitHub issues**, enable**Sync GitHub issues**.
- Select your target repository and wait for the saved confirmation.
- Return to **Work**and check**Intake**.

The repository checkbox is an organization-wide selection. Every member reads the same sources.

Connecting a repository and selecting its intake are separate steps. If no issues arrive, check Troubleshooting before creating more issues.

## Create your first issue

Create a new GitHub issue in the selected repository using this example:

```
Title: Add contribution guidance to the README
Add a Contributing section explaining how to propose a change and open a pull request.
Acceptance criteria:
- Follow the repository's existing setup and contribution instructions.
- Do not invent commands the project does not support.
- Keep the change limited to documentation.
- Open a pull request for human review.
```
- Return to **Intake**and wait for synchronization. Match the issue's number, title, and GitHub link.
- For this walkthrough, keep the Work board's phases waiting for you, so nothing starts on its own.
- Select **Investigate**, then open the card to follow the agent's work in its session.
- Read the completed investigation and any linked GitHub output. A feature request can wait for your acceptance before planning.

After the new issue arrives and investigation finishes, read the result and identify the next action to complete this check. If the session fails or keeps waiting, use Troubleshooting.

## Create your first pull request

### Accept the work

From **Work**, open the session and read the assessment. Respond to any questions by providing the missing context, or narrow the task before continuing.

A non-bug task can show **needs your approval**. Select **Accept** to take the suggested acceptance action. The card menu exposes **Accept and plan** and **Close**. Choose **Accept and plan** for this walkthrough.

Acceptance authorizes the task to enter work. It's separate from approving the plan and from merging the eventual PR. See Work items.

### Review the plan

In the session, check that the plan:

- Changes the README and no unrelated application code.
- Uses setup and test commands already present in the repository.
- Explains how the agent will check the documentation change.
- Ends with a pull request for review.

Request a revision if the scope or commands are wrong. Approve the plan only after those details match the issue. With automatic plan approval off, a waiting plan needs your response before work continues.

### Inspect the implementation

Follow the session as the agent edits files and runs checks. Use the workspace's files and changes views to inspect the patch. An agent saying a check passed is useful context. Also inspect the tool output and the PR's checks.

For the README issue, confirm the proposed commands exist, links are valid, and the patch is limited to the intended documentation. If the run fails, read the failed tool result before restarting. See Sessions.

When the work is ready, the agent opens a pull request. Open its source link and verify the base branch, changed files, description, and connection to the original issue.

### Review the pull request

Find the PR's card on **Review**. It has a separate review session from the authoring Work item. Select **Review** if a review hasn't started, then read its findings alongside the patch and checks.

Request specific changes if needed. After another push, inspect the new diff and any follow-up review. Review items explains how updated, closed, and merged PRs affect the Review card.

### Test the result before merging

- Open the PR and read the rendered README.
- Check each acceptance criterion and inspect the diff for unrelated changes.
- Confirm the required repository checks pass, or understand and resolve their failures.
- Have the responsible reviewer approve and merge through your repository's normal process.
- Return to Factory and check the original Work item and the PR's Review item as two distinct cards.

A merged PR completes its Review card and moves the originating Work item to Done, unless that item has another PR open. Check the acceptance criteria before merging, because Factory doesn't reassess the task after the merge. A task may need more than one PR. Closing the linked issue also has its own completion or cancellation effect. See the source-event table.

## Next steps

- Read What's Factory? to learn how Factory works and understand its key concepts.
- Configure Linear or Slack to bring in more work.
- Follow Deployment to run the server beyond your local session.
