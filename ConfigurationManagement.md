# Software Configuration Management

---
## Introduction to software configuration management

### What is Software configuration management

[wiki](https://en.wikipedia.org/wiki/Software_configuration_management)

In software engineering, *software configuration management (SCM)* is the task
of tracking and controlling changes in the software, part of the larger
cross-disciplinary field of *configuration management*.

SCM practices include *revision control* and the establishment of *baselines*.
If something goes wrong, SCM can determine what changed and who changed it.
If a configuration works well, SCM can determine how to replicate it across many
hosts.

### Purposes

The goals of SCM are:

* Configuration identification
* Configuration control
* Configuration status accounting
* Configuration auditing
* Build management
* Process management
* Environment management
* Defect tracking


---
## SMC Tool 1: Ansible

### Ansible use cases

#### Use Case 1: Configuration mangagement 
1. Write some kind of state description for the machine.
2. Use some tool to enforce that the machine is in that state:
    * the right packages are installed
    * configuration files contain the expected values and permissions
    * the right services are running
    * ...

These tools can be used for doing *deployment* as well.

#### Use Case 2: Deployment
*Deployment*: The Process of taking software that was written in-house, generating
binaries or static assets (if necessary), copying the required files to the 
machine(s), and then starting up the services.






