# Components
Components are the untits of deployment of a system. In java these are jar files, in .NET DLL files and in compiled languages, aggrigation of binary files, in
interpreted languages aggrigation of source files. Components can be linked togerther into single executable or grouped together in single archive as .war file, or they can be deployed independently and loaded as dynamic plugins like .jar, .dll, .exe. History of this is interestingly written in Clean Architecture
book.

>Note: The middleware project at my first company used .jar files as single deployable compinents. E.g. JSON parser was a single .jar file that if needed any
change was modified and the jar file was replaced/deployed. The jar file was dynamically used by other jars which used import statements to use them. Better 
thing was the common jars were placed in EFS location that was shared among servers. So changing JSON parser and deploying its jar in EFS, changed it for every
project that was using jars from the shared location. So our team was responsible for supporting our middleware that worked by orchestraiting such utility jars
to run CRON pileplines and also deploy updated jars on EFS so other project can use them. The code for each utility was standardised and acted like a plug and
play component. Importing them and using their functions looked like we were using some library from MVN repository. While building them we were asked to keep
plug and play and standardization in mind. Some of such jars were Parsers (JSON, CSV, XML, Fixed Width), Writers (JSON, CSV, XML , Fixed Width), Connectors
(REST, Database, SFRP using Jsch). Apart from these we had adapter jars that orchestraited these utilities and glued them togeters, also they used configs from 
client specific databases to know which parser to use, which url to call, the connection creds, the fields to map to which database column etc. These adapters
were themselves treated as jars and acted like tasks. A common runner took a queue (sequence of tasks/adapters) and client name. It was itself a jar that took
these 2 args as input in run command. The client name was used to connect to specific client db and take up config from them to run adapters (dynamic) and also 
use the client schema to have "in" and "out" tables regarding each task to facilitate parsing into in table, processing into out table and writing from out
tables. These adapters and runner jars were specific to our middleware connector and the utilities were general and used by anyone. The runner even had config
to load jars placed at client specific location rather than general ones. This lead to us easily satisfing any client whiose requirements deveiated from the architecture at any task. All the configs were taken from config tabels from database of the client. (Conenctions, task_queues (runner configs and task queue), task_parameters (adapter configs and link to connection tables etc), Configuration (timezone etc of in and out systems)).
By this system onboadring of any new client was just to do configurations for these in database and was having either a GUI or CSV bulk uploads. Support teams helped in that part. No code change was needed, however custom logic specific to a client were also supported and configurable (overrided default jars)

This is called component plugin architecture and is now a default way of deployement. In our current project its a collection of such jars combined into war that
is deployed. The essence of a component is always independednt deployability. If changing them need no other change in any other jar then its an independent
component.

---

# Components Cohesion
