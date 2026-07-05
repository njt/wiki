---
url: https://www.inkandswitch.com/upwelling/
date_fetched: 2026-07-05
backfilled: true
---

## Problems with existing collaboration tools

Professional non-fiction writing often requires several people – authors, editors, and reviewers – to work together in order to produce a finished document. For example, several journalists and editors may contribute to a news article, a group of scientists may collaborate on a research paper, and in business or government the members of a team may jointly write a proposal, report, or documentation. In all of these cases, it is important for the contributors to ensure that the final document is of a high quality, correct, and consistent in both content and style.

Most writers today use two categories of tools: traditional file-based desktop software, such as Microsoft Word, and real-time collaborative cloud software, such as Google Docs. In order to better understand professional workflows for these tools, we interviewed writers and editors working in a wide variety of environments. Several common themes emerged from these interviews:

- The goal is to bring together ideas from multiple collaborators into a single document, such as a news article or a chapter of a book.
- Writers often prefer to initially ideate in private and share the result with their collaborators later, when they are ready.
- Important documents often have a lead author or editor responsible for reviewing all changes before publication. Sometimes there are multiple lead authors. Depending on the setting this might be a formal review process or an informal arrangement.
- Common user interfaces for reviewing changes suffer from readability and usability problems.

### The “fishbowl effect” of real-time collaboration

Real-time collaboration allows each user to see their collaborators’ cursor positions and edits as they type, keystroke by keystroke. This has become a standard feature of applications including Google Docs, Figma, the web version of Microsoft Office 365, and Apple Keynote. The widespread adoption of real-time collaboration speaks to the fact that it is a valuable feature. However, we found that our interviewees also had significant reservations about real-time collaboration.

Several writers we talked to wanted a tool that would allow them to work in private, with no other collaborators reading their work in progress. Intermediate drafts aren’t always suitable to share, even with collaborators, and feedback on those drafts can be unwelcome or even embarrassing. In addition, some writers are troubled by the idea that their senior co-workers and management may be monitoring them – an unintended negative side effect of real-time collaboration.

“Writers don’t want first drafts visible to the editor.” — Journalist


Real-time collaboration becomes even more intrusive when others not only watch what a writer is typing but even start editing or commenting on the writer’s work before it is complete. Some of our interviewees reported asking collaborators to close the document and not make edits or comments while they were working. Others reported copying and pasting the entire document into a new file, working there in private, and then pasting the edited text back into the original editor window when finished.

“If you don’t want the editor or writer to see what you’re doing until you’re done, sometimes you make a copy of the doc and work in the copy, then paste it back (otherwise you have 17 canonical documents and people are editing the wrong thing).” — Newspaper Editor


Other writers reported putting their device into offline (airplane) mode to prevent their edits being shared while they worked.

We use the term “fishbowl effect” to refer to this downside of real-time collaboration: a writer’s feeling of being watched or interrupted in their workflow by collaborators. Real-time collaboration can be beneficial for lightweight writing tasks (like meeting notes) or when collaborators are working together live (e.g., during a video call). However, our interviews highlighted that many writers prefer working in a private space during phases of individual, creative work.

### Versioning in file-based collaboration

Traditional desktop word processors, which edit files on the local file system, do not suffer from the fishbowl effect. They work well for people writing alone, but have the downside of requiring manual version management when collaborating with others. Writers must send a copy of their file to each collaborator (e.g., by email), and when several collaborators independently edit their copies, somebody must manually compare and merge the resulting file versions.

“In the magazine article we wrote, it got to the point where we eventually created our own version history. After a round of comments, we duplicate the document, save v1, and create v2.” — Academic Researcher


Teams often use file-naming conventions to keep track of these versions. In fact, file names can serve as a rudimentary system for version control. A document named `Temporal Paper.docx` might evolve into `Temporal Paper - Final - PLOS Revisions v3.docx` over time. Some teams may use version control systems, such as Git, but such systems are designed for software development and are poorly adapted for working with prose: by default they use a line-based comparison, and they cannot merge file formats that are not plain text. Although file comparison tools for Microsoft Word documents and other binary file formats exist, merging several collaborators’ edits often remains a tedious and error-prone process of manually copying and pasting.

Another problem with file-based collaboration is that unless collaborators remember to explicitly inform each other what they are doing, writers have no awareness of in-progress changes that other writers are making. If two writers edit the same section of a document, the result may be many conflicting edits that could have been avoided through coordination.

