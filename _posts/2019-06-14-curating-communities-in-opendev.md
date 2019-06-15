---
layout: post
title: Curating Communities in Open Development
---
Some of the key characteristics of an [open development]({% post_url 2019-06-14-opendev-and-the-asf %}) project have to do with things that are kind of like checkbox items, like having a code repository that everyone in the company can see, or having a project web page.  Others are more nuanced and have to do with how the project behaves.  But just like the checkbox items, if you don't do them your project isn't really open, no matter what else you might be doing that makes it appear open.

One of these is a crucial element of any open development project.  Without it the project is not an "open development" project even if the code is freely visible.  It's this:  There must be time built into project planning to actually curate and manage the project.

This is best described with a counter-example.  Suppose you are working on Project A.  Elsewhere in the company, another team is working on Project B.  Project B has a dependency on Project A, and in particular there are certain capabilities Project B needs in Project A in order to deliver Project B.  Any dependency poses a risk, but the members of Project B notice that your project is an open development project, meaning they can see the code, check it out, and submit pull requests.  This makes the members of Project B a bit more relaxed.  Their success isn't wholly dependent on Project A - they are empowered to implement the changes they need themselves, if needed.

And, after a few weeks go by, this is what they do.  They needed a new API in your project that didn't exist, so someone from Project B implemented the new API and submitted a pull request to Project A.

And then nothing happens.

Why is this?  Simple:  Everyone in Project A is fully tasked with backlog items that are given higher priority.  Nobody in Project B has a say in the prioritization.  Everyone in Project A is so busy with their assigned work that nobody has the time to review a pull request and collaborate with this external contributor from Project B to eventually get this code into Project A.

If you work in software you can easily see how this can happen.  It's important to separate the net result from the details.  The net result is that outside contributors cannot actually contribute to Project A, even though it is an open development project by the letter of the law.  If a third party can't get their quality contribution considered for inclusion, your project isn't open - and saying it is open doesn't change the fact.

This phenomenon is not unique to open development projects, by the way.  Any true open source project can suffer the same effects if project members do not make the time available to manage their community.

So if you are a member of an open development project, I suggest there are a couple of things you should do.

First off, in order for the goals to actually be achieved you need to ensure that time is allotted in your planning for curation of the project itself - reviewing pull requests and working with outside contributors, answering questions about your project, improving the project webpage or online documentation, etc.  There are many ways this could be done; here are two:

* Decide as a team on some percentage of the team's time that will be set aside for this purpose.  For example, if it is 10% of your time, then if you have five engineers at 40 hours per week, for a total of 200 available hours, you know that 20 of those hours cannot be used in project planning because they are accepted overhead for managing the project.
* Assign someone each iteration to take on this task - it can be a rotating assignment.  That person would then have fewer assignments for the iteration with time set aside (one day per week?) to manage the project.

Second, you need to quantify your responsiveness window.  Decide how long is acceptable for a pull request to remain unaddressed or a question on a mailing list unanswered.  Make sure you operate within your guidelines.

Finally, you need to make it part of your definition of success.  For example, if you operate in two week sprints with a sprint review at the end, one part of the sprint review should be reviewing the requests made of your project from the outside - pull requests, questions on a mailing list, etc.  If you have external requests that have exceeded your responsiveness window or, even worse, are still unaddressed at sprint review, you should call that a failed sprint.  If you aren't willing to do that, it isn't actually important to you to be open, no matter what else you say.

One final thought:  Beware the anti-pattern of simply making your project open so anyone can contribute to the codebase directly.  It may be tempting to say "Well, let's just allow anyone to commit changes."  But that's not how open source projects work, and it shouldn't be how open development projects work either.  Requiring simple gates to maintain control of your project's quality, purpose, and vision is a good thing.  Besides, you aren't saving yourself any time doing this, anyway.  You might save a bit today, but when your project becomes unmaintainable due to all the uninformed changes you'll spend eons trying to sort it out.  Just build project curation into your plan and it will make a huge positive difference.
