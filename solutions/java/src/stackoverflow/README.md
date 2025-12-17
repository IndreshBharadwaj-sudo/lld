# StackOverflow System (LLD)

## Problem Statement

Design and implement a simplified StackOverflow-like Q&A platform. The system should allow users to post questions and answers, vote on them, comment, tag questions, and track user reputation.

---

## Requirements

- **User Management:** Users can ask questions, answer, comment, and vote.
- **Questions & Answers:** Users can post questions and answers. Each question can have multiple answers, and one accepted answer.
- **Voting:** Users can upvote or downvote questions and answers. Reputation is updated accordingly.
- **Comments:** Users can comment on both questions and answers.
- **Tags:** Questions can be tagged for categorization.
- **Reputation:** Users gain or lose reputation based on votes and accepted answers.
- **Accepted Answer:** The question author can mark one answer as accepted.

---

## Core Entities

- **User:** Represents a user, tracks reputation and user details.
- **Question:** Represents a question, holds answers, comments, tags, votes, and accepted answer.
- **Answer:** Represents an answer to a question, holds comments, votes, and accepted status.
- **Comment:** Represents a comment on a question or answer.
- **Tag:** Represents a tag for categorizing questions.
- **Vote:** Represents a vote (upvote/downvote) by a user on a question or answer.
- **VoteType:** Enum for UPVOTE and DOWNVOTE.
- **Votable (interface):** For entities that can be voted on.
- **Commentable (interface):** For entities that can be commented on.

---

## Class Design

## UML Class Diagram

![](../../../../uml-diagrams/class-diagrams/stackoverflow-class-diagram.png)

### 1. User
- **Fields:** id, name, reputation, etc.
- **Methods:** updateReputation(int delta), getReputation(), etc.

### 2. Question
- **Fields:** id, title, content, author, creationDate, answers, comments, tags, votes, acceptedAnswer
- **Methods:** addAnswer(Answer), acceptAnswer(Answer), vote(User, VoteType), getVoteCount(), addComment(Comment), getComments(), etc.

### 3. Answer
- **Fields:** id, content, author, question, isAccepted, creationDate, comments, votes
- **Methods:** vote(User, VoteType), getVoteCount(), addComment(Comment), getComments(), markAsAccepted(), etc.

### 4. Comment
- **Fields:** id, content, author, creationDate

### 5. Tag
- **Fields:** name

### 6. Vote
- **Fields:** voter, type (VoteType)
- **Methods:** getVoter(), getType()

### 7. VoteType
- Enum: UPVOTE, DOWNVOTE

### 8. Votable (interface)
- **Methods:** vote(User, VoteType), getVoteCount()

### 9. Commentable (interface)
- **Methods:** addComment(Comment), getComments()

---

## Design Patterns Used

- **Strategy Pattern:** For voting and commenting behaviors via interfaces.
- **Observer Pattern:** (Conceptually) for reputation updates on votes and accepted answers.

---

## Example Usage

```java
User alice = new User("Alice");
Question q = new Question(alice, "What is Java?", "Explain Java basics.", Arrays.asList("java", "basics"));
User bob = new User("Bob");
Answer a = new Answer(bob, q, "Java is a programming language.");
q.addAnswer(a);
q.vote(bob, VoteType.UPVOTE);
a.vote(alice, VoteType.UPVOTE);
q.acceptAnswer(a);
```

---

## Demo

See `StackOverflowDemo.java` for a sample usage of the StackOverflow system.

---

## Extending the Framework

- **Add new features:** Such as badges, user profiles, or advanced search.
- **Add new vote types:** Extend `VoteType` and update logic in `vote()` methods.
- **Add moderation:** Implement admin/moderator roles for content management.

---


≥≥ 📌 StackOverflow – Clean Low Level Design (LLD)
1️⃣ High-level Design Philosophy

Posts and Answers are first-class entities

Users do not own posts (no containment)

Posts reference users via authorId

StackOverflow acts as an orchestrator/facade, not a data store

Repositories manage persistence & retrieval

Optimized for read-heavy workloads

2️⃣ Clean UML Diagram (Mermaid – README compatible)
classDiagram
    class StackOverflow {
        +askQuestion(userId, title, content)
        +answerQuestion(userId, questionId, content)
        +vote(postId, userId, voteType)
        +acceptAnswer(questionId, answerId)
    }

    class User {
        +userId
        +name
        +reputation
    }

    class Post {
        <<abstract>>
        +postId
        +content
        +authorId
        +voteCount
        +createdAt
    }

    class Question {
        +title
        +acceptedAnswerId
    }

    class Answer {
        +questionId
        +isAccepted
    }

    class UserRepository {
        +getUser(userId)
    }

    class PostRepository {
        +getQuestion(questionId)
        +getAnswers(questionId)
        +savePost(post)
    }

    StackOverflow --> UserRepository
    StackOverflow --> PostRepository

    Post <|-- Question
    Post <|-- Answer

    Question "1" --> "*" Answer : has
    Post --> User : authorId


✅ This UML is clean, scalable, and interview-ready

