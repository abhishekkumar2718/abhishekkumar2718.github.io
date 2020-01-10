---
layout: post
title: A Curious Case of Optimisation
category: Programming
tags: ['MVC', 'Rails', 'Ruby', 'Clean Code']
excerpt: Read about efficient solution to iterating relations in MVC including benchmarks!
---

Consider the following problem often encountered in web applications:

> There are two models `Student` and `User.` The students table has an attribute `user_id` which references the `id` on the users table. You want to print out the username of students in section A.

You implement the following code, which translates to the pseudocode - Fetch all students in section A from the database. For each student, fetch the user from the database.

```ruby
students = Student.where(section: 'A')

students.each do |student|
  puts User.find(student.user_id).username
end
```

Someone reviews your code and correctly points out that you are [running N + 1 queries](https://medium.com/@bretdoucette/n-1-queries-and-how-to-avoid-them-a12f02345be5) which slows the page. You fix it by prefetching all users and finding the right user in memory.

```ruby
students = Student.where(section: 'A')

user_ids = students.pluck(:user_id)

users = User.where(id: user_ids)

students.each do |student|
  user = users.find { |user| user.id == student.user_id }
  puts user.username
end
```

You run two queries now, and page loads faster on the development database. Your code is merged, but the production feels more sluggish.

**That doesn't make sense!**

{% include toc.html %}

#### Benchmarks

Here are benchmarks for the above and two other implementations - Hash and ORM API (discussed below) on a local database:

| N      | (N + 1) Query     | Prefetch | Hash   | ORM API |
| ------:| -----------------:| --------:| -----: | -------:|
| 10     | 0.0163            | 0.0055   | 0.0053 | 0.0048  |
| 50     | 0.0557            | 0.0108   | 0.0082 | 0.0089  |
| 100    | 0.1005            | 0.0197   | 0.0120 | 0.0140  |
| 500    | 0.4517            | 0.1505   | 0.0428 | 0.0564  |
| 1,000  | 0.9273            | 0.4634   | 0.0767 | 0.0927  |
| 5,000  | 4.9729            | 10.572   | 0.3522 | 0.3989  |
| 10,000 | 9.3168            | 60.571   | 0.6063 | 0.8187  |

Observations from the above table:
1. Hash and ORM API approach scale well - taking less than a second for 10,000 records.
2. Prefetch performs better for small values of N (< 2000) which is typical for development 
3. (N + 1) query functions better for larger N, present in the production database.

> The benchmark depends on many factors - notably the round trip time from the database. Local databases respond in ~2 ms, whereas a remote database responds in ~150ms, which completely changes the benchmark results.

#### Analysis

Here's understand the costs involved in both methods:

```
Costs of (N + 1) query   = (Fetch students) + N * (Fetch user)
                         = O(N) queries
```

```
Costs of prefetch method = (Fetch students) + (Pluck user_ids) 
                           + (Fetch users) +  ((N * (N - 1)/2) * iterations
                         = O(N^2) iterations
```

Cost of iterations is much smaller than the cost of queries (10^-4ms << 0.5ms).

For small N, the cost of N queries dominates over the N^2 iterations. But for larger and larger N, the cost of N^2 iterations adds up and increases quadratically.

> Profiling is the only way to be sure about performance.

Let's see two better ways to solve this problem.

#### Hash approach

Here's the critical realization - Each of the users match precisely one of the students. Hashes are a great way to store one to one mappings. 

Using hashes converts O(n) inner iteration of prefetch method to O(1) hash table lookup.

Algorithm:
1. Fetch all students in section A from the database.
2. Fetch all user_id from such students from the database (using `SELECT students.user_id` or the ORM equivalent).
3. Fetch all users using `user_id` in step 2.
4. Create a hash using the foreign key as key and the object itself as the value.
5. For each student, look up the hash table and find the user.

Implementation: 
```ruby
# Step 1
students = Student.where(section: 'A')

# Step 2 - Avoid a query using preloaded students
user_ids = students.pluck(:user_id)

# Step 3
users = User.where(id: user_ids)

# Step 4
users = users.to_h { |user| [user.id, user] }

# Step 5
students.each do |student|
  puts users[student.user_id].username
end
```

#### Using ORM API

Databases and ORMs have recognized the need to work with relations efficiently. Rails has `includes` whereas Django has `prefetch_related` for fetching related records efficiently.

```ruby
students = Student.where(section: 'A').includes(:user)

students.each do |student|
  puts student.user.username
end
```

Phew! That was shorter and is just as fast as the hash approach.
