# 4. DATABASES IN FLASK - READING, UPDATING AND DELETING

### **Interacting with a Database**

Your database has a number of readers who subscribed to your book club and some books you already assigned to be read. Also some of your readers wrote reviews about the books and some of them might have some annotations made while reading their books on an eReader. The schema representing the database is in the image on the right.

What can you do with the database?

Say you want to list all the books you suggested or list all the subscribed readers. Or let each subscriber see only the reviews they wrote. When new people subscribe to your web service, you need to add them to your database, or when they unsubscribe delete them. If you made a mistake in changing your database, you probably want to undo the changes or ‘rollback’ to the previous correct state. When your subscribers change their e-mail, you need to update your database. Or you need to filter out the books your club read in the year 2019 and only calculate average ratings of those. These are all the most common interactions with a database, and in this lesson, you will learn how to perform them in Flask-SQLAlchemy.

Throughout this exercise we will provide you with a database containing some entries for you to query, but you will learn how to add more entries, and remove some, on the way.

Can’t wait? Let’s go.

![Untitled](4%20DATABASES%20IN%20FLASK%20-%20READING,%20UPDATING%20AND%20DELET%2074e0c1dfb1514e3292482930ee446d1b/Untitled.png)

### **Queries: query.all() and query.get()**

Querying a database table with Flask SQLAlchemy is done through the `query` property of the `Model` class. To get all entries from a model called `TableName` we run `TableName.query.all()`. Often you know the primary key (unique identifier) value of entries you want to fetch. To get an entry with some primary key value `ID` from model `TableName` you run: `TableName.query.get(ID)`.

For example, to get all the entries from the `Reader` table we do the following:

```python
readers = Reader.query.all()
```

Similarly, to get a reader with `id = 123` we do the following:

```python
reader = Reader.query.get(123)
```

We assign the result of the `.get()` method to a variable because through that variable we can access the entry’s attributes. For example:

```python
reader = Reader.query.get(450)
print(reader.name)
```

Now you see the amazing convenience of using ORM: database tables are simply treated as Python classes and database entries are Python objects. For example, you can easily use a `for` loop to loop through all the readers and print their name:

```python
readers = Reader.query.all()
for reader in readers:
    print(reader.name)
```

In the image on the right, you can see some entries we already inserted for you in the database, so you can query it.

playground.py

```python
from app import db, Book, Reader, Review, Annotation

#query all the readers from the Reader model
readers = Reader.query.all()
print(readers)

#get an entry with id = 123 
reader = Reader.query.get(123)
print(reader)

#reader with id = 450
reader = Reader.query.get(450)
print("Reader with id = ", reader.id, "is called", reader.name)

#Loop through all the readers and print their e-mails
print("\nPrint all the readers in a loop:")
for reader in readers:
  print(reader.email)

#or inline
#[print(reader.email) for reader in readers]

print("\nCheckpoint1: fetching all the reviews")
reviews = Review.query.all()

print("\nCheckpoint2: looping through all the reviews and printing their text")
for review in reviews:
   print(review.text)

print("\nCheckpoint3: fetching a book with id = 12 using the get() function")
book_1 = Book.query.get(12)
```

![Untitled](4%20DATABASES%20IN%20FLASK%20-%20READING,%20UPDATING%20AND%20DELET%2074e0c1dfb1514e3292482930ee446d1b/Untitled%201.png)

![Untitled](4%20DATABASES%20IN%20FLASK%20-%20READING,%20UPDATING%20AND%20DELET%2074e0c1dfb1514e3292482930ee446d1b/Untitled%202.png)

### **Queries: retrieve related objects**

The models we declared contain relationships. Readers write multiple reviews and have multiple annotations. Similarly, books have multiple reviews and multiple annotations. In this exercise we will see how we can fetch all the reviews made by a reader, and to fetch the author of a review.

### **Fetching many objects**

We fetch related objects of some object by accessing its attribute defined with `.relationship()`. For example, to fetch all reviews of a reader with `id = 123` we do the following:

```python
reader = Reader.query.get(123)
reviews_123 = reader.reviews.all()
```

