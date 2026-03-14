A software artifact should be open for extension but closed for modification. Expensible without modifying the artifact.

Scenario
A webapp that displays a scrollable report. The new requirement is to add a functionality of printing the report in a particular fromat.
The flow is
````mermaid
graph TD;
A[Financial Data] --> B(Financial Analyzer);
    B --> C[Finanacial Report Data];
    C --> D(Web Reporter);
    C --> E(Print Reporter);

````
Financial Data > Financial Analyzer > Financial Report Data > Web Reported + Print Reporter
We can implement OCP by creating a heirarchy and utilize interfaces for inversion where needed.

![](../../../shared-resources/images/SolidPrinciples_OCP_Fig8.2.PNG)
