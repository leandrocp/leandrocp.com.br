---
layout: post
title: How to update objects inside JSONB arrays with PostgreSQL
date: 2019-04-15
background: '/img/posts/drawers.jpg'
---

## How to update a specific value on a JSONB array

Let’s say you decided to store data in the database as json or jsonb and discovered that you just created new problems for yourself that you didn’t have before. You’re not alone.

JSONB is a powerful tool, but it comes at some cost because you need to adapt the way you query and handle the data.

And it’s not rare to load the entire jsonb object into memory, transform it using your preferred programming language, and then saving it back to the database. But, you just created another problem: performance bottlenecks and resource waste.

In this article let’s see how to update a specific value of an object inside an array with one query.

**TL;DR**: the final query is at the end of the article, and you can check out a live example at [DB Fiddle](https://www.db-fiddle.com/f/e8aeGk7cRNYnpjsqi1ncrs/1) to copy & paste and play with.

Suppose you’re implementing a customer screen to store dynamic contacts for each customer. Then you come up with the idea of storing the contacts as a JSONB column because they’re dynamic, and thus using a not relational data structure makes sense.

Then you create a customers table with a JSONB contacts column and insert some data into it:

```sql
create table customers (name varchar(256), contacts jsonb);

insert into customers (name, contacts) values (
  'Jimi',
  '[
    {"type": "phone", "value": "+1-202-555-0105"},
    {"type": "email", "value": "jimi@gmail.com"}
  ]'
);

insert into customers (name, contacts) values (
  'Janis',
  '[
	{"type": "email", "value": "janis@gmail.com"}
   ]'
);
```

Pretty easy right? But how can you update a specific contact for a specific customer? How to change Jimi's email or Janis’ phone? 🤔

Fortunately, PostgreSQL is your friend and provides the *jsonb_set* function:

`jsonb_set(target jsonb, path text[], new_value jsonb[, create_missing boolean])`

Given a jsonb column, you can set a new value on the specified path:

![]({{site.baseurl}}img/posts/sql-jsonb-1.png)

*Reference: [PostgreSQL Json functions](https://www.postgresql.org/docs/9.5/functions-json.html)*

The above selects will return:

```
  [{"type": "phone", "value": "+1–202–555–0105"}, {"type": "email", "value": "jimi.hendrix@gmail.com"}]
  [{"type": "email", "value": "janis.joplin@gmail.com"}]
```

To change Jimi's email on the contacts list, you inform the path `1, value` which means the second object on the array (starting at 0) and the key **value**. That's the *path*. The same applies to change Janis’ email, but its email object is at index 0.

You may be thinking: I just have to use jsonb_set on an update statement and it’s all done? That’s the idea, but that’s not enough yet.

The problem with non-relational data is that they’re dynamic. Well, that’s one of the reasons for using JSONB but that brings a problem: see that Jimi’s email object is at index 1 and Janis’ email object is at index 0 in the array, and another customer could have a very different array with different indexes. So, how can you discover the index of each contact type? 🤔

The answer is ordering the elements of the array and getting its index:

```sql
select index-1 as index
  from customers
      ,jsonb_array_elements(contacts) with ordinality arr(contact, index)
 where contact->>'type' = 'email'
   and name = 'Jimi';
```

That query returns **1**, which is the *index* of the email object (type email) inside the contacts array of the customer Jimi.

Now we have all the pieces of the puzzle: we know how to update a jsonb value and how to discover the index of the object to be updated.

The only step left is the update itself. Putting it all together we have:

![]({{site.baseurl}}img/posts/sql-jsonb-2.png)

The most important part of this query is the `with` block. It's a powerful resource, but for this example, you can think of it as a "way to store a variable" that is the *path* of the contact you need to update, which will be dynamic depending on the record.

Let me explain a bit about this part:

```sql
('{'||index-1||',value}')::text[] as path
```

It just builds the path as `'{1, value}'`, but we need to convert to `text[]` because that’s the type expected on the `jsonb_path` function.

### Wrapping up

JSONB is a great and valuable tool to resolve a lot of problems. But keep in mind that you also need to query and update this kind of data. That brings a cost that you have to consider when deciding which tools you pick to use.

*Side note: that solution came out of a pair programming session with [Lucas Cegatti](https://twitter.com/lcegatti).*