“My preferred workflow for Word would be to pass it around one by one—a linear order, rather than having two people working simultaneously.” — Educational Writer


To avoid manually merging documents, some writers use a sequential workflow, sending a file to their collaborator and waiting for the document to be returned to them before making any further changes. This workflow produces a linear version history and eliminates the need to merge versions of the document. However, it also slows down the collaborative writing process, preventing writers from making any changes to a document until they receive an updated version from someone else.

### The need for editorial review

Another theme that emerged from our interviews is the need for collaborators to review each other’s contributions to a document. However, review processes differ significantly from team to team. Sometimes several co-authors mutually review each other’s work; sometimes a document has a single lead author who takes responsibility for the overall cohesion and style.

In journalism, very strict review processes are sometimes used: once an editor has read a draft article, writers are no longer allowed to modify it and any further updates require explicit review and sign-off from the editor. Some publications, such as *The New York Times*, have even developed their own in-house collaboration software, customized to their editorial workflow.

Even though review processes vary between teams, the goal is the same: giving all collaborators the confidence that the final document is accurate, consistent, and complete. Software can support this goal by making it easy to review every change that has been made to the document.

“There can be hundreds of comments, and you can’t search them. If something changed in the doc, we bold it.” — Editor


### Visualizing changes to a document

Reviewers benefit from seeing how a document has changed without having to re-read the entire document. Editors have two tools that are typically used for visually highlighting modifications: Microsoft Word’s *Track Changes* (or *Suggestion Mode* in Google Docs), and comparison tools that visualize the differences between two specified versions of a document.

Both change tracking and version comparison typically render deleted text in a strikethrough font and inserted text in color (with a different color used for each collaborator). With Track Changes, each edit is additionally highlighted in a sidebar alongside buttons for accepting and rejecting the edit. Comments can also be attached to spans of text to facilitate discussion among the co-authors; those, too, appear in the sidebar.

Several of our interviewees found this user interface problematic. A common complaint was that a heavily edited text becomes difficult to read, because the mixture of struck-out text fragments and multiple colors lead to a visually overwhelming result that obscures the final text.

“I hate Track Changes! I’m dyslexic. I find reading Track Changes to be nearly impossible. If someone has heavily edited, I have to accept all of them and read it.” — Fiction Writer


Another problem is that the person making the changes needs to remember to enable change tracking or suggestion mode before making any edits. Moreover, in the Google Docs implementation of Suggestion Mode, an edit becomes indistinguishable from other text once it is accepted. If one reviewer accepts or rejects a change, any subsequent reviewers cannot use the suggested changes feature to find out what has changed in the document.

Some of those problems can be avoided by comparing two versions of a document instead of tracking changes. However, current word processors do not have a way of attaching comments to a particular text passage within such a comparison view, making it difficult to have a discussion about a change within the document itself. In software development, it is common for developers to break down a large code change into smaller, self-contained updates called *commits* in order to facilitate code review; word processors, however, do not have a comparable way of grouping together related changes in a manner that communicates their intent and purpose.

## Upwelling design goals

In the design of Upwelling we aim to address the problems with existing writing software that were uncovered in our interviews. In particular, we explore the trade-off between real-time collaboration (suffering from the fishbowl effect) and private work (which risks merge conflicts) by providing tools that enable deliberate collaboration and help users navigate conflicts. Moreover, we explore how we can make it easier for collaborators to review each other’s changes, which helps them stay up to date as the document evolves and gives them confidence in the quality of the final document.

As we are interested in the social process of writing, Upwelling is designed for multi-author collaboration, not for individual writing. We aim to support the creation of carefully written formal documents (such as official reports or research papers), not casual writing (such as messages between friends or shopping lists). We also assume a model in which the collaborators trust each other (unlike, say, negotiations between conflicting parties drafting a legal document).

### Supporting divergence and convergence

To avoid the fishbowl effect, it is sometimes necessary for a collaborator to work in a private version of the document. This may be because they are trying out a new idea or accumulating edits that they want to share as a batch, or simply because they are offline. We call this effect – some changes made in one document version but not another – *divergence*.

After diverging, documents must once again converge, bringing together edits from different drafts to a final result. When collaborators make contradictory changes, however, convergence can result in conflicts. Some of these conflicts are apparent from the textual modifications: for example, two writers independently changing the same word into two different alternatives. We call such types of conflict *syntactic*. Syntactic conflicts are often possible to detect technically, such as when one user edits a sentence that another deletes, or when two users replace the same word.

