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
Continuing to work correctly, even when things go wrong.
Functioning as expected, can tolerate user making mistakes or using it in unexpected ways, performance justified under load, prevents unauthorized use and is secure.
Things that go wrong are called faults and systems that anticipate them before and provide capabilities to handle them are resiliant/fault-tolerant.
Faults are different from failures. Faults are components going off the required spec sometimes leading to system failure and sometimes unreliability.
One can test his system by creating deleberate chaos. Terminating a system suddenly and checking error handling. Netflix Chaos Monkey is an example of such thing in Netflix. Most of the bugs are poor error handling, that dont break the system but work unexpected sometimes.

### ⚙️ Hardware Faults
- Hard Disk Crash
- Faulty RAM
- Powergrid Blackout
- Unplugging wrong cable

***Hard Disk have a mean time to failure (MTTF) of about 10 to 50 years. On a storage cluster of 10000 disks, an average of 1 disk failure can be expected each day.***
```
Failure Rate of 1 disk = 1 / MTTF
Failure Rate of system = N / MTTF  (N = Number of disks)

- In 3650 to 18250 days → 1 hard disk fails
- Disk failure rate of 1 disk = 1/3650 to 1/18250
- Disk failure rate of system with 10000 disks = 10000/3650 to 10000/18250

Worst case:
10000/3650 = 2.74 disk failures per day (10 year MTTF)

Best case:
10000/18250 = 0.55 disk failures per day (50 year MTTF)

Average:
(2.74 + 0.55) / 2 = 1.65 disk failures per day  
in a system of 10000 hard disks with MTTF 10–50 years
```
If this is not anticipated from started and fault tolerated, every day somewhere the sytem would fail.

***To mitigate hardware faults: RAID + Dual power supply to servers + Hot Swapable CPUs + Power backups.
But as the application scales, its hard to mitigate hardware faults this way
Recently, redundance is only needed. If a system fails and we can restore a backup to another system fast. Even AWS prioritizes flexibility and 
elasticity over single system fault tolerence. EC2s sometime fail without warning. So EC2s are not perfectly fault tolerated, instead another redundant system is set up, just in reasonable time with the backup. This is hardware redundancy. It provides another benefit of each node being
able to be patched so that the application dosent have to worry about downtime for maintainance/release. (Rolling Upgrade)
However the fault tolerence at every system is also taken care off using software. But majorly its redundancy that helps hardware faults.***

### 🧩 Software Errors

## Scalability

## Maintainability
