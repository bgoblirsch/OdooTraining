# OdooTraining

# Monday June 3

* Need to do third party for custom domain SSL support - recommended cloudflare.com (free)
* Staging dbs are not backed up
* recommended 2 staging branches. one exact copy of prod. and one working staging 
    * **exact copy to help diagnose and test any existing issues**
    * **another for any working improvements**
    * it can be done on one, but this is what Odoo recommends
* All custom addons/modules go in src > user folder
    - no need to mess with anything else in src folder; everything can be inherited
* roughly 1 worker per 6 concurrent users recommended
* Odoo "Cron" does not use linux Cron, but its own
* general workflow
    * fork prod > develop code in dev > merge to staging
* Collaborators
      - correlates w/ github access levels
      - after adding in odoo.sh settings, also need to grant access in Github
      - repo > settings > collaborators
* when creating a new module, it's good/easy to reference existing community manifest files
* updating manifest version # while force .sh to update the module

# Tuesday June 4

 - ORM, ORM, ORM
     - read all the documentation, then read it again
     - it abstracts how you interface with the database
- Class models are slightly dif from typical "classes" in OOP, a class model is a table in Postgres. So you don't create new class objects, when adding records to a table
    - for example, if you have 7 sale order records, it's not 7 class objects, 
    - "self" in a class model is that model's recordset
- each field in a model can be accessed as a record's attribut: record.name = "Bob"
- odoo model inheritence is seperate from Python inheritence
- utf-8 comment for coding convention is required in every .py file
- when referring to other models (within the same module) for relational fields, there is no need specify module

## Academy Problem One

We need models for:
 - Course/class
     - Title
     - level
- sessions
    - teacher
    - course
    - start & end date
- person
     - teacher or student
     - first name
     - last name

### Git Flow
 - should check the branch out right away when creating a new module
 - CD to "src/user"
 - `git add -A`
 - `git checkout [branch_name]`
 - `git commit`
	 - use this to skip message dialog `git add -m "[INSERT_MESSAGE]"` 
 - `git push https HEAD:[branch_name]`
 - `git push --set-upstream https [branch_name]`
	 - these two are the same, just diff syntax 
	 - you can use "git remote -v" to display available upstream branches
	 - after doing this the first time, upstream is set and just `git push` will suffice
 - enter github credentials
 - if for whatever reason, .sh editor is not working, you can close the repo created in .sh to local, code there, push, and .sh will auto-update

### Scaffolding
in the terminal:

    odoo-bin scaffold [module_name]

this creates a new module template

### Nice tool:

    odoo-bin -u open_academy --stop-after-init
   reloads modules without having to restart server 
   -u is upgrade 
   -i is install

# Wednesday June 5th

Very helpful to have full odoo source code available to search while you are programming

### Write methods very important; review:
https://www.odoo.com/documentation/12.0/reference/orm.html#model-reference

### adding module rights
security folder -> csv files
id = xml ID
name = human readable name of access right; example = "Courses Full" or "Courses Reader"
model_id:id = model name in python class file; all periods replaced with underscores

### PDB - python debugger
import pdb # make the this they very first import first to avoid any issues 

## Gen Notes
- computed fields are not stored in the DB by default, must use `store=True` if you want to do that
- know that you must use api@depends when doing computed fields
    - https://www.odoo.com/documentation/12.0/reference/orm.html#computed-fields
- compute() cannot track UI changes, instead use onchange()
- can use [record_set].ensure_one() to make sure that a recordset is a singleton
 - "self" starts as active user, but it can be assigned to a dif recordset
 - if no value is provided for a field, it is set as False
     - essentially < null >
     - odoo reads data top down; you can't reference something that comes after the source
     - .mapped() function will NOT return a recordset
     - data type does not normally need to be specified; odoo is good at interpreting it. But for instance, you can specify base64 for a base64 image str
 - you can use eval="[expression]" to evaluate data when data is assigned/created, for example:

< `field name="date_start" eval="date.today()"/` >
< `field name="start_date">2019-06-05</field` >

### Environment notes
[https://www.odoo.com/documentation/12.0/reference/orm.html#environment](https://www.odoo.com/documentation/12.0/reference/orm.html#environment)
- not equivalent to "environment variables"
- has 3 elements: db cursor, user, and user context
 - `odoo-bin shell` in terminal to access Odoo Python
     - can use .sudo() with a user as a parameter to imitate performing an action as a specific user; pass nothing to imitate admin
     - must be extremely careful when using sudo
- environment is global; any recordset can call/interact w/ it
- everything in env is immutable; use with_Context() if need to "change" something

### Domains
[
https://www.odoo.com/documentation/12.0/reference/orm.html#domains](https://www.odoo.com/documentation/12.0/reference/orm.html#domains)

- for querying data; all operators listed in documentation above 
- domain consists of one or more triples; simply seperate by comma to do AND
- domains are evaluated in [Polish Notation](https://en.wikipedia.org/wiki/Polish_notation) (the two operands come after the operator)

In order to set all users named "John" to inactive:

In order to update all courses to specific course:

    math_course = self.env.ref('open_academy.course_math_101')
    sessions.write(
        {
            'course_id' : math_course
        }
    )
    #sessions is assumed to be a record set that has already been seached

# Thursday Jun 6

## Gen Notes
- debug button > view metadata to extract ID & XML ID 
- debug button > edit: view form to reference source code of completed modules like sales, inventory, etc.

### Views
priority: lower #s = higher priority

Forms: you can use equivalate to bootstrap for setting up the form



- 

# Friday June 7
- Odoo User Types:
    - internal user
        - paid users, backend odoo acess; employees, managers
    - portal user
        - not paid users, frontend odoo access; website shop users
    - public user
        - not paid users, no log in
        - don't delete

### Migrations & Upgrades:
[https://upgrade.odoo.com/](https://upgrade.odoo.com/)
if odoo was contracted to do any custom development, they will migrate that for free, but any other custom code is not free for migration
best case scenarios, w/ out of the box odoo (no customization) 4-5 week process
1. you apply to do an upgrade
2. you are provided a testing database with upgrade
3. install custom modules in the test database and test EXTENSIVELY
4. back and forth w/ migration team
5. once all is good, you upgrade production

Version Upgrade:
- start by migrating code to new version, figure out what doesn't work
- there are Odoo partners that could help w/ the overall process
- we may find features that have been dropped in new versions
- very iterative process of test, identify issues, investigate cause, fix, and repeat until all issues are worked out
- in summary, unless any features are absolutely needed, it is probably better if we focus on cleaning up and optimizing our code to make future upgrades easier, and since version 13 is coming soon, it is likely better to just wait and upgrade to 13

there is a community upgrade script, but it will likely require extensive customization to make it work

### External API
anything you can do in Odoo, you can do with the XML-RPC API

this is a good place where the batch import for csv's could be implemented
### Odoo's typical workflow
1. develop locally on new/plain instance
2. check out branch in .sh, and push local code to github
3. connect to newly built instance in .sh & test
4. once good, merge to staging branch (NTS Team helps test)
5. after testing in staging, merge to production
6. if this merge fails, it will roll back to previous successful build

### There will be more training before Odoo Experience in Belgium in October; two day hands-on