Unfortunately, not all conflicts are so obvious: sometimes writers might make changes to different parts of a document that are individually correct but which produce a contradictory result when combined. For example, one author may edit a document to make a terminology change throughout, while their co-author adds a new section that uses the old terminology on a separate draft. When those edits are merged, the result is a document in which the use of terms is inconsistent.

When conflicts exist not in the structure of the text but rather in its meaning, we call this kind of conflict *semantic* to differentiate it from the *syntactic* conflicts described above.
Perhaps, in the future, AI tools will be advanced enough to automatically detect (and perhaps even resolve) such semantic conflicts. For now, however, we prefer to emphasize the social workflows of writing in Upwelling, providing tools to help users navigate such conflicts rather than trying to solve them automatically.

### Borrowing ideas from software development

Software developers collaborating on a code base are already intimately familiar with the process of divergence and convergence: for example, when using Git, a software developer typically edits source files locally on their machine, groups their changes into meaningful chunks called *commits*, and then shares them in the form of a *pull request* or *merge request*. Different team members may work independently on different *branches*, and only merge each branch once it is ready, often after having undergone a code review process.

By default, software developers generally do not use real-time collaboration in the style of Google Docs because it is unproductive for one programmer to suffer another programmer’s half-finished, buggy code while simultaneously working on their own, unrelated, half-finished code. The exception is when two developers are closely collaborating in real time on the same task (“pair 

This freedom to choose between offline private work and real-time interactive collaboration, depending on the situation, is something that we believe is also important for writers. Moreover, the process of code review, through which the quality of the final product is improved and team members are kept up to speed on changes being made, also has a direct analog in writing. Version control tools facilitate these workflows by providing branching, merging, comparison, and visualization of the editing history.

Unfortunately, version control tools like Git, which are designed for software development, do not suit the needs of writers. Git cannot compare or merge files from WYSIWYG editors such as Microsoft Word, and although it is possible to train writers in 

Upwelling aims to synthesize some of the benefits of existing writing tools with some of the benefits of formal versioning systems while avoiding the shortcomings of both.

### The human in the loop

As discussed previously, it is important in many writing projects that one or more collaborators review every change made in order to ensure the quality of the final document and uncover any semantic conflicts.

In software development, where code review is also an important part of the workflow of many projects, the version control tooling helps facilitate this review process: developers can structure their changes into *commits*, such that related changes are grouped into the same commit, and unrelated changes are placed in separate commits. A commit message helps provide a rationale for the changes (outside of the code itself), a branch name helps identify the commits, and GitHub’s “pull request” feature allows collaborators to have a discussion about the commits before and after they are merged.

In existing tools for writing prose, the review functionality is much more rudimentary. Google Docs’s *Suggested Changes* and Microsoft Word’s *Track Changes* treat every consecutive sequence of character insertions or deletions as a separate suggestion, and do not provide any way of grouping related edits into anything resembling a commit. When many small edits are made, the document’s margin quickly becomes cluttered, rendering discussion on suggested changes difficult to follow.

When a writer wants to diverge by working on their own version of a document, Upwelling allows them to choose the granularity of the changes before merging back into the main version. Like in a Git commit, the writer can collect related changes into a unit that can be reviewed efficiently, even if those changes modify many parts of the document. For example, a terminology change that requires replacing all occurrences of one word with a different word can be reviewed and merged as a unit.

We found in our interviews that the details of review processes differ significantly from one writing team to another. Upwelling therefore does not impose any one particular review process, but rather provides a variety of tools writers can use to review and integrate their collaborators’ changes.

### Reducing the risk of conflicts

While working on separate versions of a document has the benefit of giving writers space to experiment in private and avoid the fishbowl effect, it has the downside that prolonged periods of divergence lead to a greater risk of conflicts. Upwelling aims to help collaborators navigate this trade-off.

In Upwelling, it is possible to group and merge related changes as a unit. Similarly, unrelated changes can be reviewed and merged independently without the risk of introducing conflicts or losing work. This combination encourages writers to limit divergence and merge changes into the main document as soon as they are ready.

An author might use one draft to replace all instances of one term with another, allowing a collaborator to quickly review the change while concurrently carrying on discussions about a more involved rewrite of a particular section of the text. Splitting the two tasks into separate drafts makes it easier to review and merge (or discard) each one as soon as it is ready.

Upwelling also minimizes conflicts through a semi-private mode of writing. A writer can share in-progress changes without merging them into the final document. This offers less privacy than working completely offline but allows each writer to always have a sense of what their co-authors are working on.

## Upwelling implementation

Upwelling is a collaborative text editor that follows the design goals listed above. It is a research prototype, not a finished product, but we hope that it will inspire other collaborative writing products.

Upwelling introduces a few new concepts. Instead of writers editing the document directly, a document is built out of *layers*, and every edit occurs within one. Layers are titled, have one or more authors, and support comments for discussion or review. A layer can be edited until it is merged and joins the *stack*. The *stack* contains the complete editing history of the document.

### Layers and drafts

An Upwelling document is always edited within a layer. We call unmerged layers “drafts”. To change the document, a writer creates a draft and gives it a title to describe their intent to other contributors. Within the draft, a writer can edit the document as they would in any other text editor: by inserting or deleting text, by changing its formatting (e.g., from roman to italic), or by leaving comments.

Every edit to an Upwelling layer is tracked, and a contributor can always switch between viewing the document in its final state or visualizing their changes for easier review. This differs from existing products, which require a writer to enable change tracking before making a revision.

An Upwelling document can have multiple drafts at the same time, all of which are separate from each other: edits made in one draft do not appear in other drafts. As such, a writer who needs a private space can create a new draft and work there in solitude. But writers don’t have to work individually: a writer who wants to can also share their drafts to collaborate in real time.

Every draft has a title; this can be used to explain its purpose and intent, such as “Reworking the introduction” or “Copyedit Section 2.” A draft therefore serves a similar purpose to a commit or a branch in Git: it allows related changes to be grouped together, which helps collaborators understand what changes are being made and why. The existence of a shared draft titled “Reworking the introduction” also helps communicate to co-authors that they should avoid making changes to the introduction in other drafts, since they are likely to conflict.

In Upwelling, users can see all drafts accessible to them, including their name, creator, and the number of changes that have been made to them (to indicate how far the draft has diverged from the underlying document), via a drop-down menu. This is similar to GitHub pull requests, which show the number of added and removed lines. If several writers edit a draft, each contributor’s changes are highlighted in a different color.

### Reviewing and merging drafts

A draft collects a meaningful set of changes made by an author. Unlike change suggestions in Google Docs, where every individual edit results in a separate suggestion and potentially triggers an email notification, an Upwelling draft allows the author to share a set of related edits only when they are ready to do so.

A reviewer can leave comments on a draft to give feedback or request changes, but they can also simply edit a draft directly. Because Upwelling tracks all changes automatically, identifying those contributions is straightforward.

Once the authors of a draft are satisfied with it and want to incorporate it into the main document, they can *merge* it into the document’s stack of layers. Once the layer is merged, we no longer call it a “draft”. A merged layer can no longer be modified; to make any further edits, the writers must create a new draft.

Upwelling does not enforce any particular permissions or process around merging drafts: any collaborator is allowed to merge any draft. Some teams have formal review processes that may require, for example, an editor to sign off on every change. Rather than rigidly encoding such processes in our software, we believe that it is better to implement such processes as social conventions around how the software is used. For example, a team may agree that from a certain point onward, only the editor is allowed to merge drafts, even if in principle the software allows anybody to do so.

Our interviews suggested that even in professional environments, policy is governed by social rather than technical enforcement. Upwelling makes this strategy even more robust: because a full history of the changes to a document, with full attribution, is always preserved, writers and editors should always be able to determine who made a change and recover from any mistakes.

### Looking at the stack

A merged layer is placed on the top of the stack. The stack captures the full editing history of the document, starting from the moment it was created as an empty document, and each merged layer represents a batch of changes relative to the previous (lower) layer. The stack thus enables a writer or editor to review the work their colleagues have done over various periods of time.

To navigate this history, a user can select a reference layer using a drop-down menu. Selecting a layer causes changes after that point to highlight similarly to the “Show Changes” view within a single draft. This allows a collaborator to get caught up on changes merged by other editors, or just to get a sense of what parts of the document have been edited most recently and by whom.

Upwelling highlights insertions with a light background color and marks deletions with the traditional proofreaders’ 

### Rejecting changes

The primary means for accepting or rejecting changes in Upwelling is per-draft: an editor can choose to merge or, alternatively, delete a whole draft at once. Within a draft, an editor can also make fine-grained adjustments — inserting or deleting additional text at will.

We feel this approach is preferable to the span-by-span review acceptance (or rejection) process presented in existing products, because it allows for the grouping of related changes. By using several independent drafts, a copyeditor could separate uncontroversial changes, such as comma usage or capitalization corrections, from more invasive changes.

Although drafts replace the need to accept changes one by one, there are still some circumstances where it could be useful to reject changes within a draft, or even to revert an entire draft after merging it. We chose not to implement either of these features, but we describe a design for them below.

It is common for an editor to either disagree with a suggested change (and reject it) or to ask for further clarification or additional changes when reviewing a draft. We recommend supporting this with an inline interface as depicted below.

Similarly, it may be useful to provide a mechanism to retroactively remove a layer from a document’s history. Conceptually, we think the right approach would not be to rewrite history but rather to add a mechanism that creates a new draft which applies new edits that reverse the layer in question.

### Managing conflicts as drafts merge

As explained above, drafts are independent from each other in Upwelling: edits in one draft do not appear in other drafts. With this in mind, what should happen to existing drafts when a different draft is merged onto the stack?

One strategy would be to leave the other drafts unchanged. This approach has the advantage that a draft represents a stable version of the document, with no changes from other collaborators unexpectedly appearing. However, we believe that this approach increases the risk of conflicts, because authors of one draft may remain unaware of the changes that have been merged. Moreover, allowing a new draft to be based on an older draft in the stack leads to version histories that are complex to visualize and hard to understand, such as the “railroad track” diagrams depicted earlier.

Upwelling takes a different approach: all drafts *float* on top of the stack of merged layers; when one draft is merged, all other drafts are updated to include the changes in the merged layer. Using a Git analogy, every time a branch is merged into main, all other branches are automatically *rebased* on top of the latest main.

Users who are viewing a draft receive a notification when a draft is merged onto the stack. To avoid disrupting the writer’s flow, the currently viewed draft is not immediately updated: the writer can delay the update of their draft until a time when it is convenient for them. We believe that this design strikes a compromise between giving writers space to work without interruptions while also keeping them apprised of what their collaborators are doing.

Floating drafts help surface conflicts to collaborators so that they can be addressed. When different drafts contain syntactically or semantically conflicting edits, and the first of those drafts is merged onto the stack, the collaborators working on the unmerged drafts are notified. They can then use the change highlighting feature to see the changes in the merged layer and the changes in their current draft at the same time. A quick review of these changes allows the writer to determine whether there are any conflicts. (As discussed previously, detecting semantic conflicts requires a human review, as it cannot easily be automated.)

Because changes from merged layers are automatically incorporated into all existing drafts, a draft reflects the document that would be produced if that draft were to be merged now. There are therefore no surprises when the draft is merged, because any conflicts that may arise will already have surfaced. In settings where every word counts, an editor can carefully review every draft before it is merged and verify that the final document precisely matches what they have reviewed. This would not be the case if drafts did not float on top of the stack.

### How Upwelling works

The Upwelling text editor is a web application that works 

Upwelling’s collaboration features are built on top of the Automerge CRDT, which we use to track every single edit made to a document. To support asynchronous collaboration, the stack and any active drafts are combined into a single tarball (along with an additional Automerge document containing some metadata) then uploaded to the server. Real-time collaboration uses the sync protocol built into Automerge.

Upwelling is built on an experimental fork of Automerge which added some additional features not available on the stable release. The fork adds support for rich text formatting and attributing edits to individual authors. These features will be available in a more stable form in future Automerge releases.

Upwelling also works without a server. Writers can export a document as a file by appending `?download` to their URL, or import a document by dragging into the application. This capability is useful for debugging, but could also support alternative clients or data distribution strategies such as emailing revisions.

## Findings

We iterated through a wide variety of different designs during the Upwelling project. Each of them taught us something. Sometimes things that seemed promising in a paper mock-up were unusable in practice. Other times, ideas that came from our engineers inspired new features in the design. Our findings here are drawn from that process, and from our experience working with Upwelling within our group.

### Always-on change tracking puts writers in control of their experience

Google Docs conflates change tracking with review, requiring writers to manually choose to track changes. Because tracking changes is visually distracting, this forces writers to select between a comfortable writing experience or supporting thorough review. Upwelling, in contrast, records the full history of the document and lets a writer decide on the fly how much historic detail to surface.

In our interviews, we heard that making detailed edits to a paragraph with change visualization enabled can be distracting and make it difficult to read the final product. In some cases, we suspect that a reviewer may prefer to temporarily disable “Show Changes” in order to allow them to read the document more easily.

### Both real-time and asynchronous collaboration are needed

Our user interviews indicated that both real-time collaboration and asynchronous collaboration are useful in professional collaborative writing projects. However, most existing software supports only one of the two modes. Google Docs is real-time–only. Microsoft Word supports both real-time (for files stored in the cloud) and asynchronous collaboration (files stored locally that are shared by email, for example), but a file can only be in one of the two modes, and the modes cannot be mixed.

Git supports asynchronous collaboration only (creating a Git commit for every keystroke would be inefficient and onerous for the user). It is possible to combine Git with a text editor that supports real-time collaboration, such as Visual Studio Live 

As we developed Upwelling, we experimented with an asynchronous-only collaboration model but found that while drafts often have a single primary author, those authors often find it useful to be able to share an in-progress draft for initial feedback, or to ask for help with wording or copyediting before merging a draft. The need to combine both real-time and asynchronous collaboration produced the “layers” model: within a draft all changes are automatically merged in real time, but each layer is deliberately merged (presumably after review) onto the stack.

### Automatic merging is necessary but not sufficient

When several writers are collaborating in real time on the same draft, the network delay between the writers can lead to concurrent edits that cause their views of the document state to temporarily diverge. When writers are working on different drafts, the document states of those drafts can diverge more substantially. In both cases, algorithms are important for automatically merging those diverged states.

In the case of real-time collaboration, manually merging every keystroke would require an unreasonable amount of effort, so automatic merging is essential. With asynchronous collaboration, the situation is more nuanced, because merges happen less frequently, and greater divergence leads to a greater risk of conflicts. We believe that automatic merging is nevertheless useful in the asynchronous case, because it quickly produces a merged document that can then be reviewed for conflicts.

Upwelling performs automatic merges using a CRDT *rebasing* above). This merging process handles syntactic conflicts in a simplistic way: for example, if “white” is changed to “frosty” in one draft and to “soft” in another draft, the merged result is either “frostysoft” or “softfrosty.” This result requires human follow-up to correct, which should be made obvious through review using tools like “Show Changes.”

