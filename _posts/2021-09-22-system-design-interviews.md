---
layout: post
title: System design interview expectations
---

- [Company's pet peeves](#companys-pet-peeves)
- [Interview prelude discussion](#interview-prelude-discussion)
- [Interviewer responsibility](#interviewer-responsibility)
- [Candidate responsibility](#candidate-responsibility)
- [Faq](#faq)
- [PEDAL framework](#pedal-framework)
- [Areas of improvement](#areas-of-improvement)
- [References](#references)

# Company's pet peeves

Know what kind of problems interviewers will have lot of expertise and experience in. They are likely to expect a higher standard of answers in that area and also likely to ask a question in that front. Example: Scale.AI => Workflows, Rubrik => Distributed storage, Distributed databases etc, Engine team => Concurrency, Kernel etc.

# Interview prelude discussion

Always start every interview with this discussion.

* Me: Are there any particular areas you want me to dive deep into?
  * Interviewer: No
    * Me: Ok, in that case please ask me to focus on a different area if you aren't getting the signals you want to evaluate me
  * Interviewer: Yes
    * Me: Ok, what is it?
* Me: I'll try to remember to give reasoning for my decision making. If I forget, please don't ding me. Ask me why and I will tell you

# Interviewer responsibility

**Scope, focus, assess**

1. Narrow scope
2. Ask candidate to focus on particular areas
3. Jump to different sub-problem if not getting enough signals for assessment
4. Assess if making right decision within scope of problem
5. Assess if making right tradeoff within scope of problem
6. Ding if solution is incomplete
7. Explain fully otherwise the interviewer will think you didn't know how. Too vague is risky (ref: Bengali guy https://www.pramp.com/session/join/24japY5mEvfwM3ElNWpB)

# Candidate responsibility

**Solve, tradeoff, communicate, fully completed work**

1. Follow standard template and communicate in that structure
   1. Do all steps (Example: Bengali)
2. Make best decision & tradeoff at every decision point within scope
3. They should never need to ask you "Why". Always explain your reasoning on your own.
4. Produce overall best design within scope
5. Dive deep in *all* areas interviewer wants you to dive deep into. Don't waste time on other things.
6. If the problem deserves pseudocode or data modeling, give it.
7. Deep dive into area of domain expertise unless you want to go elsewhere.
8. Do NOT compromise on design quality. This is not real-life where you need to make half-baked solutions due to lack of time. Do NOT even discuss those solutions.
9. If you are presenting a fully-custom solution, you MUST compare it with a solution that uses Redis/MySQL. There is a huge cost to deviating from the standard solutions out there on the internet and deviating from interviewer's knowledge and standard expectations. And you need to justify strongly via the Redis/MySQL solution does not work if you want to deviate from it. 

# Faq

**Q:** When should I ask the clarification? When it occurs to me or when it becomes relevant?

**A:** If you are unsure about relevance, it means you are unsure about scope. So, go back to clarify scope and ask the clarification question as part of it.

**Q:** Is it Ok to say you are worrying about a thing before you have proof that it is actually material to the discussion?

**A:** Within the scope, if it is a valid concern, I need to solve that concern

**Q:** Given a scope, is it Ok to reduce the scope and then incrementally increase the scope?

**A:** No. It gives a bad impression that you are weak. It also wastes your time.

**Q:** Do I need to tackle optional requirements?

**A:** Yes, to get hired as L7. They are giving it because they expect you to solve it at that level.

**Q:** If I am sensing a potential clarification but I'm not yet able to verbally say it, do I keep quiet, ignore or say it?

**A:** Keep quiet. You will just waste time rambling otherwise.

**Q:** Use a sharded database or use DDB?

**A:** You can defer the details if you can abstract the internals from the API/data-model. If not you will need to explain why you cannot use a sharded MySQL. Since all books use sharded MySQL, you will need to explain why deviating from it like discusssed in next Q&A.

**Q:** Does a top-down approach work better? If I am designing the synchronization service should I talk about the details of how the log will be implemented immediately? 

**A:** Yes, start high-level. See next question below.

**Q:** Is it possible to explain the overall design approach without delving into the details? For example, when designing Twitter timeline system, should I have described that we are going to have queues where I will enqueue the work on a write for it to be done later? Or should I describe a much higher level solution in terms of synchronization between the caches and the writes to the store and then talk about various tradeoffs of the various approaches I could take to get there?

**A:** Describing at a high level achieves the following:
1. Prove at high level it works.
2. Allow interviewer to poke holes at the high level solution itself
3. You can checkpoint progress
4. Convince yourself and interviewer that of all other high-level solutions possible, this is the best one. This is why. And all variations are within it
5. Raises level of abstraction that allows you to think of various alternative ways to flesh out this high level design.
6. Allows you to showcase multiple solutions to same sub-problem, while also allowing you to demonstrate that while all the variations are possible, the extent of ramifications of this are only to this sub-problem because we have agreed on the high-level design as being the best way to do it.
7. Show that you can think at an abstract level which demonstrates that you are capable of working at a Senior Staff engineer level which this level of abstraction is mandatory.

**Q:** How to engage in a conversation with interviewer? What should I ask him?

**A:** Following topics are allowed:
1. Scope clarification
   1. I understand that we need to support X, Y and Z. Is that correct? Did I miss anything?
   2. XYZ scenario can occur. Should I handle that?
2. Definition clarification
   1. What happens in this particular case in this system?
   2. Where does input come from? Where does output go?
   3. Is this valid input?
   4. Are there any constraints on how the input should be processed?
3. Scale clarification
   1. What scale do we want system to support?

Ask close-ended questions so that:
1. Answer is a quick yes or no.
2. By giving your suggestion and reasoning and asking yes/no, you demonstrate you have point of view, are not lazy and not cheating

**Q:** consistent hashing vs. BlockStore style or DDB style partitioning

**A:**

**Q:** Should I build from first principles or build on top of MySQL/Redis.

**A:** You may be able to build a very simplified custom stack solution that optimizes for your needs, but you need to remember that most of the other people do not have deep expertise building foundational systems and can only build "using" them like MySQL, Redis etc. Hence, you will find that others will think of solutions using (a) Queues (b) Redis (c) MySQL when in contrast you may think of a custom solution involving custom stateful components. This is why your solution for Rate limiting differs so much from what the rest of the internet thinks. The solution does not choose Token bucket because atomic operations don't exist in Redis for it. It does not chose to have buckets for doing sliding window algorithm because that is not possible in redis easily. The dropbox synchronization service would be a custom log solution but would be a partitioned MySQL streaming binlogs or a custom change table because most people don't know how to build components from first principles. It is difficult for you to convince them that your custom solution is better unless you compare-contrast it with the standard Redis/MySQL based solution. It is hence very important for you to deeply understand how to solve all the various design problems using Redis/MySQL so that you have the ability to tradeoff it with your custom approach. Lastly, because you are building a custom approach and it is simple in your head, you don't explain it. The other person however has no clue how to get so low level and implement this custom solution and are left wondering why you are being an idiot and not talking about the various interesting situations/solutions they have come up with because having to use Redis/MySQL

**Q**: Aurora transactions summary and tradeoffs

**A**: 

**Q:** Distributed file systems summary and tradeoffs

**A**

# PEDAL framework

1. **P**rocess requirements
   1. Functional requirements
   2. Non functional requirements (SLO) for availability, ACID, performance, accuracy, freshness
   3. Scale
2. **E**stimate
3. **D**esign the service
   1. Big picture problem solving
      1. Describe the crux of the approach
      2. Defend it. Analyze tradeoffs and defend why you are making it
      3. Express big picture as logical diagram
   2. Write the APIs
   3. Write the pseudocode
4. **A**rticulate the data model
   1. Table design
   2. What are the keys
5. **L**ist scaleable architecture
   1. Describe various alternatives for physical architecture of logical design
   2. Pick one high level architecture that meets functional and non-functional requirements
   3. Ask if need to dive deep on particular area
   4. Recurse. Repeat same procedure.

# Areas of improvement

- [x] Use estimation as guide for decisions
- [x] Ask the interviewer where they need me to focus. 
- [x] Answer interviewer concerns head-on quickly.
- [x] Have a theory/principle behind the algorithm first and think about it carefully to produce algorithm first. Incrementally solving it case by case will lead you astray. 
- [ ] Binary search variation practice
- [ ] Overall increase the speed
- [x] Don't linger around in an area more than you need to. If interviewer thinks you are taking too long in 1 area, he will ding you. You need to keep saying why are you still in there lingering.
- [x] Solve the problem at higher abstractions, expressing the crux of the problem and the solution principles.
- [ ] Performance on technical, working to go thru past experience.
- [ ] Actual role not being a fit, that utilized your expertise. Not as really focused on the storage.
- [ ] System design interview red flags
  - [ ] Misalignment in requirements with job logic being language neutral. The job logic flesh out was problematic
  - [ ] Interviewer did hint that they are interested in job logic, but I mixed that discussion with other things and had trouble finishing job logic
  - [ ] Would have preferred more structure
  - [ ] Solution was more generalized and more scaleable than what interviewer intended. A simplified solution would have been possible if the requirements around scale were clarified earlier
- [ ] Providing cultural fit - Do not intimidate them. Show them you are excited to solve problems you are facing.


# References

1. https://blog.pramp.com/how-to-succeed-in-a-system-design-interview-27b35de0df26
2. https://www.amazon.com/gp/product/B09559NJKL/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1



