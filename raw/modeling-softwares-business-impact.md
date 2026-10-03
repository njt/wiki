---
url: https://www.jamesshore.com/v2/blog/2026/modeling-softwares-business-impact
date_fetched: 2026-10-03
---

## Modeling Software’s Business Impact

### September 29, 2026

Are your teams working on the right thing? It doesn’t matter how fast they are if they’re running in the wrong direction.

This is part 4 of my series on assessing AI’s impact on software development. Unlike the other essays in this series, this one isn’t really about AI at all. It’s about *building the right thing.* Or, more accurately, understanding the value of what you build. Because, ultimately, your ROI isn’t about how you build your software, but what you deliver, and what changes as a result.

In part 1, we looked at why assessing impact is important, and what not to do. In part 2, we looked at how to measure AI’s impact on delivery speed. In part 3, we considered how to measure the unintended consequences of using AI. Now, in this part, we’re examining how to model and forecast business outcomes. Finally, the epilogue puts it all together with notes you can share with your CFO.

### Measuring the Wrong Thing

You’d think that modeling the value of what teams produce would be a no-brainer, but it’s surprisingly rare.

It’s not particularly difficult. It’s just... organizationally complex. And there are some unavoidable compromises. Most of all, modeling value just isn’t something people think about with software development. They’ll spend months trying to fine-tune cost estimates and predictions, and put hardly any thought at all into making the software more *valuable*.

But there’s no guaranteed relationship between software cost and value. You can spend twenty million dollars developing software that’s worthless, and two million dollars developing software that transforms a business. Cost improvements are typically marginal, and difficult. Value improvements are *game-changing.*

Focusing on value is particularly important in the AI era. If writing code is no longer a bottleneck, then what is? Early signs point to the bottleneck moving to our *customers*.

Lee Eason, an engineering leader I spoke to recently, hypothesizes that customers have a limited ability to absorb new features. When software development was expensive, we used that cost to force prioritization. Pet projects ended up on the back burner, and the most valuable work rose to the top. (Well, sometimes. I wish it was that clean.)

Now, we can produce a lot more work, but that doesn’t mean customers can *absorb* it. They’ve got their own work to do. Eventually, they’ll stop paying attention. We need to treat customer attention as the scarce resource it is and only ship the things that are truly valuable.

Unlike the engineering bottleneck, this one’s invisible. In the old days, if we didn’t prioritize, we didn’t ship our most impactful work. In the AI era, our most impactful work could just be ignored.

I’m not going to talk about how to improve value in this series, but I am going to talk about how to *plan* and *prioritize* it. Because nothing else matters if customers don’t care about your software.

### The Attribution Problem

Part of the reason people measure costs is that it’s easy. Measuring value is easy, too, at the big picture level. It’s the literal bottom line. The challenge is attributing where changes to the bottom line came from.

Let’s say you release a new feature, and see purchases improve 10%. That’s fantastic! Chalk it up to the new release!

But your VP of Sales also launched a quota-retirement promotion that put the sales team into a feeding frenzy. And marketing got an important tech influencer to drive tens of thousands of people to your home page. The partnerships team stepped up, too, convincing your partners to highlight their integration with your new feature in their own products.

Who gets the credit for the success? Obviously, everybody contributed. But how much? Was your new feature even relevant at all, or was the success just due to the sales, marketing, and partner push?

A related problem is how delayed these numbers are. You won’t get revenue numbers until months after a new feature is launched, especially in B2B sales, which have a long sales cycle.

Address these problems by *modeling* value rather than measuring it directly. Each part of the model gets proxy measurements that help you assess and improve the model over time. The proxies give you faster feedback and allow you to reduce the impact of other departments’ activities.

I like Dave McClure’s “Pirate Metrics” for this purpose:1 Acquisition, Activation, Retention, Referral, and Revenue. (AARRR for short. You can see why they’re “pirate” metrics.) For example, you could measure what percentage of users are trying your new feature (activation), and then what percentage are continuing to use it over time (retention).

