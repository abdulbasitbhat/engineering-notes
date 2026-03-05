# Technical Design Documents

When assigned a task, there is an urge to get started right away. Do the first step. Code. But its not a good practice. I used to do this during my
internship and even as a fresher. It does have drawbacks. Let me explain some scenarios where I understood the importance of these Documents.

---
I was assigned a feature and provided the Functional Design for it. Excited to start, I jumped straight onto coding. Minimal thinking and making 
decisions on the way using code. Its deadline was preponed and set as priority by the business. While my work was still under process, I was the 
sole owner of the task now. If I felt sick or had any genuine reason to not be able to work, it would directly affect business. At this point I 
rushed and tried to complete the task. While I was about to raise PR, the Functional Design got changed. And this is no ones fault, its the nature of
business. Now while what I had built needed changes, no one else in the team could deliver in parallel with me. So assiging an extra resource would
be less productive and even slow me down. 
Somehow, I delivered withing enough time to test and it went on to production after a few bug fixes. 

## How could Technical Design Document help in this.
The solution to any task needs to be studied in terms of system rather than code. And thinking while coding is actually multitasking. 
If I had spent a day or two on TDD and had got it approvred, in times of unavoidable conditions like prepone of deadlines or redesigns, extra 
resources can be assigend and shared with the context. Both FDD and TDD would be enough to start working on the implementation. But without TDD,
the owner holds context, which become a problem in critical deliveries.

---

I worked in a team developing a system from scratch. Everyone had something on their list. We worked in parallel and delivered fast. But as we had no
proper document to explain our work, we were holding context subconsciously. New joiner needed help for each and every block of code, as they had no
other way to understand it. We had made it difficult for both us and them. But as soon as we had a prototype ready with 80% of the features, our lead
asked us to focus on design documentation. 
We started documenting and eventually found out that the system was now getting easier to explain even by sharing documents. We were not holding context
now. And this led us to relax and even take leaves when needed. Before all this, absence led to that feature not progressing anywhere as we held context.
At the end of this project, the handover to the run team was the most relaxing part of my career.

---
A formula I made to remind me of the importanc of TDD is
- One day of design/TDD work can save a month of development time by making design the core and making things like resource allocation easy

---
Learning
- Start with TDD and get it approved. Design in terms of system, not code blocks
- Keep TDD updated and prevent holding context.
- Make the system easy to understand and maintain.
- TDD makes resource allocation easier and faster.


