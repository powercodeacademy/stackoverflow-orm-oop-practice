# StackOverflow: ORMs and OOP - Bringing it all Together

## Learning Goals

- Understand what an Object Relational Mapper(ORM) is
- Gain ability to implement characteristics of an ORM when using a relational database management system (RDBMS) in a ruby program
- Practice implementing ORM methods for multiple related classes

## Instructions

This lab involves building a basic ORM for a StackOverflow-style application with `Question` and `Answer` objects. The `Question` class defined in `lib/question.rb` and the `Answer` class defined in `lib/answer.rb` implement behaviors of a basic ORM.

### **Environment**

Our environment is going to be a single point of requires and loads. It is also going to define a constant, `DB`, whose sole responsibility is setting up and maintaining connection to our application's database.

- `DB = {:conn => SQLite3::Database.new("db/stackoverflow.db")}` `DB` is set equal to a hash, which has a single key, `:conn`. The key, `:conn`, will have a value of a connection to a sqlite3 database in the db directory.

However, in our `spec_helper`, which is our testing environment, we're going to redefine the value of that key (not of the constant) to point to an in-memory database. This will allow our tests to run in isolation of our production database. Whenever we want to refer to the application's connection to the database, we will simply rely on `DB[:conn]`.

## Solving The Lab: The Spec Suite

### Question Class

#### Attributes

The first test is concerned solely with making sure that our questions have all the required attributes and that they are readable and writable.

The `#initialize` method accepts a hash or keyword argument value with key-value pairs as an argument. key-value pairs need to contain id, title, content, and views.

#### `#save`

This spec ensures that given an instance of a question, simply calling `save` will insert a new record into the database and return the instance.

#### `.create`

This is a class method that should:

- Create a new row in the database
- Return a new instance of the `Question` class

Think about how you can re-use the `#save` method to help with this one.

#### `.new_from_db`

This is an interesting method. Ultimately, the database is going to return an array representing a question's data. We need a way to cast that data into the appropriate attributes of a question. This method encapsulates that functionality. You can even think of it as `new_from_array`. Methods like this, that return instances of the class, are known as constructors, just like `.new`, except that they extend the functionality of `.new` without overwriting `initialize`.

#### `.all`

This class method should return an array of `Question` instances for every record in the `questions` table.

#### `.find_by_title(title)`

The spec for this method will first insert a question into the database and then attempt to find it by calling the find_by_title method. The expectations are that an instance of the question class that has all the properties of a question is returned, not primitive data.

Internally, what will the `.find_by_title` method do to find a question; which SQL statement must it run? Additionally, what method might `.find_by_title` use internally to quickly take a row and create an instance to represent that data?

**Note**: You may be tempted to use the `Question.all` method to help solve this one. While we applaud your intuition to try and keep your code DRY, in this case, reusing that code is actually not the best approach. Why? Remember, with `Question.all`, we're loading all the records from the `questions` table and converting them to an array of Ruby objects, which are stored in our program's memory. What if our `questions` table had 10,000 rows? That's a lot of extra Ruby objects! In cases like these, it's better to use SQL to only return the questions we're looking for, since SQL is extremely well-equipped to work with large sets of data.

#### `.find(id)`

This class method takes in an ID, and should return a single `Question` instance for the corresponding record in the `questions` table with that same ID. It behaves similarly to the `.find_by_title` method above.

#### `#answers`

This instance method should return an array of `Answer` instances that belong to the question. It should use the `Answer.find_all_by_question_id` method to find all answers associated with the question's ID.

### Answer Class

#### Attributes

The Answer class should have all the required attributes that are readable and writable.

The `#initialize` method accepts a hash or keyword argument value with key-value pairs as an argument. key-value pairs need to contain id, content, votes, and question_id.

#### `#save`

This spec ensures that given an instance of an answer, simply calling `save` will insert a new record into the database and return the instance.

#### `.create`

This is a class method that should:

- Create a new row in the database
- Return a new instance of the `Answer` class

Think about how you can re-use the `#save` method to help with this one.

#### `.new_from_db`

This method takes an array representing an answer's data from the database and creates a new Answer instance with the appropriate attributes.

#### `.find_all_by_question_id(question_id)`

This class method takes in a question_id and should return an array of `Answer` instances that belong to that question. This is how we establish the relationship between questions and their answers.

## Database Schema

The database should have two tables:

### Questions Table

- `id` (INTEGER PRIMARY KEY)
- `title` (TEXT)
- `content` (TEXT)
- `views` (INTEGER)

### Answers Table

- `id` (INTEGER PRIMARY KEY)
- `content` (TEXT)
- `votes` (INTEGER)
- `question_id` (INTEGER) - foreign key referencing questions.id

## Bonus Methods

In addition to the methods described above, there are a few bonus methods if you'd like to build out more features. The tests for these methods may be commented out in the spec file. Comment them back in to run the tests for these methods.

### `.find_or_create_by`

This method takes keyword arguments and returns an existing record if found, or creates a new one if not found.

### `#update`

The spec for this method will create and insert a record, and afterwards, it will change an attribute of the instance and call update. The expectations are that after this operation, the database record is updated with the new values.

The SQL you'll need to write for this method will involve using the `UPDATE` keyword.

### `#save` (enhanced)

Wait, didn't we already make a `#save` method? Well, yes, but we can expand its functionality! You could change it so that it handles these two cases:

- If called on an instance that doesn't have an ID assigned, insert a new row into the database, and return the saved instance.
- If called on an instance that _does_ have an ID assigned, use the `#update` method to update the existing record in the database, and return the updated instance.

Based on: https://github.com/learn-co-curriculum/phase-3-orms-bringing-it-all-together