3️⃣ Why NOT store posts inside User?
❌ Bad Design: User → List<Post>

Problems:

StackOverflow is read-heavy, not user-centric

Common queries:

“Show answers for this question”

“Sort answers by votes”

Traversing User → Posts → Answers is inefficient

Deleting a user should NOT delete posts

Breaks aggregate boundaries

📌 Posts must exist independently of user lifecycle

4️⃣ Why Posts SHOULD reference Users
✅ Correct Design: Post → authorId

Benefits:

Fast read path

Clean ownership semantics

Supports [deleted user] scenarios

Mirrors real StackOverflow DB schema

Avoids deep object traversal

5️⃣ Why StackOverflow should NOT store all posts/users
❌ Bad Design: StackOverflow contains all Users, Posts, Answers

Problems:

Becomes a God Object

Hard to scale

Hard to shard

Hard to test

Violates Single Responsibility Principle

6️⃣ Correct Role of StackOverflow Class
✅ StackOverflow as a Facade / Orchestrator
StackOverflow
- Coordinates workflows
- Enforces business rules
- Delegates data access to repositories
- Does NOT store data

Example:
askQuestion(userId, title, content)
→ validate user
→ create Question
→ save via PostRepository

7️⃣ Aggregate Root Boundaries (Important for Interviews)
Aggregate	Responsibility
User	Identity, reputation
Post (Question/Answer)	Content, votes, author
StackOverflow	Orchestration
Repository	Persistence

📌 Never require loading User aggregate to understand a Post

8️⃣ Read-heavy Optimization Justification

StackOverflow traffic pattern:

~90% reads

~10% writes

Design optimizes:

Question → Answers → Author

Vote-based sorting

Accepted answer lookup

9️⃣ How this maps to Database Design
USERS
- user_id (PK)
- name
- reputation

POSTS
- post_id (PK)
- type (QUESTION / ANSWER)
- content
- author_id (FK)
- vote_count
- created_at

QUESTIONS
- post_id (PK, FK)
- title
- accepted_answer_id

ANSWERS
- post_id (PK, FK)
- question_id (FK)
- is_accepted


✅ Matches real StackOverflow schema

🔑 Interview-ready Summary (Copy-Paste)

“Posts and answers are first-class aggregates that reference users via IDs. Users do not own posts. StackOverflow acts as an orchestrator, while repositories manage persistence. This design optimizes read-heavy access patterns, avoids tight coupling, and scales cleanly.”

10️⃣ Common Follow-up Questions (Prep)
Question	Expected Direction
How do you handle deleted users?	Keep posts, show [deleted]
How do you handle concurrency in voting?	Atomic counters / optimistic locking
How do you scale answers?	Pagination, caching
How do you enforce one accepted answer?	Transactional update
✅ TL;DR (One-liner)

Best design:

Posts reference users, repositories manage data, StackOverflow orchestrates logic.

≥≥ In stack overflow design should I keep questions comments array inside user class or only keep references of users in the posts

Short answer:
❌ Do NOT keep questions/comments arrays inside User.
✅ Keep only user references (authorId) inside Post / Comment.

Now let’s explain this clearly and interview-correctly, so you can remember it forever.

✅ Correct Design (What you SHOULD do)
Relationships

Post (Question / Answer) → references User via authorId

Comment → references User via authorId

User → does NOT contain arrays of posts/comments

User
  - userId
  - name
  - reputation

Post
  - postId
  - content
  - authorId   → User

Comment
  - commentId
  - content
  - authorId   → User

❌ Incorrect Design (What you should NOT do)
User
  - List<Question>
  - List<Answer>
  - List<Comment>


This looks intuitive but is architecturally wrong.

🧠 Why NOT store posts/comments inside User?
1️⃣ StackOverflow is post-centric, not user-centric

Most common reads:

“Show answers for this question”

“Show comments under this post”

“Sort answers by votes”

❌ None of these start from User
✅ All start from Post

If posts live inside users → you must scan users → terrible design.

2️⃣ Users can be deleted; posts must survive

Real StackOverflow behavior:

User deleted → posts remain ([deleted user])

If posts are owned by user:

Delete user → posts disappear ❌

Or leave dangling references ❌

Keeping authorId solves this cleanly.

3️⃣ Violates Aggregate Root principle (DDD)

In proper LLD:

User is an aggregate

Post is an independent aggregate

Rule:

One aggregate should not own another aggregate’s lifecycle.

Storing posts inside users breaks this rule.

4️⃣ Causes performance & memory issues

Imagine a top contributor:

10,000 answers

5,000 comments

Loading a User object would:

Pull huge collections into memory

Slow down unrelated flows

❌ Bad for scalability
✅ Repositories + queries are better

5️⃣ Makes persistence & sharding harder

Real systems shard by:

Posts

Time

Votes

They do not shard by users containing everything.

Your design should map cleanly to DB tables.

≥≥≥≥ Should Post contain User user object or just String userId and then I should get user from userRegistry

Short answer (interview-correct):
👉 Post should store userId, NOT the full User object.
👉 Fetch the User via UserRepository / UserRegistry when needed.

