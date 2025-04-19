# Where's my documentation, dude?

[ru](../ru/index.md)

![looking for documentation](https://i.kym-cdn.com/entries/icons/original/000/019/277/confusedtravolta.jpg)

> Disclaimer:
> The characters, company, situation, brand names, ~~meaning of life?~~ - are fictitious.
> Any similarities are likely to be completely accidental.

## So...
Surely you've experienced starting a new job and diving into a new environment. Regardless of your experience - whether it's a year, five years, or your first job - you are new here, and everything is new to you. Of course the more experience you have, the more you find out the familiar things around you: a task tracker, actions, a management hierarchy, colleagues with familiar role names, tools and approaches to development - you've already seen it. But still, the ghost of novelty is in the air and you need to catch it and move it away. You  need to integrate into the flow of processes and solutions for the company and the project and into the system where the project is developing.

After all, the sooner everything becomes habitual, the sooner you'll start delivering what managers repeat daily like a mantra from sacred management scriptures - **business value**.

In other words - **benefits** for the company.
Everyone is interested in that and you are the first of all. You need that to keep your position and, perhaps, move up. Do you agree?

You go through an onboarding process and colleagues share details about the system:

*"Here we have a presentation layer, here we have a layer for working with the database, here is the business logic... This microservice does that, this one does something else... here is Kafka, here is Redis... Here is a diagram of how this is __approximately__ related to each other, it's __a little outdated__ but essentially it's correct... And here is all the documentation about the system...."*

You're presented with a page from the corporate system _Crowdfluence_ (thank goodness it's not _Lotion_, you think), containing about fifty sections and subsections, the most recent update being six months ago...

"At least there IS some documentation here..." - you think, afraid to ask yourself the following questions:

- "How long will it take to read and understand everything?" 
- "Will it help me start being useful faster?

After thinking about it for a bit (and maybe even skimming through some of the documentation sections), you decide you'll figure it out as you go...

About a month has passed since you started working on the project. You have begun to understand how some parts of the system work but not everything - the project is quite large, and there is quite a lot of business logic.

One day, during a daily meeting, you're assigned the task of investigating a new bug. It is quite an unpleasant bug, but it is not so critical that any of the old-timers collegues would put aside all their work on new features and deal with it. It is assigned to you.

You've spent a day or two on research, but it, unfortunately, does not give you an answer to the question "What is the *cause* for this behavior?".
You've searched by keywords for the description of domain entities in the project documentation, description of modules that (as you've assumed) contain the cause of the bug, but haven't found answers.

You've spent hours meditating on the code, switching between files and function call locations.
The code is brilliantly written - it is clear that the people who wrote it tried to adhere to good practices and maintain the quality of codebase. There are tests - there are many of them, and the code coverage rates are extremely high. Each module, each public function is provided with a description. You have tons and tons of information telling you "**what** this or that code does", but you still do not understand "**why** it is implemented this way".

After that, you decide to assaut your colleagues with questions like, 'Why is the implementation like this now?' After all, it's not enough to find a place where the system *deviates from the normal behavior* - you also need to figure out *what is the way of returning the system to the "rails of normal behavior" without breaking anything nearby while you would correct it*.

One of your colleagues tells you "I remember that something *related* to that problem was discussed once, and Theodor K. had fixed something, but he quit a long time ago. Here is a *link* to the task. You may get up the context from it..."

You're following the link and seeing *that*:

![the context](../resources/context.png)
Looks familiar?

After reading the discussion, you realize the fix for the bug was a temporary solution to close an error that occurred as a result of getting data from the third-party service that the system was integrated. More over that you'd came across changes related to this bug and the IDE had showed them.

But how were you supposed to know that the temporary solution was supposed to be removed when the external service started sending data the expected way?

No chances you get through it, and here's why:

- You weren't here when the integration with the external service was done
- You weren't here when the problem occurred
- You weren't here when it was discussed and the fix was made
- You didn't have a clear roadmap showing you the way to solve the problem

In short, you had absolutely no **context** of what happened.
As a result, fixing a bug turns into a detective investigation (where the main thing is not to find your own fingerprint).

Despite all its completeness and verbosity, the documentation describes the system's operation quite generally and does not reflect such things as a temporary solution. And often, much of the permanent one absent too.

## What is documentation?
> The author means by documentation all knowledge about the system: technical, non-technical, architectural, business, etc.