1Many thanks to Jeff Patton for introducing me to Pirate Metrics.

You don’t have to limit yourself to pirate metrics. For example, if a usability initiative is expected to reduce customer churn, you could measure how your support calls change, or whether usability is cited less often as a reason for cancellation.

### The Organizational Challenge

To make the model, you’ll need to get the rest of your organization involved. The specifics will depend on how your company is organized, but broadly: Sales will help you understand your initiative’s impact on new sales. Customer Success will help you understand the impact on upsell and retention. Marketing, lead generation. Partners, potentially all of the above. Etc.

When I was VP of Engineering, it took me a couple of years to establish this program. Most of that time was spent convincing people to get on board, so it doesn’t have to take you that long.

I worked closely with the VP of Product on the problem. We started by introducing it to our Chief Product Officer, who brought it to the CEO. The CEO liked rigorous approaches, so he was on board. That was enough cover for us to make creating a financial model part of our official process for prioritizing major initiatives. We called them “product bets.”

At first, the models... weren’t great. People gave us dramatic forecasts without really thinking them through. But then a new CFO joined the company, and I introduced him to our product bet approach. He was supportive—it turns out that CFOs like rigor too, surprise, surprise—and he helped us improve and critique the models. That gave us a better understanding of what to ask of our various business partners, and the support of the CFO made it easier to ask for the information we needed.

People were uncomfortable making forecasts. They didn’t want to be seen as making commitments that they couldn’t meet. The support of our CEO and CFO helped here, but mostly it came down to the personal relationships we had built with our peers. We explained what we needed and cajoled them into giving it to us.

I’m no longer in that role, but I expect the process will become faster and easier over time. The challenge is in establishing a new process. The actual modeling isn’t that difficult—a bit uncomfortable, perhaps, because you never *really* know what the right answers are. Eventually, it will be part of “the way we do things here,” and then a habit. Patterns and templates will form. High-level involvement turns into delegation turns into routine.

### Creating the Model

The model is a spreadsheet that forecasts net present value (NPV) over some period of time. We used five years. If you’re not familiar with NPV, your Finance department can help, but the short version is that you predict how much you’ll make, or spend, in each of the following years, and then discount that amount according to how far in the future it is. Sum all the years together and you get the net present value.

Your model won’t be a perfect reflection of reality. Don’t try. Focus on these questions: What financial benefit are we expecting from this initiative? And why? It’s more important to establish the broad strokes than to capture every nuance or get every detail perfectly correct. Be prepared to make some educated guesses, and not-so-educated guesses. There’s too many unknowns not too. You might want to make multiple versions: optimistic, pessimistic, and “best guess.”

For a typical sales-oriented business, your model will include five categories: new sales, upsell, retention, cost savings, and expenditures. For each of these, start with the business justification for the initiative. What is it supposed to accomplish, and why? The answers go in the model.

For **new sales**, you might include the number of customers the new initiative will gain you, the number that its *lack* will lose you, and how much each customer is worth.

For **upsell**, consider how many people will upgrade and how much revenue each upgrade will bring.

For **retention**, consider the value per customer, your churn rate, and the percent of at-risk customers that the initiative will save.

**Cost savings** is most applicable for internally focused initiatives, but even customer-facing initiatives can have an effect by reducing support costs or other manual work. Still, be cautious of this one. Cost-saving initiatives are a common request—people can always think of things that makes their lives easier—but you can usually get a lot more value out of sales and retention than you can get out of optimizing costs. The low-hanging fruit has already been plucked.

**Ongoing expenditures** are the costs you’ll incur as a result of the initiative after it’s been released. For example, if you’ve built an AI-powered service, you’ll have ongoing token costs. They can be modeled by estimating number of sessions and an average cost per session.

And finally, **development costs** represent the money you’ll spend to develop the initiative. It’s dominated by the cost of labor and AI token costs.

