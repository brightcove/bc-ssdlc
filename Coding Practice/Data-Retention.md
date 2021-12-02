# Data Retention [Coding Practice]

## Overview

Almost all web applications have a feature that is responsible for retaining persistent data. Whether it's sent by the client or generated by a server, data should always be secured as appropriate, and only retained long enough to complete its purpose.

## General Retention
### Recommendations
#### Description

**Data retention** refers to how long persistent data is kept by its controller, i.e. your application.
#### Why We Care

Keeping data longer than its needed causes an unneeded increase in risk for the owner of the data, whether it's our own here at Brightcove, or one of our users' that they've provided as part of our services.
#### Example of Issue (Optional)

User analytics are one of the most common cases of data being kept longer than it should. 

An example of this would be an analytics application that is developed for marketing contractors. It does its job of collecting data and providing feedback, etc. But when the contractors complete their jobs, the data is still kept, whether by mistake or a "just in case" situation where its kept around because it's full usage by other applications is unknown. 
#### How to Fix?

There are several best practices you can introduce to maintain responsible retention of data:

* Include data retention from the design phase of the SDLC process for the app
  * Decide how long you think you'll need the data obtained by the application
  * Make sure to include a way to easily change the retention period, if warranted
* Regularly reassess the retention needs of the application as time goes on
* Keep in mind compliance frameworks and privacy laws, such as GDPR
* Ensure there's an easy and straightforward process involved for fully deleting the data set, when ready
* Make sure that data is encrypted when needed, both during transport and at rest
    * Ex: customer information used in the example user analytics above should be sent over TLS, and any databases the data is stored in should be encrypted at the database level
    * AWS RDS databases and S3 buckets both offer an encryption at rest option
#### Risk Rating

The risk associated with improperly retained data range from Low to High, depending on the data involved.

## Data Auditing/Versioning
### Recommendations
#### Description

**Data auditing** or **data versioning** refers to the implementation of an audit trail that tracks changes made to a given data set. That data could be customer templates, user account profiles, etc.
#### Why We Care

Several reasons:
* It helps ensure several of the pillars of security - non-repudiation and integrity - are covered within your application
* When an operational issue that occurs has the chance of being caused by a user's own actions, this helps differentiate whether the root cause was a misconfiguration or an actual bug within your application
* In the case of a security incident, these records can be referenced for forensic investigations
* Some compliance frameworks require this for certain types of data
#### Example of Issue (Optional)

A company has developed a web app that provides a WYSIWYG web site editor for their users to easily create their own business websites. Because you're in the early stages of development and are a new company, you have not yet implemented data auditing for your customer's HTML templates.

One day, a customer calls your support team to claim that all of their data on their account has been lost or heavily changed. A support engineer confirms the customer's claim.

Now what is the root cause? Did the customer have a disgruntled employee that had access to this data delete everything on their way out? Was the account - or worse, the entire application - breached by an outside attacker? All of these questions have a better chance of being answered when data auditing is implemented.
#### How to Fix?

To implement a proper data auditing system for your software, you first have to decide on several design choices.
#### What data points should have an audit trail for them?
The recommendation for this is to audit any record that an outside party (whether that be a user, third-party company, etc) has the ability to directly change. In our example above, we would want to save the HTML templates for the user's account any time they are changed, both via the UI and API.

#### What method/algorithm am I going to use for the audit trail?
While there are several algorithms and techniques to chose from, some of which may fit your application's use case better, the recommended method at Brightcove is to use a **dedicated data-tracing table/DB**.

This table works by saving a record for each column change made within a data store. Again using our example from above, an audit record for the HTML template may look something like:

| audit_record_id | table | column | pri_key | changed | changed_by | originally_created | originally_created_by | original_val | last_val | new_val |
| --------------- | ----- | ------ | ------- | ------- | ---------- | ------------------ | --------------------- | ------------ | -------- | ------- |
| 1 | html_templates | template_html | 254 | 2020-01-01 12:05:46 | jsmith | 2015-03-04 09:33:02 | cthompson | `<INITIAL_HTML>` | `<OLD_HTML>` | `<NEW_HTML>` |

A few points to keep in mind for this table:

- This table should ONLY store the audit records; none of the source data should be stored alongside it (a different table in the same database is fine)
- At a minimum, you should include the following data points:
  - Identifier for original record (table/column/pri_key)
  - Timestamp of when the change happened
  - Identifier of who made the change
- It's _preferred_ to resolve all foreign keys to their actual values (e.g. jsmith instead of 5, with 5 corresponding with employee_id) for auditability purposes
  - However, there are use cases where this may not be possible due to storage concerns, or the preference of the data management advantages of using foreign keys vs the preciseness of using the actual value.
#### How should I store this data?
In a nutshell, you should store audit trail data in the same way as the original data. For example, if you're using a relational database, use a relational database to store the records. If a NoSQL database is being used, store the record in a NoSQL database with additional audit-related fields added.

Doing this allows for developers and operational users alike to easily be able to grasp how your application auditing works, and without having to implement any additional dependencies to work with these audit records, should the need arise.

Additionally, you should also mirror the storage requirements of the original data your versioning. That means that if the original data is encrypted, or air-gapped for example, so should the audit records. Passwords and other sensitive data (e.g. private keys) should almost never be included as part of versioning as the security risk of saving this data in multiple places is not worth the advantages of auditing the data.
#### Risk Rating

The risk associated with not auditing data can range from not being a problem at all if the data is inconsequential, all the way up to HIGH in the case of a severe security breach.

#### Additional Notes

One of the downsides of data auditing, and likely one of the reasons a lot of companies/software engineers don't implement it into their applications, is that:

* It's not an easy task to get completely right, programmatically speaking
  * Edge cases tend to come up, both with data input and data output
* It adds a non-trivial amount of extra cost to the application
  * Because data auditing, by design, tends to take up a fair amount of storage, some companies forgo it to save on costs
#### References

- https://dba.stackexchange.com/questions/114580/best-way-to-design-a-database-and-table-to-keep-records-of-changes
- https://www.codeproject.com/Articles/105768/Audit-Trail-Tracing-Data-Changes-in-Database