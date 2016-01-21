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

Ansible is a framework for describing, deploying and managing configurations.
* It has a domain specific programming languages for execution flow (loops, conditionals)
* Its own set of (primitive) modeling concepts: Plays, Playbooks, Modules, Tasks, Roles
* It embedds YAML to describe these entities

### Introduction to Ansible

#### Ansible use cases

##### Use Case 1: Configuration mangagement 
1. Write some kind of state description for the machine.
2. Use some tool to enforce that the machine is in that state:
    * the right packages are installed
    * configuration files contain the expected values and permissions
    * the right services are running
    * ...


##### Use Case 2: Deployment

Ansible (and some other SCM tools) can be used for doing *deployment* as well.

*Deployment*: The Process of taking software that was written in-house, 
generating binaries or static assets (if necessary), copying the required files 
to the machine(s), and then starting up the services.

#### How Ansible works

A configuration process in Ansible:
1. Playbook:
    The state ist described in Ansible with yaml scripts called *playbooks*:
    1. A playbook describes *which hosts (remote servers) to configure*
    2. An ordered list of *tasks* to perform on these hosts
2. Execute playbook using the `ansible-playbook` command. Example:

        ansible-playbook webservers.yml

    * Ansible will make ssh connections in parallel to the machines specified in
      the `*.yml` file. Then it will execute the tasks on the list.
      This works by:
      1. Ansible generating a Python script for the task
      2. Copy the script to the target machines
      3. Execute the srcipt
      4. Wait for the script to complete execution on all target machines
      5. Ansible will then move to the next task on the list

Important:
* Ansible runs each task in parallel across all hosts
* Ansible waits until all hosts have completed a task before moving to the next 
  one
* Ansible runs the tasks in the specified order

#### Pros of Ansible

Advantages of Ansible:
* Ansible has easy-to-read YAML syntax: executable documentation
* Nothing to install on the remote hosts (only ssh, Python 2.5)
* Push-based:
   1. You: Make a change to a playbook
   2. You: Run the new playbook
   3. Ansible: Connect to the machines, execute the modules
* Ansible scales down: Painless even for one-machine-usage.
* Built-in modules: 
   * Modules are like a (standard) library of Ansible -
     they can be used to install a package, restart a service, copy a 
     configuration file, ...
*  * Ansible modules are declarative: the state is described that a server should
     be in.
*  * Ansible Modules are *idempotent*. If a task has already been executed 
     Ansible won't do it again (like `make`).
     This means it's safe to run Ansible multiple times on the same server.
* Thin layer of abstraction; doesn't abstract away details of the operating 
  system. All the known abstractions can be used.


#### Unit of reuse in Ansible

The unit of reuse in Ansible is the module.
The scope of a module is small and can be operating system specific,
it's easy to implement well-defined, shareable modules.

Ansible playbooks aren't really intended to be reused across different contexts.
Roles: A way of collection playbooks together so they can potentially be reused.


##### ansible.cfg file

The `andible.cfg` file specifies 
* the location of the inventory file (`hostfile`)
* the user to ssh as (`remote_user`)
* the ssh private key (`private_key_file`)

Example:

    [defaults]
    hostfile = inventory
    remote_user = vagrant
    private_key_file =/Users/<username>/.vagrant.d/insecure_private_key


#### Ansible and version control

Ansible uses `/etc/ansible/hosts` as the defualt locations for the inventory 
file. If the inventory files should be under version control it is better to 
place it elsewhere (alongside the playbooks).

It is strongly recommended to use a verion system like git to maintain the
playbooks.


### Central Component 1: Playbooks (Ansibe "Programs")