It is tempting to try to devise more sophisticated merging policies or user interfaces for resolving syntactic conflicts. Unfortunately, since semantic conflicts cannot always be detected, human review of the merged result is necessary in any case. We therefore prioritized a design that makes reviewing changes easy rather than investing in more sophisticated merge behavior.

### It’s better to avoid conflicts in the first place

Rather than introducing more sophisticated handling of conflict cases, we believe that it is better to manage the social process of writing in such a way that conflicts are less likely to arise in the first place. Many conflicts can be avoided if the collaborators communicate their intentions to each other beforehand (for example, “I’m working on the introduction today, please don’t touch that section”).

The current design of Upwelling allows editing intentions to be communicated through the naming of drafts. In future work, it would be interesting to explore further UIs for capturing intent: for example, a writer working on a draft might reserve a section of the text so that the software shows a warning message if others attempt to edit the same section in a different draft. Another possibility would be to provide early warning regarding potential conflicts by alerting collaborators if changes to other drafts have affected parts of the document close to the section they’re working on.

Finally, perhaps the most straightforward way of reducing the risk of conflicts is to limit the number of edits made in each draft: we encourage writers to merge drafts before they diverge too far from the existing stack. Smaller drafts are not always better (in the extreme, creating a separate draft for every single keystroke would not make much sense), but keeping drafts fairly small and focused on a single subject, and swiftly merging them onto the stack, helps writers stay apprised of what their collaborators are doing.

