Resource - https://youtu.be/STKCRSUsyP0?si=SGVfyASI9Qm7JX0-
## Event vs Command
	Event - Hey something happened
	Command - Do something

Event - when one don't care about the response from the relying downstream service
Command - when one might care about the response from the relying downstream service

### Examples
	Event - address changed
	Command - change address

## Storytime
It all starts with decoupling and making system as minimally dependent to function correctly
![[EDA Example 1.png]]
Here, Customer Management has to be aware of the downstream Insurance Quoting service.

![[EDA Example 2.png]]
Here, Insurance Quoting is aware about the Customers required for its functionality.
But, still one of the service needs info about the other in this case Customers data is required by Insurance Quoting service and hence it must know about the Customer Management service to fetch required data.

**Introducing Events**
![[EDA Example 3.png]]
Now, managing events in queue or log can help add more services relying on the same event.

### Types of Patterns observed in EDA
	Event Notification
	Event Carried State Transfer
	Event Sourcing
	CQRS - Command Query Responsibility Segregation

#### Event Notification
Generic code notifies specific code by events for example GUI elements
Pro: 
- Decoupling
Contra:
- No statement of behavior
![[EDA Example 4 - EN.png]]
#### Event Carried State Transfer
Subscribers don't ask for additional information after an event occurred, all necessary state is given in the events. Subscribers copy whatever they need. 
Pro:
- reduced network traffic and frequent calls to service for required data and hence more decoupled.
- Better availability because of the copied/replicated data 
Contra: 
- Eventual Consistency
![[EDA Example 5 - ECST.png]]

#### Event Sourcing
Ability to rebuild the full state of the system by a persisted log of events.
Event Sourcing works with your data like version control systems work with your code 
Pro:
- Audit, Debugging and Historical time travelling
- "time traveling" like for debugging. use a copy of the system, feed it the events and see what happened 
- Alternative state - like branches in git, diff branches with a change and update
- Memory Image
Contra:
- unfamiliar - when I rebuild the state from the log I can't replay answers of external systems unless I record all answers as kind of events 
- old event schema still has to work somehow with new versions of code 
- IDs that are generated during replay are probably different than before 
- Asynchrony is difficult for people (but it's optional. You can synchronously use event sourcing) 
- Versioning (Snapshots can make that easier) 
- (Conclusion for me: Always remember that replay has to work for everything, otherwise there is no point in using event sourcing) 
 
Which events to record in the event store? 
Two events happen for a state change. One shows the intention (command), the other shows all the different changes that were done. Events that show intention often hold specific knowledge, the Event Store normally should stay generic (git only knows txt, not the used programming language syntax) It's important to think about which of those events are important to persist. Probably both.
#### Command Query Responsibility Segregation 
Separate components for updating the system and reading from the system 
Different views of the data are pretty common (e.g. a reporting database), 
the important thing here is that the command model is never used for reading from outside
![[EDA Example 6 - CQRS.png]]