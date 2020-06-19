---
layout: post
author: Sanjeev
comments: true
categories: [EntityFramework]
tags: [EntityFramework, EF6]
identifier: EF6NameSpaceRefactoring
title: Entity Framework And Namespace Refactoring
---
As the time goes by, few Namespaces used in project(s) no longer make sense. It is better to refactor such Namespaces to some meaningful ones. Namespace refactoring usually should not be a problem. Surprise is waiting if entity framework is used in the project.

This is specifically true if the project uses Entity framework(v6.0)'s Code first migration. This can be confirmed by checking the schema of <code>__MigrationHistory</code> table. If this table has a column by name <code>ContextKey</code> then EF code uses Namespace.

If entity framework's migtation is using Namespace and there is a need to for refactoring the Namespace, then below solution would work for a typical project. Here typical project mean, a project with single <code>DBContext</code>. Simple trick is to drop the column <code>ContextKey</code>. With this, when the migration is run next time, Entity framework will auto re-create the column <code>ContextKey</code> with new Namespace. Below SQL can be used to delete the column

<pre>
    <code class="SQL">
        ALTER TABLE __MigrationHistory DROP PRIMARY KEY;
        ALTER TABLE __MigrationHistory DROP COLUMN ContextKey
        ALTER TABLE __MigrationHistory ADD CONSTRAINT PK_dbo.__MigrationHistory PRIMARY KEY (MigrationId)
    </code>
</pre>

<script>hljs.initHighlightingOnLoad();</script>