ressources:
* [Playbooks docu](https://docs.ansible.com/ansible/playbooks.html)
* [Example Playbooks](https://github.com/ansible/ansible-examples)

*Playbook* is the term that Anisble uses for a configuration management script.
It is the device used for:
* configuration
* deployment
* orchestration

Ansible modules: tools; Playbooks: design plans.

A playbook is an ordered list of one or multiple *plays*;
A play maps a group of hosts to compound execution units: *roles*.
A role consits of several atomic execution units: *tasks*.

By composing a Playbook of multiple plays it is possible 
to orchestrate multi-machine deployments.

##### Example of a simple playbook

Simple playbook where a host is configured to run a Nginx web server:

    ---
    - name: Configure a webserver with nginx
      hosts: webservers
      sudo: True
      tasks:
        - name: install nginx
          apt: name=nginx update_cache=yes
        - name: copy nginx config file
          copy: src=files/nginx.conf dest=/etc/nginx/sitex-availabe/default
        - name: enable configuration
          file: >
            dest=/etc/nginx/sites-enabled/default
            src=/nginx/sites-available/default
            state=link
        - name: copy index.html
          copy: src=files/index.html dest=/usr/share/nginx/html/index.html mode=0644
        - name: restart nginx
          service: name=nginx state=restartd


#### Basics

##### Hosts and Users

For each play in the playbook it has to be specified:
* Which machine in the infrastructure to target.
* What remote user to complete the tasks.
    * global for the whole play:

        ---
        - hosts: webservers
          remote_user: root

    * local for one task:

        tasks:
           - name: test connection
             ping:
             remote_user: foo

    * local with sudo rights

        tasks:
           - name: test connection
             ping:
             remote_user: foo
             become: yes
             become_method: sudo

##### Tasks

Each play contains a list of tasks.
Tasks are executed in order, one at a time, against all machines matched by the
host pattern, before moving on to the next task.

Goal of each task: Execute a module with specific arguments.


##### The `command` and `shell` modules 

The `command` and `shell` modules are the only modules that juest take a list of 
arguments and don't use the `key=value` form.

    tasks:
      - name: disble selinux
        command: /sbin/setenforce 0

The `command` and `shell` modules care about return codes.
If they exit with a failure the whole play for that machine is aborted
(or use `ignore_errors`).


#### Playbook Roles and Include Statements

##### Code reuse

Code can be included in Playbooks:
* including task files: task includes pull in tasks from other files.
* include plays from other playbook files.

Philosophie behind this:
When you start thinking about it -- tasks, handlers, variables, ... --
begin to form larger concepts.
You start to think about about modeling what something is,
rather than how to make something look like something.
(Kind of duck-typing for machines:
* A machine is of type webserver if it has a webstack installed
* A machine is a Haskell Development if it has a Haskell stack and vim installed
* A machine is a SMT solver if it has a SMT solver installed
* A machine is an email client if it has mutt, ... installed)
It's no longer apply this handful of THINGS to these hosts", you say
"these hosts are dbservers" or "these hosts are webservers".
(Polymorphicity of machines).

*Roles* in Ansible build on the idea, of including files and combine them to 
form clean, reusable abstractions - they allow you to focus on the big picture
and dive only into the details when needed.

We'll start with includes but roles are the better abstraction and should be 
used in every playbook.

Roles are units of organization in Ansible.

##### Task include files

Include files enable the reuse of task lists between plays and playbooks.

Example:

A task include file contains a flat list of tasks:

    ---
    # possibly saved as tasks/foo.yml
    
    - name: placeholder foo
      command: /bin/foo
    
    - name: placeholder bar
      command: /bin/bar


Include directives can be mixed with regular tasks in a playbook.

##### Parameterized include files
Parameterized includes: Includes that accept variables

    # standard syntax
    tasks:
      - include: wordpress.yml wp_user=timmy
      - include: wordpress.yml wp_user=alice
      - include: wordpress.yml wp_user=bob

    # modern syntax (from version 1.0 up) that allows structured variables:

    tasks:
      - include: wordpress.yml
        vars:
          wp_user: timmy
          ssh_keys:
            - keys/one.txt
            - keys/two.txt

Variables in files can be referred to like this:

    {{ wp_user }}

(In addition to the passed in parameters, all variables from the vars section
are also available in the include files)


##### Playbooks in Playbooks

Includes can be used to import one playbook into another.
This allows to define a top-level playbook that is composed of other playbooks.

Example:

    - name: this is a play at the top level of a file
      hosts: all
      remote_user: root
    
      tasks:
    
      - name: say hi
        tags: foo
        shell: echo "hi..."
    
    - include: load_balancers.yml
    - include: webservers.yml
    - include: dbservers.yml


#### Roles

The best way to organize playbooks: Roles!

Roles are ways of automatically loading 
* vars_files
* handlers
* tasks
based on a known file structure.

Roles are automation around `include` directives.
They improve includes by improvements to search path handling for referenced 
files.

Example project structure:

    site.yml
    webservers.yml
    fooservers.yml
    roles/
       common/
         files/
         templates/
         tasks/
         handlers/
         vars/
         defaults/
         meta/
       webservers/
         files/
         templates/
         tasks/
         handlers/
         vars/
         defaults/
         meta/

In a playbook, it would look like this:

    ---
    - hosts: webservers
      roles:
        - common
        - webservers

This designates the following behaviours, for each role `x`:

* If roles/x/tasks/main.yml exists, tasks listed therein will be added to the play
* If roles/x/handlers/main.yml exists, handlers listed therin will be added to the play
* if roles/x/vars/main.yml exists, variables listed therin will be added to the play
* if roles/x/meta/main.yml exists, anu role dependencies listed will be added to the list or roles (1.3 and later)
* Any copy, script, template or include tasks (in the role) can reference files in roles/x/files/ without 
  having to path them relatively or absolutely

Important:
In Ansible 1.4 and later you can configure a `roles_path` to search for roles.
Use this to check out all your commen roles to one location
and share them between multiple playbook projects.
(See Configuration file for details to set this up in `ansible.cfg`)

##### Parameterized Roles

Roles can be parameterized:

    ---
    - hosts: webservers
      roles:
        - common
        - { role: foo_app_instance, dir: '/opt/a',  port: 5000 }
        - { role: foo_app_instance, dir: '/opt/b',  port: 5001 }

##### Role Default Variables

Role default variables enable setting default variables for included or 
dependent roles. To create defaults add a `defaults/main.yml` file to the role
directory. These variables will have the lowest priority of any variables 
available, and can be overwritten.

##### Role dependencies

Role dependencies enable automatically pulling in other roles when using a role.
Role dependencies are stored in the `meta/main.yml` file in the role directory.

Example:
    
      dependencies:
    - { role: common, some_parameter: 3 }
    - { role: apache, port: 80 }
    - { role: postgres, dbname: blarg, other_parameter: 12 }

Role dependencies can also be installed from files or source control repos:

    ---
    dependencies:
       - { role: '/path/to/common/roles/foo', x: 1 }


Further properties of role dependencies:
* Role dependencies are always executed before the role that includes them
* Role dependencies are recursive
* By default, Role dependencies can also only be added as a dependency once 
  - if another dependency lists the same role it will not be run again
     (can be overwritten).





#### Truth values in playbooks

Module arguments (like `name=yes`) are treated differently from values elsewhere
in the playbook. 
Elsewhere values are handled by the YAML parser and so use the YAML conventions
for truthiness:

* YAML truthy: `true`,`True`,`TRUE`,`yes`,`Yes`,`YES`,`on`,`On`,`ON`,`y,`Y`
* YAML falsy: `false`,`False`,`FALSE`,`no`,`No`,`NO`,`off`,`Off`,`OFF`,`n`,`N`

Module arguments are passed as strings and use Ansible's internal conventions:
* module arg truthy: `yes`,`on`,`1`,`true`
* module arg falsy: `no`,`off`,`0`,`false`

Typical: `yes` and `no`.


#### Ansible conventions

Ansible conventions: 
* files are kept in a subdirectory `files`.
* Jinja2 templates are kept in a subdirectory called `templates`.


#### Target machines

Creating target machine groups

Groups declared in the inventory file can be refered to in the playbook.

Inventory files are in INI file format. Example:

    [webservers]
    testserver ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222


#### Running the playbook

The `ansible-playbook` command executes playbooks.

    ansible-playbook <playbookname>.yml

When Ansible starts executung a play, the first thing it does is collect 
information about the server it is connection to:
* Operation system
* hostname
* IP addresses of all interfaces
* MAC addresses of all interfaces
* ...

This information can later be used in the playbook.

#### Structure of a Playbook

A playbook is a list of dictionaries (a playbook is a list of *plays*).
Plays connect hosts to tasks.

Mandatory settings:
* A set of hosts to configure
* A list of tasks to be executed on those hosts

Optional settings:
* `name`: a description what the play is about.
* `sudo`: Ansible will run every task as root (by sudoing).
    Useful for Ubuntu.
* `vars`: variables and values.

Example 1: simple task

    ---
    - name: Play1
      sudo: True/False
      tasks: >
        - name: task1
        <task1>
        - name: task2
        <task2>
        ...

Example 2: extended task
    
    ---
    - name: Play1
      sudo: True/False
      vars:
        <var1>: <varvalue1>
        <var2>: <varvalue2>
        - ...
      tasks: 
        - name: task1
        <task1>
        - name: task2
        <task2>
        - ...
      handlers:
        - name: handler1
          service: 
        - name: handler2
          service: 
        - ...


#### Tasks

Tasks are the execution units of plays.
Tasks are structured in the following way:

Mandatory Arguments:
* Tasks are key/value pairs:
    * Key with the name of a module.
    * Value with the arguments to that module.

Optional Arguments:
* `name`: What a task is about; Ansible prints the name of the task when executed.


Some Notes:
* Arguments are treated as strings and not as dicitionaries;
  line folding has to be with >:

      - name: <taskname>
        <module>: >
            <arg1>=<value1>
            <arg2>=<value2>

* Ansible can be forced to start a Task at the middle of a play:
  `ansible-playbook --start-at-task <taskname>`.

* Ansible has also syntax for specifying module arguments as a YAML dictionary.
  (later)

#### Vars

Plays can have a secion called `vars`; each variable consists of a 
key/value-pair.
Variables can be used in tasks as well as in template files:
they are referenced using `{{ varname }}`.
The value of the variables is substituted for the varname.

Attention:
* Strings that contain variables directyly at the beginning have to be quoted
  to disambiguate parsing. Example:

       - name: some name
          command: "{{varname}} -a foo"

* Arguments that contain a colon have also to be quoted. Example:

       - name: some name
           debug: "msg='this contains a colon:'"



#### Configuration templates

Configuration files can be generated from templates:
Similar config files with variables in the place of values that get instantiated 
to their proper values when the machine is fixed and the values are known.

Templating in Ansible is implemented using the Jinja2 template enginge.

Template files are customarily in a `templates` subdirectory.

[Jinja2 documentation](http://jinja.pocoo.org/).


#### Handlers

Handlers are one of the conditional forms that Ansible supports.
A handler is similar to a task, bit it only runs if it has been *notified* by a
task.
A task will fire the notification if Ansible notices that the task changed the 
sytem (like an implication).

Example:

    - name: Play
      tasks:
        - name: Task
          notify: Handler
      handlers:
        - name: Handler 

Use cases for handlers:
* Restarting services (change of the configuration file, ...)
* reboot

Restarting services can happen unconditionally at the end of a playbook,
so handlers are pretty useless most of the time.

Some things about handlers:
* Handlers only run abter all of the tasks are run.
* Handlers only run once, even if they are notified multiple times.
* They always run in the order in that they appear in the play,
  not the notification order.

---
### Ansible Component 2: Modules (Ansible "Libraries")

Official Ansible documentation:
* [About modules](https://docs.ansible.com/ansible/modules.html)
* [Module Index](https://docs.ansible.com/ansible/modules_by_category.html)

Modules are the unit for generic abstractions in Ansible:
* Standard Modules: Some of them come directly with Ansible (standard library).
* User Modules: User defined modules are possible.

Modules are scripts that perform some action on the target machine.
Each task is related to exactly one module.


---
### Ansible description language: YAML

Ansible playbooks are written in YAML syntax (YAML is the markdown of JSON).

##### YAML Syntax

1. Beginning of the file: 
   YAML files are suppoes to start with three dashes:

        ---

2. Comments:
    Line comments are initialized with a hash

        # YAML comment

3. Strings:
    A YAML string doesn't have to be quoted.

        This is just a YAML string
	"This is just a YAML string"
    
4. Booleans:
    * YAML truthy: `true`,`True`,`TRUE`,`yes`,`Yes`,`YES`,`on`,`On`,`ON`,`y,`Y`
    * YAML falsy: `false`,`False`,`FALSE`,`no`,`No`,`NO`,`off`,`Off`,`OFF`,`n`,`N`

5. Lists:
    YAML lists are like arrayz in JSON and RUBY or like lists in Python;
    called *sequences* in YAML. Lists are determined with hyphens:

        # regular list definition
        - First item
        - Second item
        - Third item

        # inline list definition
        [First item, Second item, Third item]

6. Dictionaries
    YAML dictionaries are like objects in JSON (mappings). Example

        # regular dictionary definition
        first_name: XXX
        last_name: YYY
        age: 0

        # inline list definition
        [first_name: XXX, last_name: YYY, age: 0]

7. Line folding
    YAML line folding with the *>* character. The YAML parser will replace line
    breaks with spaces. Example:

        first_name: >
            XXX
        last_name: >
            YYY
        age: >
            0

YAML is a superset of JSON - a valid YAML file is also a valid JSON file.



### Best Practices

[Best Practices](https://docs.ansible.com/ansible/playbooks_best_practices.html)
