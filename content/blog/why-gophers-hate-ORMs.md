+++
title = "Why Gophers Hate ORMs"
date = 2025-12-05T11:05:38+05:30
draft = false
showToc = true
cover.image = '/images/gopher_orm.png'
tags = ['golang', 'tech']
+++

Last year, when I started learning Golang, and while building a project, I needed to work with databases. I stumbled upon an interesting conundrum in the Golang community: **Everyone here seems to hate ORMs.**

I wondered why. Coming from ecosystems like Python (Django/SQLAlchemy), the ORM is usually the default starting point. Although I went my way with writing SQL queries by hand for that specific project, recently this question again resurfaced in my mind.

Why do Gophers generally reject the Object-Relational Mapper?

These were the points which one would normally hear _for_ using ORMs in other languages:

- **Velocity:** "I want to move fast and break things."
- **Abstraction:** "I don't want to know which SQL dialect I'm using."
- **Convenience:** "I just want `user.save()`."

But Go is a different beast. After combing through discussions and Reddit threads, it becomes clear that the rejection of ORMs isn't just about preference - it's about the fundamental philosophy of the language.

## 1. The Burden of "Mental Translation"

One of the strongest arguments against ORMs is the issue of transferable knowledge versus proprietary knowledge. SQL is the lingua franca of data. It has existed for decades and will exist for decades more.

When you use an ORM, you are often forced to learn a "Domain Specific Language" (DSL) or a complex chain of method calls that only applies to _that specific library_.

> If i have to interact with the database already, i just want to write sql, not learn another framework with its own rules and quirks.

**Consider this scenario:**
You need to write a complex query involving a `LEFT JOIN`, a `GROUP BY`, and a Window Function.

1.  **With SQL:** You write the SQL. You test it in your DB client. It works. You paste it into your code.
2.  **With an ORM:** You know the SQL, but now you have to figure out how to translate `OVER (PARTITION BY...)` into the ORM's method chaining syntax. You spend hours reading documentation for a wrapper around a language you already speak.

As another user pointed out, the abstraction layer often creates more work than it saves:

> Things that could have just been sql queries had to go through abstractions and "magic" which eventually shoots you in the foot when you didn't handle that one edge case, or don't understand how it works underneath the table.

## 2. The Fallacy of the "Black Box"

Go is a language that values **explicitness**. We handle errors explicitly (`if err != nil`). We prefer readable code over clever code. ORMs, by definition, rely on "Magic", implicit behaviors that happen without the developer seeing them.

This "Black Box" nature becomes a nightmare when performance issues arise.

> For gods sake i just want to unmarshal my row from a merge sql query into the damn field, not think about how the orm first executes the query and a prefetch of some kind which maps the value back to the foreign key object IF AND ONLY IF it exists in the first query.

**The N+1 Problem:**
This is the classic ORM killer. You think you are fetching a list of users, but the ORM implicitly triggers a separate query for every user to fetch their associated "Profile." Suddenly, one page load triggers 51 database queries.

In Go, because you write the query explicitly, you see the `JOIN`. You know exactly the cost of the operation before you compile the code.

## 3. Encouraging Bad Architecture

Perhaps the most insidious issue with ORMs is how they influence application design. Most ORMs encourage the "Active Record" pattern, where your database table is 1:1 mapped to your application object.

This leads to dangerous coupling:

> Orms also encourage bad usage, I have seen code that just saves whatever "object" is passed from front end. You cant imagine the amount of overwritten data and invalid states that caused.

When your API request body, your internal business logic struct, and your database model are all the exact same struct (because the ORM makes it easy to do so), you risk:

1.  **Mass Assignment Vulnerabilities:** A user updating a field they shouldn't be able to touch (like `is_admin`).
2.  **Anemic Domain Models:** Business logic gets scattered across controllers rather than residing in the domain layer.

## 4. The "Leaky Abstraction" Reality

ORMs promise that you can switch databases easily (e.g., from Postgres to MySQL) without changing code. In practice, this is rarely true for non-trivial applications.

Eventually, you will need a feature specific to your database (like Postgres' `JSONB` or `PostGIS`). Once you need that, the ORM's abstraction leaks. You end up writing "Raw SQL" fragments inside the ORM methods anyway.

> I know its good if you need to migrate databases due to the abstraction layer but for gods sake just write sql.

If you are going to end up writing raw SQL to optimize performance or use specific features, why carry the heavy baggage of the ORM framework?

## 5. The Gopher's Alternative: Type-Safe SQL

So, if Gophers hate ORMs, do they enjoy parsing raw rows manually? No.

The Go community has coalesced around a "Middle Way." We want the **control** of SQL with the **convenience** of struct mapping.

### sqlx / scany

These libraries are not ORMs. They don't generate SQL for you. They simply say: _"You write the SQL, and I will handle mapping the result rows into your Golang structs."_

### sqlc (The Gold Standard)

`sqlc` has taken the Go world by storm. It reverses the ORM model entirely.

- **ORM:** You write Go code -> It generates SQL at runtime.
- **sqlc:** You write SQL -> It generates type-safe Go code at compile time.

This approach aligns perfectly with Go's philosophy:

- **Compile-time safety:** If your SQL is invalid, your code won't compile.
- **No Magic:** You know exactly what query is running.
- **Performance:** No runtime reflection overhead.

## Conclusion

The hatred for ORMs in Go isn't elitism; it's pragmatism. Go developers are responsible for the maintenance and performance of their services. We prefer tools that are sharp, explicit, and do exactly one thing well.

We don't want a framework that tries to be smarter than us. We just want to execute a query.
