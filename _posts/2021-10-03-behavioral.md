---
layout: post
title: Behavioral questions
---

*general*

1. Why this position? *
2. Walk me through your resume *
3. Why are you transitioning from your current position? *
4. What makes a good [job title] / bad [job title]?
5. Tell me what others would say about you
6. Tell me about what do you want to do in the future
7. What are your strengths and weaknesses?
8. How do you handle tradeoffs
9. How do you balance engineering limitations with customer requirements
10. How do you sell an idea to senior management

*working style*
1.  Who else did you work with when you were doing X?

*accomplishments*
1. Tell me about the greatest accomplishment of your career

*general failure & weaknesses*
7. Tell me about a time you failed and what you learned from it
8.  Tell me about the area where you have the most to learn
12. Tell me about a time you were given feedback that was constructive
21. Tell me about a recent / favorite project and some of the difficulties you had
22. Tell me about a time you struggled on one of your software projects
5. Tell me about a time you were wrong

*TL*
1.  Tell me about a time you lead a team
2. Tell me a time you brought up a transformative change

*Teamwork*
1.  Tell me about a time you had to step up and take responsibility for others

*Boss fit*
18. Tell me about your worst boss and why they were bad
4. Tell me about a time your manager strongly believed in something and you did not. How did you manage the situation.

*Tradeoffs*
1. Tell me about a time you had to make a decision to make short-term sacrifices for long-term gains

## Stories

## A

1. Managing conflict
   1. Tell me about a time when you had a conflict with a peer engineer.
   2. Tell me about a time when you had a conflict with a junior engineer.
   3. Tell me about a time when you had a conflict with a manager.
   4. Tell me about a time when you had a conflict with a cross-function team.
   5. Tell me about a time you had to resolve conflict in the team.
2. Evangelizing, Proposing initiatives, Think Big
   1. Tell me about a time you proposed an initiative but it didn't pan out
   2. Tell me about a time you proposed an initiative and it was successful
   3. How would you advocate for a commitment to a priority, when that priority is not high on someone else's list?
3. Cross function situations
   1. Stories
      1. LB issue with SRE
      2. Should CSD pod capacity be part of system quota or ABS quota
   2.  Tell me about a time you worked with cross-functional teams and the role you played
   3.  Have you ever collaborated with multiple teams? What challenges did you face?
   4.  How would you manage timelines in a highly matrixed environment, where there is no top down authority?
4. Mentoring
   1. How do you develop a relationship with TLs/Staff engineers?
   2. How do you grow a TL/Staff engineer?
   3. How do you grow the interpersonal skills of a Staff engineer?
5. Think clearly
   1. Tell me about a time when you had to make a difficult decision. Specifically, what considerations were involved and how did you decide which ones to focus on?
   2. Tell me about a time data conflicted with intuition about an importnant decision. What did you do
      1. Impact of async code in replication code was misjudged. Intuition was wrong. Lesson learnt -- It's easy to make a intuitive decision about an important question when you are making lots of decisions. Keep notes of important questions and use that to track if you used intuition for any. If so, ask if you can do a quick PoC style test to assert the intuition.
      2. R-tree approach vs. practice of columnar. Lesson learnt -- R-Tree decision was made before I joined the team. But I learnt, when deviating from the norm be doubly cautious.
   3. Tell me about a time you made a mistake because you weren’t thinking clearly
      1. Impact of async code in replication code was misjudged. Intuition was wrong. Lesson learnt -- It's easy to make a intuitive decision about an important question when you are making lots of decisions. Keep notes of important questions and use that to track if you used intuition for any. If so, ask if you can do a quick PoC style test to assert the intuition.
      2. CA Authority decision. Lesson learnt - Precision questioning and don't take things at face value.
   4. Describe a time when you observed someone else on the team struggling to find clarity on a project. What did you do?
      1. 
6. Focus and simplify
   1. Tell me about a time when you had to say no to something you really wanted to do
      1. Delegate the overseeing of the performance improvement aspects to V for the replication, iop improvement work because I had to focus on scale analysis, intrinsic metadata project etc. and I didn't have cycles to fully oversee that too. We split up in terms of I will focus on correctness aspects and he can cover the integration and performance aspects
   2. Tell me about a time when the complexity of a task kept you from being successful. Thinking back now, what would you have done differently?
      1. AIDB project failure -- There was focus, failed because I didn't simplify enough. It was too ambitious. Undertanding cross-org prioritites better. Understanding PR aspects better sooner and sensing the moment better would have helped for pushing such a large initiative. I pick this example because it is beyond scope of expectations of applied role and mistake here is non-costly
   3. Describe a project or initiative that you worked on or managed that was jeopardized due to lack of focus
      1. 
   4. Tell me about a time when you were able to achieve your goals by simplifying a task
      1. Intrinsic punt some more migrations to next milestone to achieve current date
      2. Milestone 1 goal was reduced to reading back from disk before sending it to both followers. Required an extra read. But it allowed the entire system to be integrated and well-tested before we made this optimization.