### Keystrokes should be grouped meaningfully

In Google Docs and Microsoft Word, every sequence of consecutively inserted or deleted characters becomes a suggestion that can be individually accepted or rejected. We believe that this approach is too fine-grained, and Upwelling instead provides drafts as a way of grouping together edits that are related to each other—e.g., a copyediting pass over a particular section of the document. This grouping is particularly useful for collaborators who need to review what has changed and why. Upwelling’s approach mirrors the use of commits in Git and other version control systems.

By allowing several drafts to be in progress at the same time, Upwelling allows unrelated changes to be placed in separate drafts, similarly to multiple branches in a Git repository. Git, however, offers some powerful history rewriting features that Upwelling does not; for example, it is not possible in Upwelling to “cherry-pick” a subset of commits from a branch, to combine (“squash”) several commits into one, or to split one commit into separate commits.

While Git’s history rewriting features make it possible to fine-tune the grouping of edits into commits, we decided that adding similar features to Upwelling would introduce too much complexity and add too little value. In order to make drafts as easy as possible to review, Upwelling does not allow the grouping of edits within a draft to be changed.

When reviewing a draft, if an editor finds that they want to use some but not all of the edits it contains, they can simply edit the draft to change the unwanted edits back to what they were before. We believe that this approach is much simpler to understand than a cherry-picking interface, and is equally effective.

