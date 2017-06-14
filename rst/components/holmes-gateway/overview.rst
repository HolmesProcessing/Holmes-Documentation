Overview
**************
.. _Totem: https://github.com/HolmesProcessing/Holmes-Totem
.. _Totem_Dynamic: https://github.com/HolmesProcessing/Holmes-Totem-Dynamic

Holmes-Gateway orchestrates the submission of objects and tasks to HolmesProcessing. Foremost, this greatly simplifies the tasking and enables the ability to automatically route tasks to Holmes-Totem and Holmes-Totem-Dynamic at a Service level. In addition, Holmes-Gateway provides validation and authentication. Finally, Holmes-Gateway provides the technical foundation for collaboration between organizations.

Holmes-Gateway is meant to prevent a user from directly connecting to Holmes-Storage or RabbitMQ. Instead, tasking-requests and object upload pass through Holmes-Gateway, which performs validity checking, enforces ACL, and forwards the requests.

If the user wants to upload samples, he sends the request to /samples/ along with his credentials, and the request will be forwarded to storage.

If the user wants to task the system, he sends the request to /task/ along with his credentials. Holmes-Gateway will parse the submitted tasks and find partnering organizations (or the user's own organization) which have access to the sources that are specified by the tasks. It will then forward the tasking-requests to the corresponding Gateways and these will check the task and forward it to their AMQP queues. The Gateway can be configured to push different services into different queues.

This way Gateway can push long-lasting tasks (usually those that perform dynamic analysis) into different queues than quick tasks and thus distribute those tasks among different instances of `Holmes-Totem <Totem_>`_ and `Holmes-Totem-Dynamic <Totem_Dynamic_>`_.

Highlights
""""""""""""""
- Collaborative Tasking: Holmes-Gateway allows organizations to enable other organizations to execute analysis-tasks on their samples without actually giving them access to these samples.


- ACL enforcement: Users who want to submit tasks or new objects need to authenticate before they can do so.


- The Central point for tasking and sample upload: Without Holmes-Gateway, a user who wants to task the system needs access to RabbitMQ, while a user who wants to upload samples needs access to Holmes-Storage.
