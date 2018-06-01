# Disaster Recovery plan

Products
--------
- What products or services are we supporting
- How do they impact the business
- What is deemed to be a recovered state

Scenarios
---------
- What are the possible scenarios to trigger a disaster recovery

Dependencies
-----------
- Who are dependent on these services
- Who is affected downstream

Backups
-------
- Store backups on a different AWS region. 
- Replicate to a different AWS region.
- Back up on a daily basis as a raw data dump to a S3 bucket in a seperate AWS account.
- Manually encrypt RDS snapshots daily.

Roles & responsibilities
------------------------
- Create a RACI chart (Accountable, Responsible, Consulted, Informed)

Escalation
----------
- Create an escalation process. Define which groups are involved at what stages.