### Draft layers should be independent from each other

Early during the design of Upwelling, we experimented with restricting drafts to a single author, corresponding to writing workflows with clear role separations between writers and editors. Single-author drafts make it easier to attribute authorship. However, we found that in this model, if an editor wanted to leave a comment or even just fix a typo in a draft, they would have to create a new draft to contain that edit, and this draft would have to depend on the earlier draft it referenced.

In Git, branches can be dependent on each other: you can have many branches that depend on other branches, not just on the `main` branch of the repository. Within a branch, commits from different authors can depend on each other. We experimented with “Git-like” dependent drafts in Upwelling and found that it made the model more difficult for users to understand and did not add meaningful value. Moreover, it encouraged the use of long-lived branches, which in turn increases the risk of conflicts.

A key insight of Upwelling is therefore that there should be no dependence between one draft and another, and that each draft should be able to contain edits from several collaborators (with edits automatically tracked so that different writers’ contributions can be highlighted when necessary). This approach enables a familiar real-time collaboration model within a single draft and allows any given draft to be merged onto the stack as soon as it is ready, independent of any other drafts.

### Drafts should float on top of the stack

When watching collaborative writers work, we noticed that they spend significant time reviewing changes to a document before calling a particular version the “latest” or “final” draft. We want to preserve this valuable process; we believe it helps editors ensure semantic consistency of the final document. In particular, we want to ensure that if a reviewer reads a draft and then merges that layer onto the stack, the resulting document will be exactly the same as what they just read.