Note that `reviews` attribute was defined as a column in the model for `Reader` using the `.relationship()` method (see **app.py** to remind yourself).

### **Fetching one object**

For `Review` object we can fetch its authoring `Reader` through the `backref` field specified in `Reader`’s `.relationship()` attribute. For example, to fetch the author of review `id = 111` we do the following:

```python
review = Review.query.get(111)
reviewer_111 = review.reviewer
```

You can modify the examples above so not to have temporary variables (`reader` and `review_1`) by chaining the operations:

```python
reviews_123 = Reader.query.get(123).reviews.all()
```

and

```python
reviewer_111 = Review.query.get(111).reviewer
```

Notice the subtle difference between the examples above. The first needs `.all()` because one reader can have many reviews. In the second example, we do not use `.all()` since each review is associated with only one reader. That is our one-to-many relationship.

playground.py

```python
from app import db, Book, Reader, Review, Annotation

#Fetching 'many' objects
reader = Reader.query.get(123) #fetch a reader with id = 123
reviews_123 = reader.reviews.all() #fetch all reviews made by reader wiith id = 123
[print(review.id) for review in reviews_123]
#check the image on the right - Ann Adams (id = 123, wrote reviews with ids 111 and 113)

#fetching 'one' object
review = Review.query.get(111) #fetch a review with id = 111
reviewer_111 = review.reviewer #get the reviewer for review with id = 111. There's only one!
print("The author of [", review, "] is", reviewer_111)

#By chaining we avoid using temporary variables
reviews_123 = Reader.query.get(123).reviews.all() #same result as line 5
reviewer_111 = Review.query.get(111).reviewer #same result as line 10

print("\nCheckpoint 1: fetch all the reviews made for a book with id = 13.")
book_13 = Book.query.get(13).reviews.all()
[print(book.id) for book in book_13]

print("\nCheckpoint 2: fetch all the annotations made for a book with id = 19.")
book_19_an = Book.query.get(19).annotations.all()
[print(annotation.id) for annotation in book_19_an]

print("\nCheckpoint 3: fetch the reader who owns the annotation with `id = 331.`")
author_331 = Annotation.query.get(331).author
print(author_331)
```

![Untitled](4%20DATABASES%20IN%20FLASK%20-%20READING,%20UPDATING%20AND%20DELET%2074e0c1dfb1514e3292482930ee446d1b/Untitled%203.png)

### **Queries: filtering**

Often times you don’t want to retrieve all the entries from a table but select only those that satisfy some criterion. Criteria are usually based on the values of the table’s columns. To filter a query, SQLAlchemy provides the `.filter()` method.

For example, to select books from a specific year from the `Book` table we use the following command:

```python
Book.query.filter(Book.year == 2020).all()
```

Notice the additional `.all()` method. `.filter()` returns a `Query` object that needs to be further refined. This can be done by using several additional methods like `.all()` that returns a list of all results, `.count()` that counts the number of fetched entries, or `.first()` that returns only one result, namely the first one.

```python
Book.query.filter(Book.year == 2020).first()
```

Multiple criteria may be specified as comma separated and the interpretation of a comma is a Boolean `and`:

```python
Review.query.filter(Review.stars <= 3, Review.book_id == 1).all()
```

This query will return all entries in the `Review` table that have fewer than 3 stars for the book with `id = 1`.

Note: there is also the `.filter_by()` method that uses only a simple attribute-value test for filtering.

playground.py

```python
from app import Book, Reader, Review, Annotation

#select books from the year 2020
book_2020 = Book.query.filter(Book.year == 2020).all()
print("All the suggested books in the year 2020:")
[print(book) for book in book_2020]

#instead of all books suggested in 2020, fetch only the first one
book_2020_first = Book.query.filter(Book.year == 2020).first()
print("\nThe first book fetched from the year 2020: ", book_2020_first)

#you can specify multiple criteria for filtering
rev_3_boook13 = Review.query.filter(Review.stars <= 3, Review.book_id == 13).all()
print("\nThe review of 3 stars or lower written for a book with id = 13: ", rev_3_boook13)

#Checkpoint 1: fetching all the readers with "Adams" surname
adams = Reader.query.filter(Reader.surname == "Adams").all()
[print(person) for person in adams]

#Checkpoint 2: fetching the first book dating prior to the year 2019
books_pre2019 = Book.query.filter(Book.year <= 2019).first()
[print(book) for book in books_pre2019]
```

