---
layout: post
title:  "Rollback db"
date:   2017-06-17 16:33:26 +0000
---


While working on one of the ActiveRecord association labs, I accidentally created an *artists* table instead of *actors* table and persisted the table by running *rake db:migrate*. After realizing my mistake a few minutes later, I went on and tried to fix my error by changing *artists* to *actors* in the migration file and ran ```rake db:migrate``` again. That, in my naivete, was a big mistake. Since the version of the migration file was the same, it did not change anything. But because I changed the file, I could not rollback to the previous version either.

From what I could glean from my research, any correction in the migration file needs to follow the following pattern:
1. Rollback to the previous version if the most current version is the erroneous one. This will essentially undo the latest ```rake db:migrate``` task.
2. If the error is not in the current version, then move the version down to the one before the erroneous version using 
```rake db:migrate:down VERSION = version_number```.
3. Make necessary corrections.
4. Run *rake db:migrate* again.

But since I did not follow this pattern, I inadvertently created 2 tables in the database *artists* and *actors*. And when I tried to rollback, it would not let me because it cannot find the *artists* table in the migration file as I'd already overwritten it with the *actors* table. My *artists* table was lost in the database purgatory. I tried deleting the migration file, but that only deleted the content but the version number persisted in the system, even after dropping the *artists* table manually.

I had 2 options: 
1. Write another migration file to drop the *artists* table and create *actors* table and leave the bad migration version be, or,
2. Do some more google. CAVEAT EMPTOR: What I ended up doing was to change the *schema_migration* table. This is apparently where ActiveRecord keeps tab on all the migration files. 
```
DELETE FROM schema_migration WHERE VERSION='erroneous_version_number'
``` 
would remove the migration file from the memory. I could then create the correct migration file and run the migration to persist the tables into the database. 

I am not sure how this will work out if you have a complex database model especially if you want to delete a migration file that is not the most current one. I'd rather just stick with the correct pattern.


