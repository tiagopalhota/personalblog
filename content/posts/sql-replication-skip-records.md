---
title: 'Skipping records in SQL Server replication'
date: 2023-04-18T16:07:15+01:00
draft: false
description: 'This post explains the steps required whenever transactional replication is stopped due to an error on the subscriber and we want to force it to continue by ignoring the problematic transaction'
thumbnail: '/images/Replication is broken.png'
---

### **Replication is down!😟 Now what?!**

This happened to us today, one of our subscribers, decided to implement a few triggers on their subscriptions. These triggers were inserting data into other tables that had a few foreign keys to respect their data integrity.

The problem here is that for some reason one of these foreign keys failed, the entire replication got brought to a halt. No more data is delivered until the replicated transaction can complete.
In our case, a deeper look had to be taken by our colleagues into the way foreign keys have been applied and we could not have the entire replication stop because of this( it only happened for a few records due to a change in natural keys done a while ago).

So, how can tell the distributor agent to ignore this transaction and just move on?
Time for [sp_setsubscriptionxactseqno](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-setsubscriptionxactseqno-transact-sql?view=sql-server-ver16).

{{% notice tip "sp_setsubscriptionxactseqno Syntax" %}}
sp_setsubscriptionxactseqno [ @publisher = ] 'publisher'
, [ @publisher_db = ] 'publisher_db'
, [ @publication = ] 'publication'
, [ @xact_seqno = ] xact_seqno
{{% /notice %}}

{{% notice info "Where do I run this?" %}}
This has to be run in the subscriber database.
{{% /notice %}}

### ❓ How do we find which records are causing replication to stop?

Our first option is to use our old replication monitor friend.
![Replication Monitor, failed xact](/images/replication-monitor-xact-failed.png)

Here we can use the replication monitor to identify which log sequence number is causing the issue or we could have a look at subscription errors using [sp_helpsubscriptionerrors](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-setsubscriptionxactseqno-transact-sql?view=sql-server-ver16).

{{% notice tip "sp_helpsubscriptionerrors Syntax" %}}
sp_helpsubscriptionerrors [ @publisher = ] 'publisher'
, [ @publisher_db = ] 'publisher_db'
, [ @publication = ] 'publication'
, [ @subscriber = ] 'subscriber'
, [ @subscriber_db = ] 'subscriber_db'
{{% /notice %}}

We should get something like this:
![sp_helpsubscriptionerrors, resultset](/images/replication-skip-records-helpsubscriptionerrors.png)

{{% notice warning "Warning" %}}
It´s possible that you see past events in the output of this stored procedure. Take the time in consideration.
{{% /notice %}}

To understand what command we are really talking about, we can use sp_browsereplcmds. This will return a result set in a readable format of the replicate commands stored in the distribution database. This SP is executed at the distributor on the distribution database.
The syntax of the procedure is:

{{% notice tip "sp_browsereplcmds Syntax" %}}
sp_browsereplcmds [ [ @xact_seqno_start = ] 'xact_seqno_start' ]
[ , [ @xact_seqno_end = ] 'xact_seqno_end' ]
[ , [ @originator_id = ] 'originator_id' ]
[ , [ @publisher_database_id = ] 'publisher_database_id' ]
[ , [ @article_id = ] 'article_id' ]
[ , [ @command_id= ] command_id ]
[ , [ @agent_id = ] agent_id ]
[ , [ @compatibility_level = ] compatibility_level ]
{{% /notice %}}

## ❓ I´ve run sp_setsubscriptionxactseqno on the subscriber, now what?

The next step is to stop and start the distributor agent again. The next time the agent starts, it will see that there´s a flagged transaction to be skipped and it will pick up from the next available transaction available.
