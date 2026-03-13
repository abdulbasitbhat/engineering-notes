### SRP does not mean ***"Module must have one and only one reason to change"***, instead a more fitting definition is ***"Module must be responsible to one and only one user/stakeholder"***. 
So if a module needs to change the change must be requested from the stakeholder and the changes should effect the stakeholder only.

### SRP Violation Symptoms
- **Symptom 1: Accidental Duplication**  
Creating a function "calculateDate()" may be following SRP easily as it is only calculating date, single responsibility. But imagine if 2 stakeholders
say Finance team and HR team depend on it. And finance team request to change the return format to MM/DD/YYYY from DD/MM/YYYY. On making the changes
the Finannce Team validates it and code is deployed. Everything seems correct but HR team gets the wrong date now, and this will go unnoticed untill a
big issue appears. This is actually the first symptom of violating SRP. Imaging instead of date, currency was to be changed.
This system is now comupled. Any change needs discussions with both stakeholders and eventually a failure. 

>Note: During my second project, I worked on a project that exactly violated SRP using this. It has multiple screens used by different teams and the
code is tightly coupled. So changing anything required analysis for days and a constanct check of finding whare the code is getting used before changing 
anything. Also while testing, sometimes in some feature implementations, the whole system need to be tested. But the UAT testes the required test cases,
they dont know the coupled code and eventually are unaware. Eventually its passes UAT and its deployed. Next is production bugs in other untouched places
that are unexplainable and only explaination is "It was working fine before this feature was deployed". Working on this system is actually scary and freshers
are bound to make career ending mistakes without their fault. While my first project taught me what to do, how to build, this taught me what not to do.
Addition of any minor feature which appears easy, eventually becomes a nightmare and the timeline is unexplainable to business.
