# 📘 Chapter 1

# Thinking Aboout Data Systems

Data systems are not just databases. Message Queues and Databases do same thing od storing data for some time, but have completely different implementation
and use. But now many systems blur the line. Resdis is a datasource that can also act as message queues and Apache Kafka is a message queue with database 
level durability.

***"Applications are becoming demanding and no single tool can be used for everything. The application tasks are divided and best tools that can handle that
task well are used. These tools and orchestraited by application code. Eg. Application needs cache and database. And application code needs to orchestrate 
how these work together, update each other etc. 
If we contain this and provide an interface (API) to this system with abstractness, it is a full fleged datasystem with certain features that ensure
required caching based in a policy, reliability and robustness. This is what a data system engineer does."***

While building this datastore system many questions arise
- How doyou ensure that the data remains correct and complete, even when things go wrong internally? 
- How do you provide consistently good performance to clients, even when parts of your system are degraded? 
- How do you scale to handle an increase in load?
- What does a good API for the service look like?

The major concern while building this system can be grouped into three needs. 
- 🛡 Reliability: The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or soft‐
ware faults, and even human error)
- 📈 Scalability: As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth
- 🧹 Maintainability: Maintaining current behavior and adapting the system to new use cases

## Reliability

## Scalability

## Maintainability