**Documentation** is a **consistent**, **currently relevant** document or set of documents that provides knowledge about the project:
- The idea of ​​the project, the goals and objectives that the project solves (why do we make the project?) - this is the raison d'être[^1] of the system being built. ("earnings" is never the goal of the project. it might be only the owner's  goal)
- Project requirements (what are the project's limitations?)
- Roadmap of project (how will the project goals be achieved?)
These knowledges must be thought out and described **initially**.

And before you can tell that waterfall is dead and showed its total inefficiency, and that agile is a trendsetter today...

The above is a necessary but not sufficient part for a **LONG-TERM** and **COMPLEX** project. And in case some of that changes, the project must done and start a new one (possibly borrowing something from the existing ones) corresponding to new ideas and goals or new requirements.

Otherwise, it may happen that [Burj Khalifa](https://en.wikipedia.org/wiki/Burj_Khalifa) has just been named as a modest family cafe and is went to change the foundation on which it stands.

Details about "how" and "what" will be done to achivment of the goals and correspondence of requirements of the design system within the development plan can be supplemented and adjusted. **Only if it does not contradict the basis**. Moreover, new details **MUST** find their place in the documentation.

The purpose of documentation is to have ***knowledge*** about the system's operation at hand, and to reflect its current state, for solving work tasks - the notorious **context** - and not to be ***just information*** describing some facts about the system. 
If you like, then documentation is a augmented roadmap of the system or a navigator with rivers, mountains, roads, showing how to get from point "A" to point "B" **at the current moment in time**, taking into account weather conditions, time of day and traffic on the roads.

[^1]: The meaning of existence (fr.)

## Where does context live?
> In the article, the author uses the word *context* to denote knowledge about the behavior of a system, including the reasons for such behavior,
> since decisions about changing and developing a system are made based on *context*.

According to (purely subjective) statistics, *context* could be found:
- in a ~~knowledge base~~ heap of information where the description of the main elements of the system is concentrated;
- in the task tracker with a description of features, incidents, distribution of tasks among employees and the time of completion and comments on the tasks with some attachments;
- in instant messengers, where discussions of tasks also actively take place, spreading the answer to the question "what and how it should be done?" over hundreds and hundreds of messages in different groups/threads/topics or pms;
- in the version control system, there you can find who added/deleted/changed something, when and in relation to which task in a task tracker it happened;
- in repository management systems (like github or gitlab), where sometimes in addition to the list of changes you could also find discussions within the code review process of the solution;
- in the code, where you have a full description of **how** the execution logic is made at the moment (but without data from the production environment... without real infrastructure... without an answer to the question "**why** the system is made this way?"). Here in the code you can also come across `TODO: redo later. TASK#1234567`;
- in the code, where you have a full description of **how** the execution logic is made at the moment (but without data from the production environment... without real infrastructure... without an answer to the question "**why** the system is made this way?"). Here in the code you can also come across `TODO: redo later. TASK#1234567`;
- in tests (you do have ALL the TESTS for ALL the functionality, right?);
- in the email, where decisions were discussed (at least in the past years), especially with external agents of the company (good luck getting emails of Theodor K. 's work inbox);
- at meetings and phone calls (they are recorded, aren't they?);
- in the heads of colleagues (please put down the surgeon's saw... and try to find Theodor K.);

Given all of the above, try to get an answer to the question:
"How many actions do you need to get up the context of the problem being solved?"

Even if you are lucky, to answer the question: "**why** does the system work the way it does now and how it **should** work?" You, as a developer, need:

to find a commit in the version control system -> to find the task by the commit -> to find the task description in the task tracker by the task id -> to find a link to the documentation in the task tracker -> to follow the link to the documentation section and *maybe* at the end of this breadcrumb trail you would see the answer to the question "**why** it is like it is, and how it really should be".

## A fly in the ointment...
...which "progress" adds to the already unpleasant picture these days. When someone came up with the *BILLIANT* idea of ​​generating documentation and tests based on the current code base. That someone probably missed the fact that the description of the system (documentation) is the **rootcause** of the implementation (executable code), and **not the consequence**. And if we develop the idea - the term "**source** code" (in this case, the stress is on the word "source") is more suitable for documentation than for implementation. In defense of the statement in this "chicken-and-egg" question, one can cite the fact that even under the most careful and attentive look, the executable code tell not so much about the reasons for its appearance. 

> REM: Here the author would like to say hello to the supporters of the idea of ​​"self-documenting code".

## Every cloud has a silver lining...
After quick research for solutions to the problem related to keeping system documentation up to date, autor revealed several interesting tools and instruments.
One such idea is [docs-as-code](https://www.writethedocs.org/guide/docs-as-code/), when *the documentation changes to the same extent and in the same way as the executable program code*.
Probably that finding could say that even if the problem isn't widespread - it "sores" enought.

## A place for forward step
Is it possible to develop the idea of ​​documentation changing during development, while maintaining its relevance, completeness, and transparency if possible, with minimal costs?

Before trying to find an answer to the question, it is worth analyzing the "conventional" approach:
1. An idea or need to add new functionality to the system arises;
2. A group of people (stackholders, analysts, architects, managers...) is assembled and they work through the idea over several meetings;
3. As a result of this meeting, a **document** is obtained, which  describes the top-level vision of what should be in the end;
4. A task is created in the task tracker (with the type 'story', 'epic', or 'epic story', or even 'legendary-heroic-epic story'), with a description of **what** needs to be done and **why**. The existing **document** is also attached;
5. Next, the analyst, the manager, and the development team discuss the **description of the idea** for **completeness**, **clarity**, and **consistency with the current system** (the scrum master constantly tries to intervene in the discussion, being very interested in finding out "how many story points the development of this or that functionality will take and why so many");
6. The legendary-heroic-epic story is decomposed into less ~~legendary~~ voluminous tasks and distributed among the development team members;
7. The functionality is implemented, tests for the new functionality are written, the release is assembled;
8. The release is tested on the test stand, the implementation of the idea is shown on the pre-production stand;
9. There are some bugs that are fixed (task in task tracker, linking with epic and current release, etc.) then everything is repeated from point (7) until the functionality is good enough to be released to real users;
10. The legendary-heroic-epic story ends;
11. New entries may appear in the heap of information (but this is not certain);
12. The cycle repeats from the beginning...

> In the written sequence of the process, many steps are intentionally omitted.
> Otherwise, it would resemble the spreading ash tree [Yggdrasil](https://en.wikipedia.org/wiki/Yggdrasil), holding all nine worlds.

You may have noticed that in the described process the development team had a moment when they shared the same **context** based on a **document** at the same time and in the same place, but then, unfortunately, the context splited up into many small contexts in different places and lost its connection with the original source. The new record in the heap of information was made after the fact, and this entry will not allow a person to understand "how and why the system works this way" without additional investigation.

Why not change the process and make code changes to the system along with the documentation taking into account the docs-as-code approach?

In this case, the description of the functionality or its change is undividible with the functionality itself:
- functionality and description are synchronized for "free", including in time due to the version control system;
- no need to rummage through hundreds of places to collect all the **necessary and sufficient** context - here it is next to the changes themselves;
- documentation can be collected into a single document if it needs, which reflect the current behavior of the system;
-  one more level of validation appears for "free": when making changes to the code that change the logic - they should be described. After all, something has changed, otherwise what is the point of the work? (these are almost like tests, but written in ordinary words);

Maintaining the "context" in this case will be much easier due to the inertia of the process. After analyzing the task there is already a text description of what needs to be done and why: why not make it the "first brick" of the solution? Why not start MR, where the first commit will be that description?

The new process will look like this (the first 6 points remain unchanged):

(7). The description of what needs to be done - in other words, **what state the system should reach** within a specific task - is added as the initial commit in the version control system branch. In this way, all of participants agree to these changes together.

(8). The functionality is implemented, **based on** the documentation, tests for the new functionality are written, **based on** the documentation, the release is assembled;

And then according to plan...

What's the difference with original process?

Unlike the "conventional" process, with the help of the version control system, participants see and know **from what state the system is moving** and **to what it should go**, and also ideally see **why the system is changing its state**. And the description of these changes is supported by the changes themselves right here. Which provides automatic "free" synchronization between the system and knowledge about it. 

For a new team member, the number of places to find an answer when a problem arises is reduced many times over. The amount of information that needs to be familiarized with in order to solve this problem is also reduced - because all information is distilled to **knowledge** about when, how and why changes occurred from system state A to state B.

"Free" is deliberately put in quotes and probably it should be spotted with many large question marks. Questions both to the idea itself conceptually and questions concerning the implementation of the idea in practice. Because, as we know, nothing is "free".

The advantages of the approach are listed - you could find more advantages and even more significant ones than those described above if it needed. But what are the disadvantages? 

What are the costs? And what will you have to pay?

Another mantra that often comes to mind is **TTM** (time to market). After all, *at the moment of developing some feature* it is necessary to additionally spend time on transforming the task context into **knowledge of the system** - into **documentation**.

However, the more the inconsistency between the current state of the system and **knowledge of it**, the more time will be spent by **everyone** who will need to make up for this difference in order to **solve the problem at the moment**. And after each cycle of changes of the system it will be more and more difficult to level out both the **qualitative** and **quantitative** difference between knowledge about the system and its current state. It can ultimately lead to **obscurity and unpredictability of the system** at all.

Thus, the time is spent in *now* will bear fruit in the long term. The company you work in does have long term, right?