7. Sense the moment
    1. Tell me about a situation at work in which you pushed too hard at the wrong time, and failed as a result
    2. Tell me about a story from your experience where timing something appropriately allowed you to be successful
    3. Tell me about a time when someone delivered bad news to you at an inopportune time and how you reacted to that
    4. Tell me about a time when you had to caution someone about taking a next step because you felt the timing wasn't right. What was the result?
8.  See around corners
    1.  Describe a time when you succeeded because you accurately anticipated what would happen in the future
    2.  Tell me about a time when you help on too tightly to a way of doing things when you should changed
    3.  Describe a time when you have intuition about the future could have saved you time, money or resources
    4.  What practices do you employ as a manager to keep your team focused on what’s coming tomorrow, in addition to what’s happening today?
9.  Innovate down to the details
    1.  Tell me about the last time you had to get deep into the details to change an important element of the project. How did you do it?
    2.  Describe a time when you were not successful due to your inability to understand details of the project
    3.  Tell me about a time when you completely rethought a small component of a project that made a big difference to the whole
    4.  Describe a small detail in a product or service that use frequently and that makes all the difference to you
10. Demand difference
    1.  Tell me about a time when your ability to push people to think differently resulted in a better outcome. How did you do it?
    2.  Tell me about a time when you had to settle for the standard way of doing something when you had been pushing for a better alternative
    3.  Tell me about a time when you proposed a new, potentially risky solution and it was implemented
    4.  Tell me about a time when you challenged the status quo. How did people react?
11. Approach problems flexibly
    1.  Tell me about a time when you have had to modify your approach one or more times to a problem at work
    2.  Tell me about a time when you were not flexible in approaching a problem, and failed as a result
    3.  Tell me about a time when you arrived at a solution in a way you did not expect. What happened?
    4.  Tell me about a time when you’ve managed or worked with soeone who was unwilling to change their way of doing things
12. Fight for excellence
    1.  Tell me about the last time you had to fight to get great results
    2.  Describe to me the steps you take to ensure high quality in your work. Give me an example from your experience
    3.  Describe an experience where your quest for excellence taught you something that you remember to this day
    4.  Tell me about a time when you had to push your team members further than they expected in order to achieve excellent results
13. Drive what matters
    1.  Tell me about the last time you had to adjust priorities to align with the organization
    2.  Tell me about a time when you were successful in convincing someone else to change priorities to accomplish a higher priority
    3.  Tell me about a time you had to stay no to something that was important to someone else in order to accomplish your own goals
    4.  Tell me about a time when you had to shift a colleague’s priority to something that mattered more to the company. Did you convey your message? What was the outcome?
14. Cut through ambiguity
    1.  When was the last time you felt bogged down by lack of clarity? Tell me about it and what you did, if anything to respond.
    2.  Tell me about a time you were faced with a complex problem and didn’t know where to begin. What process did you rely upon to gain clarity and direction?
    3.  Tell me about a time when you saw a project in jeopardy due to conflicting data
    4.  Tell me about a time when you guided someone through a project where you could not provide many answers or details.
15. Own the hard calls
    1.  Tell me about a time when you had to make a difficult call on a project. What was the result?
    2.  Describe a situation in which others disagreed with a decision that was made. How did you respond?
    3.  Tell me about a time when you had to make a tough decision about an employee situation. What did you do and what was the outcome?
    4.  Tell me about a time you defended an unpopular decision you had made. What did you do?
16. Listen, challenge and commit
    1.  Tell me about a time when you had a hard time listening to someone else. What did you do?
    2.  Tell me about an experience where you had to go along with a decision you didn't agree with at first. How did you deal with it?
    3.  When was the last time you challenged a decision at work that directly affected you or your team? Tell me about the challenge and the outcome.
    4.  Tell me about a time when you had to convince others to agree with a direction that they didn’t like. How did you get them to commit?
17. Know people
    1.  Tell me how you establish relationships at work. Can you describe a specific example when you did so successfully?
    2.  How have you created a large network at work? How do you help others to do so?
    3.  Tell me a story about a time wen your instincts about a work colleague failed you. What did you learn from it?
    4.  Give me an example of a time when a team member had difficulty establishing a relationship that was critical to their project?
18. Foster trust
    1.  Tell me how you create a trusting environment at work and provide an example of when you did so successfully
    2.  Tell me about a time when you made a committment at work that you could not keep. What were the consequences?
    3.  Tell me about the last time you trusted someone at work and were disappointed. What did you learn?
    4.  Tell me about a time when you did something that broke someone’s trust in you. How did you repair that trust?
    
## References

1. [IGotAnOffer behavioral questions list](https://igotanoffer.com/blogs/product-manager/behavioral-interview-questions-tech-companies)
2. [Exponent How to answer XFN influence questions](https://www.tryexponent.com/courses/cross-functional-pmm/how-to-answer-cross-functional)
3. [Story bank](https://www.tryexponent.com/courses/behavioral/behavioral-interviews-creating-story-bank)
4. [Team alignment](https://pulseasync.com/operators/remote-team-alignment/)