Now let’s explain why, in a way that will impress an interviewer and also help you design clean systems.

✅ Correct Design (Recommended)
class Post {
    String postId;
    String content;
    String authorId;   // userId only
}

class UserRepository {
    User getUser(String userId);
}

❌ Incorrect Design (Tight coupling)
class Post {
    User author;   // ❌ bad
}

🧠 Why Post should NOT contain User object
1️⃣ Loose coupling (MOST IMPORTANT)

If Post contains User:

Any change in User affects Post

Leads to tight coupling

Harder to evolve independently

With userId:

Post is independent

User lifecycle is separate

2️⃣ Aggregate boundaries (DDD principle)

User is an aggregate root

Post is another aggregate root

Rule:

One aggregate should reference another by ID, not by object.

This is textbook DDD and interviewers love this answer.

3️⃣ Performance & memory efficiency

If Post contains User:

Loading a post loads the full user

Wasteful in read-heavy systems

Using userId:

Fetch user only when required

Allows caching & lazy loading

4️⃣ Real-world scenario: deleted users

StackOverflow behavior:

User deleted → posts remain

Author shown as [deleted]

This is easy with:

authorId = "deleted"


Hard with:

User user = null ❌

5️⃣ Serialization & persistence

Storing objects inside objects:

Complicates DB schema

Causes circular references

Makes caching harder

Storing IDs:

Clean DB tables

Clean JSON

Clean APIs

✅ When is it OK to store User inside Post?

Only in very limited cases:

In-memory DTOs

Read models

View projections

Example:

class PostView {
    Post post;
    User author;   // OK here
}


📌 This is NOT your domain model.

🧠 Interview-ready explanation (memorize this)

“Post and User are separate aggregates.
A post should reference its author by userId only, and the full User should be fetched via a repository when needed.
This keeps the model loosely coupled, scalable, and consistent with real-world systems like StackOverflow.”

≥≥≥≥≥ can i keep comments inside post in stackoverlow design abstract class Post implements Votable{
    private int postId;
    private String content;
    private String authorUserId;
    private long createdAt;
    private Map<String,VoteType> voters;
    private List<Comment> comments;

Yes — you can keep comments inside Post, and in fact this is the correct design for StackOverflow-like systems, as long as you understand the trade-offs.

Let’s go through this cleanly, interview-ready, and with real-world reasoning.

✅ Correct: Keeping comments inside Post
abstract class Post implements Votable {
    private int postId;
    private String content;
    private String authorUserId;
    private long createdAt;
    private Map<String, VoteType> voters;
    private List<Comment> comments;
}


This design is valid and commonly used.

🧠 Why this design makes sense
1️⃣ Domain truth (most important)

On StackOverflow:

Comments do not exist independently

A comment always belongs to exactly one post

If the post is deleted → comments are deleted

This is a strong ownership relationship.

➡️ That means Post is the aggregate root, and Comment is a child entity.

2️⃣ Correct Aggregate modeling (DDD)

Post = aggregate root

Comment = entity inside aggregate

Rule:

All access to Comment should go through Post.

This matches your design.

3️⃣ Simplifies invariants & rules

Examples:

Max comments per post

Comment ordering

Comment deletion rules

Comment visibility

All can be enforced inside Post.

4️⃣ Avoids unnecessary global registries

If comments were stored separately:

You’d need CommentRepository

Extra lookups

More coordination

For StackOverflow scale, comments are always accessed via post, so embedding is correct.

🧠 Interview-ready explanation (memorize)

“In StackOverflow, comments are owned by posts and don’t exist independently, so modeling them inside Post is correct.
Post acts as the aggregate root, and Comment is a child entity.
We store only authorUserId in comments and control access via the post to preserve invariants.”

This answer is exactly what interviewers want.

✅ Final verdict
Design Choice	Verdict
Post contains List<Comment>	✅ Correct
Comment has postId only	❌ Not needed
Post is aggregate root	✅ Correct
Comment has User object	❌ Wrong

≥≥≥≥ Lets say I'm designing stack overflow for lld, I am using post registry and user registry to get users and posts, now my question is should I keep String userId and String postId inside vote or not? Alternatively I can just make Vote having voteId etc. and posts have a vote array and user have a post array

This is an excellent LLD question — you’re thinking at the right abstraction level for SDE-2+.

Short answer (interview-correct)

👉 Yes, Vote should store userId and postId.


≥≥≥≥≥ So if a user is deleted I should let the user registry take care of it and not make the user id null in vote class?

Yes — exactly right, and this is a very strong design instinct 👍
You should NOT set userId = null in Vote.

👉 User deletion must be handled by UserRegistry / UserService, not by mutating votes.
2️⃣ Violates immutability of historical facts

A vote is a historical event:

“User X voted on Post Y at time T”

Deleting a user should not rewrite history.

StackOverflow, Reddit, GitHub all follow this.

3️⃣ Breaks analytics & moderation

Examples:

Vote fraud detection

Abuse patterns

Rate limiting

Rollback / disputes

All rely on user identity remaining intact.