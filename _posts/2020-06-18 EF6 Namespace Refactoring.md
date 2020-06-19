---
layout: post
author: Sanjeev
comments: true
categories: [Entity Framework]
tags: [EntityFramework, EF6]
identifier: EF6NameSpaceRefactoring
title: Refactoring project name space
---
As the projects get older, some times old Namespace no longer make sense. It is better to change the name space using refactoring. Namespace refactoring usually should not be a problem. However you are in for a surprise if you happen to refactor the names space used by your code first migration.

This is specifically true if you are using Entity framework's Code first migration v6.0. You can confirm this by checking the schema of your <code>__MigrationHistory</code> table. If this table has a column <code>ContextKey</code> then your code uses Namespace else not an issue.

If your entity framework's migtation is using Namespace and you would like to refactor your Namespace, then below solution would work for a typical project. Here typical project mean, a project with single <code>DBContext</code>. Simple trick is to drop the column <code>ContextKey</code>. When the migration is run next time, Entity framework will recreate the column with new Namespace. Below SQL can be used to delete the column

<pre><code class="sql">
ALTER TABLE __MigrationHistory DROP PRIMARY KEY;
ALTER TABLE __MigrationHistory DROP COLUMN ContextKey
ALTER TABLE __MigrationHistory ADD CONSTRAINT PK_dbo.__MigrationHistory PRIMARY KEY (MigrationId)
</code>
</pre>