Upwelling achieves this property by having drafts float on top of the latest stack; in other words, when a draft is merged onto the stack, it is also merged into all existing drafts. Any conflicts are therefore surfaced and resolved in the drafts before they are merged onto the stack.

A prerequisite for this behavior to work as expected is that in the moment where a writer merges a draft onto the stack, their copy of the stack is completely up to date. Two collaborators concurrently merging different drafts onto the stack would violate the principle that the reviewed draft is identical to the final document, because the result of the concurrent merges could contain unresolved conflicts. The Upwelling prototype currently does not enforce this rule.

One way of enforcing “one merge at a time” would be to keep track of the latest stack on a server, and to allow writers to merge onto the stack only if approved by the server (writers would still be able to edit drafts while offline; they would only need to be online to merge a draft). Another approach would be to design a custom CRDT that allows merges of drafts to only happen sequentially; if multiple collaborators concurrently tried to merge different drafts, only one would succeed and the others would revert back to draft status.

## Future work

We built Upwelling to explore the UX of collaborative writing software. Though you can try the 

### Permissions and sharing

Upwelling uses a fairly simple permissions model: a draft is either completely private to a single writer or it is shared with everyone who can view the greater document. As a convenience for demonstration, individual drafts receive a unique sharing link which can be shared to invite new people to that draft. Anyone with that link can view, edit, and merge the draft in question.

We deliberately did not implement a more complex permissions system because we found that professional collaborators usually trust each other, and therefore Upwelling can rely on social processes and conventions to manage the workflows for a document. For example, if a team has agreed that only the lead author should merge drafts onto the stack, they can abide by that rule without requiring it to be enforced in software.

In future work, it would be interesting to allow for drafts to be shared with only a few co-authors before sharing with the broader team. For collecting feedback from external reviewers, it would be good to be able to generate a link that allows a reviewer to make suggestions and leave comments on a particular draft without being able to see other drafts or merge that draft. This would allow the authors to send a document to several people for feedback at the same time and still keep each reviewer’s comments private from the others.

For some use cases, such as journalism, it might be necessary to enforce more rigorous access controls. For example, free-form editing of an article may be allowed while it is still being drafted, but once an editor has reviewed it, further changes may only be merged with the editor’s approval. In such a scenario, drafts may also need additional attributes, such as a flag to indicate whether it is ready for review yet.

In security-sensitive situations, as when writers are working with sources whose identities they wish to protect, it could be important to elide the history of a document. Because Upwelling currently records and shares the full history of a document, there is no way to excise names or information that may have appeared in earlier drafts. This is a problem caused by keeping too much history, a consequence of changing from a design that keeps much less.

