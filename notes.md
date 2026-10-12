I’d like to share my experience and journey through a transformational change in how we use AI to augment engineering work.

I’m a curious engineer.

When I see something interesting, I want to get underneath it. Understand how it works. Push it a bit. Find out where it breaks.

That was my reaction to Faraz’s GVAR personal project, and the way he used AI agents to do engineering work.

I was impressed. But I also thought: “Yes, but that’s a personal project. Try doing that at UBS.”

Here, we have systems that have been running for years. Dependencies we cannot casually break. Controls. Real people relying on the output.

So curiosity got me started, but I had a hypothesis: this would hit a wall in our environment.

I wanted to find that wall.

I took the approach into the Data Distribution Hub, our replacement for a research distribution engine that has been running for twenty years.

Why modernisation?

Because so much of the knowledge is scattered across old code and people’s memories. And being wrong has consequences.

It gave me a practical way to test the approach against the things that made me sceptical.

And the slightly uncomfortable headline is: I set out to prove it couldn’t work here.

I failed to prove that.

I thought my technical ability would be the test. I discovered that the harder skill was recognising what we didn’t know, and organising the work to find out.

That’s the journey I want to share today.

I’d like to think I’m a strong engineer. But pretty quickly, I found that the code itself wasn’t the hardest part of this modernisation.

Think of a house people have lived in for twenty years.

It has lots of rooms. People have moved in, moved out, added things, changed things, and found their own ways of making it work.

The cupboards are full.

Some things are useful every day. Some are broken. Some haven’t been touched for years.

And some look useless until someone says: “Please don’t throw that away. I need it once a year.”

Now we want to rebuild that same house.

We can design better wiring. Better plumbing. A stronger structure.

But before we empty those cupboards, we need to understand what’s in them, and what people still depend on.

Otherwise, we give everyone a beautiful new house, and on the first morning someone asks: “Where’s the thing I need to do my job?”

That’s our modernisation problem.

Why does that exception exist? Who still relies on it? Is that strange behaviour a bug, or has someone quietly depended on it for ten years?

You can build a replacement that looks clean and passes its own tests, and still miss something important.

Because you’ve tested what you knew to ask about.

The code helps us see what is there. Our colleagues help us understand why it matters.

We need both before deciding what the replacement must preserve.

So what I want to show you is what happened when we applied this approach to the part of our work that hasn’t been fully written down.

We gave the agents bounded jobs: discover the behaviour, describe the rules, challenge the interpretation, build tests, and investigate differences.

The useful question became: what can we establish from the evidence, and what still needs a person to decide?

Let me show you what that looks like.

We start with a small part of the legacy system, rather than asking an agent to understand everything.

The discovery work traces the relevant code, database objects, flags and jobs, and brings the findings together.

Here is a behaviour it has surfaced. And here is the code behind it.

That click matters.

I can inspect the claim. A colleague who knows this area can challenge it.

Instead of asking someone to remember everything this system does, we can ask a focused question: “We found this. Is our interpretation right? Does anyone still need it?”

It gives us something concrete to discuss.

Then we turn the findings into business rules in plain English, and a separate checking pass goes back to the code to challenge the interpretation.

And it doesn’t all come back green.

Some findings are partial. Some aren’t supported by the cited code.

A convincing first answer can still be incomplete or too broad. The checking pass gives us another opportunity to catch that, although it can have blind spots too.

The gaps tell us where to investigate next.

Someone who understands the business still has to decide whether a rule is correct, needed, or obsolete.

The agent helps us open the cupboard. The people who use the house decide what belongs in the new one.

Once a rule is approved, we make it testable.

We record what the old system does for that case, so we can check whether the replacement does the same.

Then we deliberately break the behaviour and check that the test catches it.

That gives me more confidence than a green result alone. I’ve seen the check detect the problem it was designed to detect.

We also compare the systems using a copy of traffic, without sending the replacement’s output to live consumers.

Differences become things to investigate.

It could be a new bug. An intentional change. A legacy quirk. Or a problem with the comparison itself.

The agent helps assemble the evidence and, where appropriate, propose a fix with a test that reproduces the problem.

An engineer still reviews the diagnosis and the change.

We have evidence for the cases we’ve checked. We still have to understand what remains outside them.

So what did I learn?

I started by asking whether the AI could handle the technical work.

I ended up paying much more attention to how I defined the work in the first place.

“Understand this system” sounds like a task. But it leaves a lot unsaid.

“Trace this entry point. Cite the evidence. Separate facts from assumptions. Tell me what you couldn’t establish.”

That gives me something I can review.

I had to get clearer about the outcome, the boundaries, and what a useful answer would look like.

I also changed how I reviewed the output.

I began asking: where did this come from? What might be missing? How would we find out if it was wrong?

A polished explanation wasn’t enough.

The gaps became part of the work, rather than something to gloss over.

And it changed how I thought about our experienced colleagues.

Their knowledge is still essential. But we can give them a better starting point.

“Here is what we found. Here is the evidence. And here are the questions we couldn’t resolve.”

That makes their judgment easier to bring into the process, without asking them to reconstruct twenty years of history from a blank page.

Finally, I had to rethink what being a strong engineer means in this way of working.

I still need technical depth. I need it to spot a weak interpretation, question a test, and assess a proposed fix.

But I don’t have to personally perform every step to take responsibility for the result.

I have to know where to challenge, where to dig deeper, and when the evidence isn’t good enough.

I had to evolve my way of working.

And I’d like to think that makes me a stronger engineer. More deliberate about the decisions I’m making, and more honest about what I still don’t know.

I started with Faraz’s project and a fairly confident thought: “Interesting, but it won’t work here.”

This experience gave me enough evidence to reconsider that position.

It also gave me a much clearer view of the work that still needs human judgment.

My question now is: where else are skilled people spending their time piecing together information before they can make a decision?

I’d welcome your help identifying one bounded problem in your areas where we could test this approach, and agree upfront what evidence would make it useful.

I came in looking for the limits of the technology.

I came out reconsidering how I work.