![Untitled](4%20DATABASES%20IN%20FLASK%20-%20READING,%20UPDATING%20AND%20DELET%2074e0c1dfb1514e3292482930ee446d1b/Untitled%204.png)

### **Queries: more advanced filtering**

Flask-SQLALchemy allows more complex queries and operations such as checking whether a column starts, or ends, with some string. One can also order retrieved queries by some criterion. There are many more possible queries, but here we cover only some of them.

For example, to retrieve e-mails that end with `edu` we do:

```python
education = Reader.query.filter(Reader.email.endswith('edu')).all()
```

To retrieve all the readers with e-mails that contain a ‘.’ before the ‘@’ symbol we use `.like()`:

```python
emails = Reader.query.filter(Reader.email.like('%.%@%')).all()
```

You might recognize the `like` operator from SQL. It is used to search for a specified pattern in a column. The wildcard `%` represents zero, one, or multiple characters.

In the two examples above, we used methods on the column of the table (SQLAlchemy’s `ColumnElement`).

To order books by year we use the `.order_by()` method on `Query`:

```python
ordered_books = Book.query.order_by(Book.year).all()
```

We suggest checking the SQLAlchemy Core + ORM documentation to see other querying options.

app.py

```python
from app import db, Book, Reader, Review

#retrieve all reader with .edu e-mails
education = Reader.query.filter(Reader.email.endswith('edu')).all()
print(education)

#retrieve all readers with e-mails that contain a . before the @ symbol
emails = Reader.query.filter(Reader.email.like('%.%@%')).all()
print("\nReaders with e-mails having a . before the @ symbol:")
for e in emails:
  print(e.email)

#order all books by year
ordered_books = Book.query.order_by(Book.year).all()
print("\nBooks ordered by year:")
for book in ordered_books:
  print(book.title, book.year)

print("\nCheckpoint 1: your code here below:")
s_names = Reader.query.filter(Reader.surname.endswith('s')).all()
#print(s_names)

print("\nCheckpoint 2: your code here below:")
sample_emails = Reader.query.filter(Reader.email.like('%@sample%')).all()
#print(sample_emails)

print("\nCheckpoint 3: your code here below:")
ordered_reviews = Review.query.order_by(Review.stars).all()
print(ordered_reviews)
```

```bash
[Reader ID: 345, email: sam.adams@example.edu, Reader ID: 753, email: tom.fox@sample.edu]

Readers with e-mails having a . before the @ symbol:
sam.adams@example.edu
sam.smalls@example.com
earl.grey@sample.com
tom.fox@sample.edu

Books ordered by year:
Demian 2018
The Book of Why 2019
Guns, Germs and Steel 2019
Hundred years of solitude 2020
The Stranger 2020

Checkpoint 1: your code here below:

Checkpoint 2: your code here below:

Checkpoint 3: your code here below:
[Review ID: 113, 2 stars 13, Review ID: 112, 3 stars 12, Review ID: 116, 3 stars 19, Review ID: 119, 3 stars 18, Review ID: 114, 4 stars 13, Review ID: 115, 4 stars 13, Review ID: 118, 4 stars 12, Review ID: 111, 5 stars 12, Review ID: 117, 5 stars 19]
```

### **Session: add and rollback**

A set of operations such as addition, removal, or updating database entries is called a database transaction. A *database session* consists of one or more transactions. The act of committing ends a transaction by saving the transactions permanently to the database. In contrast, *rollback* rejects the pending transactions and changes are not permanently saved in the database.

In Flask-SQLAlchemy, a database is changed in the context of a session, which can be accessed as the `session` attribute of the database instance. An entry is added to a session with the `add()` method. The changes in a session are permanently written to a database when `.commit()` is executed.

For example, we create new readers and would like to add them to our database:

```python
from app import db, Reader
new_reader1 = Reader(name = "Nova", surname = "Yeni", email = "nova.yeni@example.com")
new_reader2 = Reader(name = "Nova", surname = "Yuni", email = "nova.yeni@example.com")
new_reader3 = Reader( name = "Tom", surname = "Grey", email = "tom.grey@example.edu")
```

Note that we didn’t specify the primary key `id` value. Primary keys don’t have to be specified explicitly, and the values are automatically generated after the transaction is committed.

Adding each new entry to the database has the same pattern:

```python
db.session.add(new_reader1)
try:
    db.session.commit()
except:
    db.session.rollback()
```

Notice that we surrounded `db.session.commit()` with a try-except block. Why did we do that? If you look more carefully, `new_reader1` and `new_reader2` have the same e-mail, and when we declared the `Reader` model, we made the e-mail column unique (see the **app.py** file). As a consequence, we want to undo the most recent addition to the transaction by using `db.session.rollback()` and continue with other additions without interruption.

playground.py

```python
from app import db, Reader #notice we import db here as well
import add_data #we use this script to recreate the database, put all the entries so every time you run this script
                #you get the same result

#creating new readers
new_reader1 = Reader(name = "Nova", surname = "Yeni", email = "nova.yeni@sample.com")
new_reader2 = Reader(name = "Nova", surname = "Yuni", email = "nova.yeni@sample.com")
new_reader3 = Reader(name = "Tom", surname = "Grey", email = "tom.grey@example.edu")

print("Before addition: ")
for reader in Reader.query.all():
  print(reader.id, reader.email)

print("\nNote that before committing, the id of the new readers is: ", new_reader1.id, "\n")

#adding the first reader - the commit should succeed
db.session.add(new_reader1)
try:
    db.session.commit()
    print("Commit succeded.", new_reader1, "added to the database permanently. The exception was not raised.\n")
except:
    db.session.rollback()

#adding the second reader - the commit should fail because e-mails should be unique
db.session.add(new_reader2)  
try:
    db.session.commit()
except Exception as ex:
    print("The commit of", new_reader2,"didn't succeed. Duplicate primary key values. We will empty the current session.\n")
    print("The error is the following:", ex)
    db.session.rollback() 

#adding the third reader - the commit should succeed
db.session.add(new_reader3)  
try:
    db.session.commit()
    print("Commit succeded.", new_reader3, "added to the database permanently. The exception was not raised.\n")
except Exception as ex:
    db.session.rollback() 

print("\nNote that after committing, the id of the new readers is now: ", new_reader1.id, "\n")

#print all the readers after the addition, and we see nova.yeni@sample.com there, but not twice
for reader in Reader.query.all():
  print(reader.id, reader.email)
print("\nThe new readers Nova Yeni and Tom Grey are in the database. Notice that Nova Yeni doesn't appear twice.\n")

print("\nCheckpoint 1: create a new_reader:")
new_reader = Reader(name = "Peter", surname = "Johnson", email = "peter.johnson@example.com")

print("\nCheckpoint 2: add the new reader to the database:")
#your code here
db.session.add(new_reader)

print("\nCheckpoint 3: commit and rollback if exception is raised:")
try:
  db.session.commit()
except:
  db.session.rollback()
```

add_data.py