Empowering users to control how much document history they share with their collaborators is an open problem. Solving this will demand both new user interfaces and also new data structures. CRDTs are currently designed to guarantee that users will see the same results from the same inputs. In this case, users will want to see the same results from different inputs.

### Viewing multiple drafts at once

The user interface of Upwelling currently only allows users to view one draft at a time, even though the underlying CRDT library does in principle allow several drafts to be viewed at the same time. For example, if several reviewers each leave suggestions and comments in a separate draft, the lead author might want to see all reviewers’ comments at once rather than having to switch between several draft views.

When a writer is viewing a single draft and they make any edit to the document, that edit can be added directly to the draft they are viewing. The problem with viewing multiple drafts at once is that if the writer wishes to make an edit, it can be unclear which draft that edit should be implemented on. For this reason, the document would probably have to be read-only when viewing multiple drafts at the same time.

### Interface improvements

#### Change highlighting

When reviewing changes (within a draft, or across layers in the stack), change highlighting aims to make it efficient for a reader to understand how a document has changed. We believe that Upwelling’s style of change highlighting provides better readability than the style used in Microsoft Word and Google Docs, which render deleted content as crossed-out text. Instead, Upwelling hides deleted text behind a small marker: the “➰” symbol. This design could be refined in future work by exploring alternative ways of highlighting changes.

#### Partially reverting changes

In Upwelling it is easy to make small edits to a draft and then to “accept all” changes in that draft by merging the layer. However, in some cases, a reviewer might want to accept only some of the edits in a given draft and not others. We designed a feature for partially reverting the edits within an individual draft but have not yet implemented it in Upwelling. This feature would be worth exploring in more detail.

#### Attribution and additional context

When highlighting changes, Upwelling currently displays authorship through nicknames and colors. It would be interesting to explore showing additional information about edits (e.g., time, name of the draft, merged by, etc.), especially for teams that require more formal review and approval processes.

#### Other content types

Upwelling currently only supports text with basic formatting; it does not yet have support for code, images, videos, tables, or other content types that also occur in professional writing. Further work should investigate what kinds of collaborative workflows arise around these types of data, and how to support them.

### Supporting different workflows

Our user interviews and the design of Upwelling have focused on professional non-fiction writing. We already understand from our user research and professional experience on the team that fiction writing, in particular, tends to exhibit different editing patterns and team dynamics than non-fiction writing. In future work, we would be interested in understanding these behaviors more deeply and designing user interfaces to support different working styles.

While Upwelling is currently a research prototype, we would like to see these ideas incorporated into existing word processing software. An ideal implementation would offer a file format or exchange protocol that makes it possible for writers to use the writing software of their choice to create documents while offering support for Upwelling features.

## Conclusion

Professional writing is ill-served by both a traditional file-based approach and the modern cloud approach to collaboration. Writers and editors across a variety of disciplines perform elaborate work-arounds due to the shortcomings of existing solutions. In particular, writers express a desire for creative privacy and only sharing work when it is ready, and editors need to closely follow the development of a document so that inconsistencies do not creep into the final version.

Upwelling addresses these needs with a new model that unifies real-time and asynchronous collaboration. Instead of treating writing and suggesting changes as separate modes, Upwelling tracks all edits and allows changes to be highlighted retrospectively. To support real-time collaboration, Upwelling tracks changes at the granularity of individual keystrokes, but it also encourages writers to group related edits into sets of changes that can be reviewed efficiently, similarly to commits, branches, and pull requests in Git.

We hope that future collaboration tools for professional writers will incorporate the ideas from the Upwelling research prototype and adapt them to the needs of their audience. As the technological basis for Upwelling (in particular, CRDT libraries that can efficiently track editing history and automatically merge drafts) matures, we believe that Upwelling presents an opportunity to significantly improve the writing workflows of many knowledge workers, including scientists collaborating on research papers, journalists writing articles, and educational writers preparing learning resources.

We welcome your feedback: 

*Thank you to Chris Sun (daiyi) and Peter van Hardenberg, who contributed to the development of Upwelling. Special thanks to Orion Henry, who worked tirelessly on the Rust implementation of Automerge to make Upwelling fast and reliable. Thank you to the Peritext team (Geoffrey Litt and Slim) for conducting interviews that inspired this project, and for their design of Peritext, the rich-text CRDT used in this prototype. Thank you to Todd Matthews, who designed the diagrams, and to all contributors who gave valuable feedback on early drafts of this paper. A special thank-you to all interview participants, who gave their time to inform the design goals.*
