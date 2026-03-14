### SRP does not mean ***"Module must have one and only one reason to change"***, instead a more fitting definition is ***"Module must be responsible to one and only one user/stakeholder"***. 
So if a module needs to change the change must be requested from the stakeholder and the changes should effect the stakeholder only.

### SRP Violation Symptoms
- **Symptom 1: Accidental Duplication**  
Creating a function "calculateDate()" may be following SRP easily as it is only calculating date, single responsibility. But imagine if 2 stakeholders
say Finance team and HR team depend on it. And finance team request to change the return format to MM/DD/YYYY from DD/MM/YYYY. So the developer creates another function and adds it formatDate and calls it in calculatedDate. On making the changes the Finannce Team validates it and code is deployed. Everything seems correct but HR team gets the wrong date now, and this will go unnoticed untill a big issue appears. This is actually the first symptom of violating SRP. Imaging instead of date, currency was to be changed. This system is now comupled. Any change needs discussions with both stakeholders and eventually a failure. So the HR and Finance team functionalities used calculateDate and since Employee class where all other functions are also present its difficult to know the stakeholders of that code block.

>Note: During my second project, I worked on a project that exactly violated SRP using this. It has multiple screens used by different teams and the
code is tightly coupled. So changing anything required analysis for days and a constanct check of finding whare the code is getting used before changing 
anything. Also while testing, sometimes in some feature implementations, the whole system need to be tested. But the UAT testes the required test cases,
they dont know the coupled code and eventually are unaware. Eventually its passes UAT and its deployed. Next is production bugs in other untouched places
that are unexplainable and only explaination is "It was working fine before this feature was deployed". Working on this system is actually scary and freshers
are bound to make career ending mistakes without their fault. While my first project taught me what to do, how to build, this taught me what not to do.
Addition of any minor feature which appears easy, eventually becomes a nightmare and the timeline is unexplainable to business.

- **Symptom 2: Merges**  
If two developers from different teams check out a particular class and change it, merges may appear. Even if the tasks are completely unrelated.

### Solution
***Move the functions into differnet classes in such a way that each class has only one reason to be changed. 
Eg. If previously Employee class had calculatePay, reportHours, saveEmployee functions. Without split if a change was needed for claculatePay by one and reportHours by other then still the point of change will be Employee class
Solution is to split the functions into classes PayCalculator (calculatePay), HourReporter(reportHours), EmployeeSaver (saveEmployee). So any change is isolated. Changes are made and the team is informed about the updates and all stakeholders are clear on what was changed.
These splitted classes can have a Facade class that has structure as original un splitted class but the functions point to their implementation in respective classes. Simply said this limited the area of impact. Redusced merges and saved developer from making mistakes.
So in above case if we had to change the data format and use our updated classes, the point of change was easily identifiable, stakeholders were identifiable and if any change or handling was needed separately, it could be done easily. And even the business can be explained about the impact of the change requirements, the stakeholders. Also as in version control we push files rather than code blocks its easier and leads to less merge conflicts and even an easy task distribution withing team***


SRP means to separate blocks in such a way that any changes, its effects and stakeholders are identifiable, explainable and have a boundary of effect which can be easily analysed and tested. 

> For simplification i have not added the images here as they are present in book Clean Architecture