My preferred approach to development costs is to use a milestone and ceiling approach. Based on the net present value of the rest of the model, you have a rough idea of the maximum you can spend and still have a good investment.2 Choose a spending cap, break it into spending milestones, and use an iterative “Build-Measure-Learn” approach to learn what to build as you go, checking in with an executive steering committee at each milestone and making a “continue / cut losses” decision based on what you’ve learned.

2If you choose a discount rate that represents the minimum rate of return you want to see, the net present value represents the most you can spend.

(You can also do a more traditional predict-and-estimate approach, but I think an iterative approach allows you to steer to more value.)

### Making It Useful

“All models are wrong, but some are useful,” George Box famously said.

Your financial model will definitely be wrong. There are too many guesses involved for it not to be. The question is: how do we make it *useful?* Despite its flaws?

If you make the model a requirement for prioritization—not for bugs and little changes, but for substantial initiatives—then it can be a useful way of improving your understanding of the strategy underlying the initiative. All too often, stakeholders present a solution without verbalizing the problem they’re trying to solve. The model will force some of those hidden assumptions to the surface, where they can be dissected and discussed.

Once they’re on the surface, I’m a fan of a somewhat adversarial approach to prioritization. Not so adversarial that it becomes dysfunctional, but I think it’s useful to ask proponents of various initiatives to critique each other’s models. Without this critique, it’s too easy to get inflated estimates without much thought put into them.

I found it very useful to involve Finance, and initially my CFO. They have an interest in clear-eyed assessment of opportunities and costs. They can help get the ball rolling on a critique, and demonstrate how a model can be criticized and improved without damaging relationships.

On the other end, after an initiative has been delivered and the work is done, you can use the model to demonstrate product development’s accountability and value. People get stuck on delivery dates as an accountability mechanism. The model won’t prevent that entirely, but it sure is useful to be able to point to a model of the *value* you delivered instead.

This is also an opportunity, finally, to judge how AI is affecting productivity. Use your measured speed increases to model how much initiative would have cost without AI, and forecast maintenance costs using your models of unintended consequences. The results will give you a basis for judging AI’s productivity benefits. Just remember that the models are based on guesses and assumptions, not objective reality.

And, of course, as you’re developing the software and iterating through releases, keep an eye on your proxy metrics. Talk to your business partners about actual sales results and cost savings. Use the results to revise your forecasts and move the model closer to reality.

### Prioritization is Strategy

I’ve worked with a lot of companies, and prioritization is almost never done well. But software priorities are the realization of a company’s strategy. *Nearly everything* a company does is supported by software in some way. Software priorities—whether purchased or built—quite literally determine what the company will be capable of next month, next quarter, next year.

So when people ignore value to focus on cost estimates and predictions, when they say “everything is top priority,” when they don’t engage with the question of *why* something deserves development... they’re unintentionally sabotaging their company’s strategy.

That’s why this is the most important essay in the series. Delivery speed and unintended consequences are important to find the best approach to using AI. Modeling value is important for understanding *what to do with it.*

It’s not perfect, of course. Financial models are no substitute for good judgment around product and organizational strategy, and you shouldn’t blindly choose initiatives just because they have a high net present value. The models are there to inform your portfolio prioritization decisions, not substitute for them.

Like all the metrics I’ve presented in this series, modeling is subject to gaming. In this case, the temptation to hide bad news will be high. If a proxy measurement comes in poorly, or anecdotal sales evidence is negative, folks might be tempted not to update the model to match. Keep an eye out for people hiding bad news. As always, you’ll be better off using this as *a tool for decision-making* rather than a way of assessing people’s performance.

This approach is also fairly heavyweight. It’s not suitable for small features and bug-fixes. It’s a complement to the other metrics I’ve mentioned, not a replacement for them. You can get a sense of speed and unintended consequences relatively quickly. Modeling and assessing value will take longer. But that time will pay off with a better understanding of the value you’re bringing as a whole... and, with luck, patience, and hard work, better ability to execute on your company’s strategy and more success overall.

This brings us to the end of my series on getting the most out of your AI investment—and out of your software investment in general. Next, we’ll bring it all together in an epilogue, which you can also share with your CFO and other members of your leadership team.