```python
from app import db, Book, Reader, Review, Annotation
import os

if os.path.exists('myDB.db'):
  os.remove('myDB.db')
  
db.create_all()

r1 = Reader(id = 123, name = 'Ann', surname = 'Adams', email = 'ann.adams@example.com')
r2 = Reader(id = 345, name = 'Sam', surname = 'Adams', email = 'sam.adams@example.com')
r3 = Reader(id = 450, name = 'Kim', surname = 'Smalls', email = 'kim.smalls@example.com')
r4 = Reader(id = 568, name = 'Sam', surname = 'Smalls', email = 'sam.smalls@example.com')
r5 = Reader(id = 753, name = 'Tom', surname = 'Fox', email = 'tom.fox@sample.edu')
r6 = Reader(id = 653, name = 'Earl', surname = 'Grey', email = 'earl.grey@sample.com')
db.session.add(r1)
db.session.add(r2)
db.session.add(r3)
db.session.add(r4)
db.session.add(r5)
db.session.add(r6)
try:
  db.session.commit()
except Exception:
  db.session.rollback()

b1 = Book(id = 12, title = 'Hundred years of solitude', author_name = 'Gabriel', author_surname = 'Garcia Marquez', month = 'April', year = '2020')
b2 = Book(id = 13, title = 'The Stranger', author_name = 'Albert', author_surname = 'Camus', month = 'May', year = '2020')
b3 = Book(id = 14, title = 'The Book of Why', author_name = 'Judea', author_surname = 'Pearl', month = 'September', year = '2019')
b4 = Book(id = 18, title = 'Demian', author_name = 'Herman', author_surname = 'Hesse', month = 'June', year = '2018')
b5 = Book(id = 19, title = 'Guns, Germs and Steel', author_name = 'Jared', author_surname = 'Diamond', month = 'August', year = '2019')
db.session.add(b1)
db.session.add(b2)
db.session.add(b3)
db.session.add(b4)
db.session.add(b5)
try:
  db.session.commit()
except Exception:
  db.session.rollback()

rev1 = Review(id = 111, text = 'This book is amazing...', stars = 5, reviewer_id = r1.id, book_id = b1.id)
rev2 = Review(id = 112, text = 'The story is hard to follow.', stars = 3, reviewer_id = r2.id, book_id = b1.id)
rev3 = Review(id = 113, text = 'The story is quite complicated.', stars = 2, reviewer_id = r1.id, book_id = b2.id)
rev4 = Review(id = 114, text = 'Albert Camus is an amazing writer who really understands the society.', stars =4, reviewer_id = r2.id, book_id = b2.id)
rev5 = Review(id = 115, text = 'The book is simply written and a rather quick read, but the depth Camus manages to convey through this simplicity is astounding.', stars =4, reviewer_id = r2.id, book_id = b2.id)
rev5 = Review(id = 116, text = 'Despite the fascinating subject matter I found this book a bit dry.', stars =3, reviewer_id = r4.id, book_id = b5.id)
rev6 = Review(id = 117, text = 'I liked this book, and it taught me a bunch of things.', stars =5, reviewer_id = r3.id, book_id = b5.id)
db.session.add(rev1)
db.session.add(rev2)
db.session.add(rev3)
db.session.add(rev4)
db.session.add(rev5)
db.session.add(rev6)
try:
  db.session.commit()
except Exception:
  db.session.rollback()

ann1 = Annotation(id = 331, text = 'Human history is a function of geography.', reviewer_id = r4.id, book_id = b5.id)
ann2 = Annotation(id = 332, text = 'I opened myself to the gentle indifference of the world.', reviewer_id = r1.id, book_id = b2.id)
ann3 = Annotation(id = 333, text = 'Everything is true, and nothing is true!', reviewer_id = r2.id, book_id = b2.id)
ann4 = Annotation(id = 334, text = 'Good that you ask -- you should always ask, always have doubts.', reviewer_id = r3.id, book_id = b4.id)
db.session.add(ann1)
db.session.add(ann2)
db.session.add(ann3)
db.session.add(ann4)
try:
  db.session.commit()
except Exception:
  db.session.rollback()

db.session.close()
```

```bash
Before addition: 
123 ann.adams@example.com
345 sam.adams@example.com
450 kim.smalls@example.com
568 sam.smalls@example.com
653 earl.grey@sample.com
753 tom.fox@sample.edu

Note that before committing, the id of the new readers is:  None 

Commit succeded. Reader ID: 754, email: nova.yeni@sample.com added to the database permanently. The exception was not raised.

The commit of Reader ID: None, email: nova.yeni@sample.com didn't succeed. Duplicate primary key values. We will empty the current session.

The error is the following: (sqlite3.IntegrityError) UNIQUE constraint failed: reader.email
[SQL: INSERT INTO reader (name, surname, email) VALUES (?, ?, ?)]
[parameters: ('Nova', 'Yuni', 'nova.yeni@sample.com')]
(Background on this error at: http://sqlalche.me/e/gkpj)
Commit succeded. Reader ID: 755, email: tom.grey@example.edu added to the database permanently. The exception was not raised.

Note that after committing, the id of the new readers is now:  754 

123 ann.adams@example.com
345 sam.adams@example.com
450 kim.smalls@example.com
568 sam.smalls@example.com
653 earl.grey@sample.com
753 tom.fox@sample.edu
754 nova.yeni@sample.com
755 tom.grey@example.edu

The new readers Nova Yeni and Tom Grey are in the database. Notice that Nova Yeni doesn't appear twice.

Checkpoint 1: create a new_reader:

Checkpoint 2: add the new reader to the database:

Checkpoint 3: commit and rollback if exception is raised:
 
6/10
```

### **Session: updating existing entries**

Sometimes you will need to update a certain column value of an entry in your database. This is rather easy in the context of SQLAlchemy ORM and is done in the same way you would change Python object’s attribute.

The [commands](https://www.codecademy.com/resources/docs/sql/commands) below change the email of a reader with `id=3` and commit the changes to the database:

```python
reader = Reader.query.get(3)
reader.email = “new_email@example.com”
db.session.commit()
```

If you want to undo the update, you can use

```python
db.session.rollback()
```

instead of committing.

playground.py

```python
from app import db, Book, Reader #notice we import db here as well
import add_data

#fetch the reader with id = 123 and change their e-mail
reader = Reader.query.get(123)
print("Before the change:", reader) #print before the change
reader.email = "new.email@example.com"
db.session.commit()
print("After the commit:", Reader.query.get(123)) #print after the change

#rollback
reader = Reader.query.get(345)
print("Rollback example - before the change: ", reader) #print before the change
reader.email = "new.email@example.edu"
db.session.rollback()
print("Rollback example - after the rollback: ", Reader.query.get(345)) #print after the change

print("\nCheckpoint 1: use get to fetch a book entry:")
book_19 = Book.query.get(19)

print("\nCheckpoint 2: modify the month attribute to June:")
book_19.month = "June"

print("\nCommit the change:")
db.session.commit()
```

![Untitled](4%20DATABASES%20IN%20FLASK%20-%20READING,%20UPDATING%20AND%20DELET%2074e0c1dfb1514e3292482930ee446d1b/Untitled%205.png)

### **Session: Removing database entries**

Removing entries is an important aspect of database management and is used often in real-world applications. Users unsubscribe from services, products are removed from web applications, and some relationships are lost (unfollowing other users).

However, before we proceed, we need to be careful about one-to-many relationships. If we remove a reader, we would expect that all the reader’s reviews are also removed from our database. Similarly, removing a book should also remove all the reviews for that book. This procedure is called cascading deletion. Unfortunately, the way we previously declared our `Reader` and `Book` models will not perform the cascading deletion by default. To enable cascading deletions, we did a naive solution in this exercise by changing our models and re-initializing the database. In practice, database migration management is used to update a database schema.

To enable cascade deletions, we changed the models in the **app.py** by adding the `cascade` parameter to the `.relationship()` fields of `Reader` and `Book` models:

```python
reviews = db.relationship('Review', backref='reviewer', lazy='dynamic', cascade = 'all, delete, delete-orphan')
```

In contrast, removing a review does not have any other cascading consequences on `Book` and `Reader` tables. Hence, specifying the cascading deletion option in `Review` is not needed.

Finally, to remove a reader with `id = 753` we use the following command:

```python
db.session.delete(Reader.query.get(753))
```

When you run **playground.py** you see that we print all the readers, all the reviews before and after the deletion. You can notice that when the reader with `id = 753` is deleted, all their reviews are deleted as well. Refer to the image on the right to see the initial entries of some database tables.

playground.py

```python
from app import db, Book, Reader, Review #notice we import db here as well

#let us first print all the readers current in the database (image on the right)
for reader in Reader.query.all():
   print(reader)

#print all the reviews
print("\nAll the current reviews:")
for review in Review.query.all():
  print(review)  

#delete reader with id = 753 (Nova Yeni, nova.yeni@example.com)
db.session.delete(Reader.query.get(753))

#print readers again to validate that the reader is indeed deleted
print("\nReaders after deleting a rader with id = 753")
for reader in Reader.query.all():
   print(reader)

#print reviews to see that all the reviews made by reader id = 753 are deleted
#(see the image on the right)
#print all the reviews
print("\nAll the current reviews:")
for review in Review.query.all():
  print(review)  

#Checkpoint 1:
#your code here below
db.session.delete(Reader.query.get(123))
```

```bash
Reader ID: 123, email: ann.adams@example.com
Reader ID: 345, email: sam.adams@example.com
Reader ID: 450, email: kim.smalls@example.com
Reader ID: 568, email: sam.smalls@example.com
Reader ID: 653, email: tom.grey@example.com
Reader ID: 753, email: nova.yeni@example.com

All the current reviews:
Review: stars:5 reviewer ID: 123
Review: stars:3 reviewer ID: 345
Review: stars:2 reviewer ID: 123
Review: stars:4 reviewer ID: 345
Review: stars:3 reviewer ID: 568
Review: stars:5 reviewer ID: 450
Review: stars:4 reviewer ID: 753
Review: stars:3 reviewer ID: 753

Readers after deleting a rader with id = 753
Reader ID: 123, email: ann.adams@example.com
Reader ID: 345, email: sam.adams@example.com
Reader ID: 450, email: kim.smalls@example.com
Reader ID: 568, email: sam.smalls@example.com
Reader ID: 653, email: tom.grey@example.com

All the current reviews:
Review: stars:5 reviewer ID: 123
Review: stars:3 reviewer ID: 345
Review: stars:2 reviewer ID: 123
Review: stars:4 reviewer ID: 345
Review: stars:3 reviewer ID: 568
Review: stars:5 reviewer ID: 450
```

### **Queries and templates**

In this exercise we combine database queries with Jinja templates. In the **routes.py** file, besides the `home()` route listing all the books in the database, we also added several routes for displaying web pages with filtered entries from the database. For example,

```python
@app.route('/books/<year>')
def books(year):
   books = Book.query.filter_by(year = year)
   return render_template('display_books.html', year = year, books = books)
```

is a dynamic route with the `year` variable that can be set to some valid year in the URL. Next, we filter out the books from the asked year and display them using the **display_books.html** file (you can find in the templates folder). The template expects the provided `year` and a list of `books` to display.

In the URL bar of the integrated Web browser on the right, type or copy-paste the following: **[http://localhost:8000/books/2020](http://localhost:8000/books/2020)**. You see all the books suggested in 2020, together with all the reviews for each book. Cool, ha?

If the list of the retrieved queries is empty, we handle that in the **display_books.html** template by outputting that there are no books suggested in that year. Alternatively, one can use `.first_or_404()` as we do in the following:

```python
reader = Reader.query.filter_by(id = user_id).first_or_404(description = "There is no user with this ID.")
```

If a reader with some `id` does not exist in the database, 404 error is returned with a custom made message.

Take some time to observe the complete code provided on the right paying attention to **routes.py**, and explore the templates in the `template` folder.

routes.py

```python
from app import app
from app import db, Reader, Book, Review, Annotation
from flask import render_template, request, url_for, redirect

@app.route('/home')
@app.route('/')
def home():
  books = Book.query.all()
  return render_template('home.html', books = books)

@app.route('/profile/<int:user_id>')
def profile(user_id):
   reader = Reader.query.filter_by(id = user_id).first_or_404(description = "There is no user with this ID.")
   return render_template('profile.html', reader = reader)

@app.route('/books/<year>')
def books(year):
   books = Book.query.filter_by(year = year)
   return render_template('display_books.html', year = year, books = books)

@app.route('/reviews/<int:review_id>')
def reviews(review_id):
   #your code here
   review = Review.query.filter_by(id = review_id).first_or_404(description = "There is no user with this ID.")
   return render_template('_review.html', review = review)
```

display_books.html

```html
<html>
    <head>
        <title>Book selected for the year {{year}}</title>
    </head>
    <body>
        {% for book in books %}
             <p>{{ book.title }}</p>
             {% for review in book.reviews %}
                   {% include '_review.html' %}
             {% endfor %}
        {% else %}
         <p> Sorry. There are no books suggested in the year {{year}}</p>
        {% endfor %}
    </body>
</html>
```