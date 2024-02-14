# 3. INTRODUCTION TO FLASK-SQLALCHEMY

### **Why have databases in your web applications?**

Web applications are often built around a lot of data that change frequently. The data is usually organized in entities related in some way. For example, entities such as users are related to products by the act of purchasing, music albums are related to a specific artist by authoring, users are related to other users by befriending, etc.

*Relational databases* offer robust and efficient data management. A usual relational database consists of tables that represent entities and/or relationships amongst entities. The attributes of entities are constrained (for example, NAME attribute is a string, and a user’s PASSWORD should not be empty). The way a database is organized in entities, attributes and relationships, without data being present, is called the database schema.

**Database schema design: A simple book club scenario**

You want to create a personal book club application. Each month you pick a book your friends can review and rate. Your web app manages registered readers, the list of books you choose each month, and the ratings the readers write for those books. Moreover, you can show the annotations your friends made while reading the suggested books.

The schema for this problem is shown on the right. Inspect the schema by following the instructions below.

![Untitled](3%20INTRODUCTION%20TO%20FLASK-SQLALCHEMY%20922f2f699ad04b72ab1ce1f883e5d99e/Untitled.png)

### **Flask application with Flask-SQLAlchemy**

*Flask-SQLAlchemy* is an extension for Flask that supports the use of a Python SQL Toolkit called SQLAlchemy.

To start creating a minimal application, in addition to importing Flask, we also need to import `SQLAlchemy` class from the `flask_sqlalchemy` module:

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
```

The next step is to create our Flask app instance:

```python
app = Flask(__name__)
```

To enable communication with a database, the Flask-SQLAlchemy extension takes the location of the application’s database from the `SQLALCHEMY_DATABASE_URI` configuration variable we set in the following way:

```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///myDB.db'
```

Next, we set the `SQLALCHEMY_TRACK_MODIFICATIONS` configuration option to `False` to disable a feature of Flask-SQLAlchemy that signals the application every time a change is about to be made in the database.

```python
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
```

Finally, we create an SQLAlchemy object and bind it to our app:

```python
db = SQLAlchemy(app)
```

app.py

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

#database configuration
app = Flask(__name__) #application instance
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///myDB.db' #path to database and its name
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False #to supress warning
db = SQLAlchemy(app) #database instance

#some routing for displaying the home page
@app.route('/')
@app.route('/home')
def home():
    return "Congrats! You have just created your first Flask application supporting databases!"
```

![Untitled](3%20INTRODUCTION%20TO%20FLASK-SQLALCHEMY%20922f2f699ad04b72ab1ce1f883e5d99e/Untitled%201.png)

### **Declaring a simple model: Book**

The database object `db` created in our application contains all the functions and helpers from both SQLAlchemy and SQLAlchemy Object Relational Mapper (ORM). SQLAlchemy ORM associates user-defined Python classes with database tables, and instances of those classes (objects) with rows in their corresponding tables. The classes that mirror the database tables are referred to as *models*.

We would like to create a Flask-SQLAlchemy ORM representation of the following table schema:

![https://content.codecademy.com/programs/flask/databases/book.jpg](https://content.codecademy.com/programs/flask/databases/book.jpg)

The key symbol represents the primary key column that denotes a column or a property that uniquely identifies entries in the table. For example, student number, social security number, SKU (stock keeping unit), ISBN (International Standard Book Number), and similar, often serve as [primary keys](https://www.codecademy.com/resources/docs/sql/primary-keys).

`Model` represents a declarative base in SQLAlchemy which can be used to declare models. For `Book` to be a database model for the database instance `db`, it has to inherit from `db.Model` in the following way:

```python
class Book(db.Model):
```

As you can see in the code editor, the `Book` model has 5 attributes of `Column` class. The types of the column are the first argument to `Column`. We use the following column types:

- `String(N)`, where N is the maximum number of characters
- `Integer`, representing a whole number

`Column` can take some other parameters:

- `unique`: when `True`, the values in the column must be unique
- `index`: when`True`, the column is searchable by its values
- `primary_key`: when `True`, the column serves as the primary key

app.py

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///myDB.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False #to supress warning
db = SQLAlchemy(app)

#declaring the Book model
class Book(db.Model):
    id = db.Column(db.Integer, primary_key = True) #primary key column, automatically generated IDs
    title = db.Column(db.String(80), index = True, unique = True) # book title
    author_name = db.Column(db.String(50), index = True, unique = False) #author name added
    author_surname = db.Column(db.String(80), index = True, unique = False) #author surname
    month = db.Column(db.String(20), index = True, unique = False) #the month of book suggestion
    year = db.Column(db.Integer, index = True, unique = False) #tthe year of boook suggestion
    
    #Get a nice printout for Book objects
    def __repr__(self):
        return "{} in: {},{}".format(self.title, self.month,self.year)

@app.route('/')
@app.route('/home')
def home():
    return "Congrats! You have just made your first Flask-SQLAlchemy model declaration!"
```

![Untitled](3%20INTRODUCTION%20TO%20FLASK-SQLALCHEMY%20922f2f699ad04b72ab1ce1f883e5d99e/Untitled%202.png)

### **Declaring a simple model: Reader**

Adding another model or table schema to your application is simple. You only need to create another class that inherits from `Model`.

The model you will create next, `Reader`, is simple and similar to `Book`. Let us try it together. You can do this!

To make it easier for you, here’s the schema representation of `Reader`

:

![https://content.codecademy.com/programs/flask/databases/reader.jpg](https://content.codecademy.com/programs/flask/databases/reader.jpg)

We have already provided the `Reader` class declaration and the representation method.

app.py

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///myDB.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False #to supress warning
db = SQLAlchemy(app)

#declaring the Book model
class Book(db.Model):
    id = db.Column(db.Integer, primary_key = True) #primary key column
    title = db.Column(db.String(80), index = True, unique = True) # book title
    author_name = db.Column(db.String(50), index = True, unique = False) #author name
    author_surname = db.Column(db.String(80), index = True, unique = False) #author surname
    month = db.Column(db.String(20), index = True, unique = False) #the month of the book suggestion
    year = db.Column(db.Integer, index = True, unique = False) #tthe year of the book suggestion
    
    #Get a nice printout for Book objects
    def __repr__(self):
        return "{} in: {},{}".format(self.title, self.month,self.year)

#Add your columns for the Reader model here below.
class Reader(db.Model):
  id = db.Column(db.Integer, primary_key = True)
  name = db.Column(db.String(50), index = True, unique = False)
  surname = db.Column(db.String(80), unique = False, index = True)
  email = db.Column(db.String(120), unique = True, index = True)
  
  #get a nice printout for Reader objects
  def __repr__(self):
      return "Reader: {}".format(self.email)

#some routing for displaying the home page
@app.route('/')
@app.route('/home')
def home():
    return "Congrats! You have just made a Flask-SQLAlchemy model declaration all by yourself!"
```

### **Part I: declaring relationships (one-to-many)**

Often times in real-world applications we will have entities that are somehow related. Students take courses, customers buy products, and users comment on posts. In SQLAlchemy we can declare a *relationship* with a field initialized with the `.relationship()` method. In one-to-many relationships, the `relationship` field is used on the ‘one’ side of the relationship. In our use case we have the following one-to-many relationships:

1. One book ———< many reviews for that book
2. One reader ——–< many reviews from that reader

Hence, we add `relationship` fields to the `Book` and `Reader` models. In this exercise, we will show you how to add a relationship to the `Book` model, and you will do the same for the `Reader` model.

We declare a one-to-many relationship between `Book` and `Review` by creating the following field in the `Book` model:

```python
reviews = db.relationship('Review', backref='book', lazy='dynamic')
```

where

- the first argument denotes which model is to be on the ‘many’ side of the relationship: `Review`.
- `backref = 'book'` establishes a `book` attribute in the related class (in our case, class `Review`) which will serve to refer back to the related `Book` object. *`lazy = dynamic` makes related objects load as SQLAlchemy’s query objects.

By adding `relationship` to `Book` we only handled one side in our one-to-many relationship. Specifically, we only covered the direction denoted by the red arrow in the schema below:

![https://content.codecademy.com/programs/flask/databases/book-review.jpg](https://content.codecademy.com/programs/flask/databases/book-review.jpg)

In the next exercise, we will add the `Review` model and its relationship with the `Book` model (the blue arrow).

app.py

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///myDB.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False #to supress warning
db = SQLAlchemy(app)

#declaring the Book model
class Book(db.Model):
    id = db.Column(db.Integer, primary_key = True) #primary key column
    title = db.Column(db.String(80), index = True, unique = True) # book title
    author_name = db.Column(db.String(50), index = True, unique = False)
    author_surname = db.Column(db.String(80), index = True, unique = False) #author surname
    month = db.Column(db.String(20), index = True, unique = False) #the month of the book suggestion
    year = db.Column(db.Integer, index = True, unique = False) #the year of the book suggestion
    reviews = db.relationship('Review', backref = 'book', lazy = 'dynamic') #relationship of Books and Reviews
    
    #Get a nice printout for Book objects
    def __repr__(self):
        return "{} in: {},{}".format(self.title, self.month,self.year)

#Add your columns for the Reader model here below.
class Reader(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    name = db.Column(db.String(50), index = True, unique = False)
    surname = db.Column(db.String(80), unique = False, index = True)
    email = db.Column(db.String(120), unique = True, index = True)
    reviews = db.relationship('Review', backref = 'reviewer', lazy = 'dynamic')
  
    #get a nice printout for Reader objects
    def __repr__(self):
        return "Reader: {}".format(self.email)

#some routing for displaying the home page
@app.route('/')
@app.route('/home')
def home():
    return "Congrats! You are making your first one-to-many relationship in Flask-SQLAlchemy!"
```

### **Part II: declaring relationships (Foreign keys)**

In the previous lesson, we began adding a one-to-many relationship to the `Book` and `Reader` models by using `.relationship()`. But that does not completely specify our one-to-many relationship. We additionally have to specify what the *foreign keys* are for the model on the ‘many’ side of the relationship. To remind you, a foreign key is a field (or collection of fields) in one table that refers to the primary key in another table.

In this exercise we want to create the following database schema:

![https://content.codecademy.com/programs/flask/databases/reader-book-review.jpg](https://content.codecademy.com/programs/flask/databases/reader-book-review.jpg)

To complete the schema, we need to add the `Review` model, and specify the foreign keys (blue arrows) representing the following relationship:

- One review ——– one book for which the review was written
- One review ——– one reader who wrote that review

The red arrows were covered in the previous exercise with the `db.relationship()` columns.

Similar to the previous models we declared, the `Review` model has its own columns such as `text`, `stars` (denoting ratings), and its own primary key field `id`. `Review` additionally needs to specify which other models it is related to by specifying their primary key in its foreign key column:

```python
book_id = db.Column(db.Integer, db.ForeignKey('book.id'))
```

The `book_id` field is a foreign key that refers to the primary key `id` of the `Book` table. Similar to the primary key, a foreign key is just another column in our model with unique entries.

app.py

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///myDB.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False #to supress warning
db = SQLAlchemy(app)

#declaring the Book model
class Book(db.Model):
    id = db.Column(db.Integer, primary_key = True) #primary key column
    title = db.Column(db.String(80), index = True, unique = True) # book title
    author_name = db.Column(db.String(50), index = True, unique = False)
    author_surname = db.Column(db.String(80), index = True, unique = False) #author surname
    month = db.Column(db.String(20), index = True, unique = False) #the month of the book suggestion
    year = db.Column(db.Integer, index = True, unique = False) #the year of the book suggestion
    reviews = db.relationship('Review', backref = 'book', lazy = 'dynamic') #relationship of Books and Reviews
    
    #Get a nice printout for Book objects
    def __repr__(self):
        return "{} in: {},{}".format(self.title, self.month, self.year)

#Add your columns for the Reader model here below.
class Reader(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    name = db.Column(db.String(50), index = True, unique = False)
    surname = db.Column(db.String(80), unique = False, index = True)
    email = db.Column(db.String(120), unique = True, index = True)
    reviews = db.relationship('Review', backref='reviewer', lazy = 'dynamic')
  
    #get a nice printout for Reader objects
    def __repr__(self):
        return "Reader: {}".format(self.email)

#declaring the Review model
class Review(db.Model):
    id = db.Column(db.Integer, primary_key = True) #primary key column, automatically generated IDs
    stars = db.Column(db.Integer, unique = False) #a review's rating
    text = db.Column(db.String(200), unique = False) #a review's text
    #here below is the foreign key column linking to the primary key (id) of the Book model (book). 
    #Note the lower case here: 'book.id' instead of 'Book.id'
    book_id = db.Column(db.Integer, db.ForeignKey('book.id')) #foreign key column
    #your code here
    reviewer_id = db.Column(db.Integer, db.ForeignKey('reader.id'))
    #get a nice printout for Review objects
    def __repr__(self):
        return "Review: {} stars: {}".format(self.text, self.stars)

#some routing for displaying the home page
@app.route('/')
@app.route('/home')
def home():
    return "Congrats! You have just made a foreign key column in your Flask-SQLAlchemy model!"
```

### **Putting it all together: initializing the database**

When you ran your application in the previous exercises you might have realized that there is no database file created in the application folder. The reason is simple: we need to explicitly initialize the database according to the models declared.

We can initialize our database in two ways:

**1:** Using the interactive Python shell.

- In the command-line terminal, navigate to the application folder and enter Python’s interactive mode:

```bash
$ python3
```

- Import the database instance `db` from **app.py**:

```bash
>>> from app import db
```

(this assumes the application file is called **app.py**) *Create all database tables according to the declared models:

```bash
>>> db.create_all()
```

**2:** From within the application file.

- After all the models have been specified the database is initialized by adding `db.create_all()` to the main program. The command is written after all the defined models.

The result of `db.create_all()` is that the database schema is created representing our declared models. After running the command, you should see your database file in the path and with the name you set in the `SQLALCHEMY_DATABASE_URI` configuration field.

app.py

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///myDB.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False #to supress warning
db = SQLAlchemy(app)

#declaring the Book model
class Book(db.Model):
    id = db.Column(db.Integer, primary_key = True) #primary key column
    title = db.Column(db.String(80), index = True, unique = True) 
    author_name = db.Column(db.String(50), index = True, unique = False)
    author_surname = db.Column(db.String(80), index = True, unique = False) 
    month = db.Column(db.String(20), index = True, unique = False) 
    year = db.Column(db.Integer, index = True, unique = False) 
    reviews = db.relationship('Review', backref = 'book', lazy = 'dynamic') 
    
    #Get a nice printout for Book objects
    def __repr__(self):
        return "{} in: {},{}".format(self.title, self.month, self.year)

#Declaring the Reader model
class Reader(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    name = db.Column(db.String(50), index = True, unique = False)
    surname = db.Column(db.String(80), unique = False, index = True)
    email = db.Column(db.String(120), unique = True, index = True)
    reviews = db.relationship('Review', backref='reviewer', lazy = 'dynamic')
  
    #get a nice printout for Reader objects
    def __repr__(self):
        return "Reader: {}".format(self.email)

#declaring the Review model
class Review(db.Model):
    id = db.Column(db.Integer, primary_key = True) #primary key column, 
    stars = db.Column(db.Integer, unique = False) #a review's rating
    text = db.Column(db.String(200), unique = False) #a review's text
    book_id = db.Column(db.Integer, db.ForeignKey('book.id')) #foreign key 
    reviewer_id = db.Column(db.Integer, db.ForeignKey('reader.id'))

    #get a nice printout for Review objects
    def __repr__(self):
        return "Review: {} stars: {}".format(self.text, self.stars)

#some routing for displaying the home page
@app.route('/')
@app.route('/home')
def home():
    return "Congrats! You have just initialized your database!"
```

### **Creating database entries: entities**

Now that we initialized our database schema, the next step is to start creating entries that will later populate the database. The beauty of SQLAlchemy Object Relational Mapper (ORM) is that our database entries are simply created as instances of Python classes representing the declared models.

We will create our objects in a separate file called **create_objects.py**. To create objects representing model entries, we first need to import the models from the **app.py** file:

```python
from app import Reader, Book, Review
```

We can create an object of class `Book` in the following way:

```python
 b1 = Book(id = 123, title = 'Demian', author_name = 'Hermann', author_surname = 'Hesse', month = "February", year = 2020)
```

An example object of class `Reader` could be:

```python
r1 = Reader(id = 342, name = 'Ann', surname = 'Adams', email = 'ann.adams@example.com')
```

Thanks to the ORM, creating database entries is the same as creating Python objects.

We interact with database entries in the way we interact with Python objects. In case we want to access a specific attribute or column, we do it in the same way we would access attributes of Python objects: by using `.` (dot) notation:

```python
print("My first reader:", r1.name) # prints My first reader: Ann
```

create_objects.py

```python
#This is a separate Python script in which we practice creating database objects
#You can also perform these operations in command-line terminal
from app import Reader, Book, Review

b1 = Book(id = 123, title = 'Demian', author_name = "Hermann", author_surname = 'Hesse', month = "February", year = 2020)
r1 = Reader(id = 342, name = 'Ann', surname = 'Adams', email = 'ann.adams@example.com')

print("My first reader:", r1.name)

#your code here below
#Checkpoint 1: 
b2 = Book(id = 533, title = "The Stranger", author_name = "Albert", author_surname = "Camus", month = "April", year = 2019)
#Checkpoint 2: 
r2 = Reader(id = 765, email = "sam.adams@example.com", surname = "Adams", name = "Sam")
#Checkpoint 3: 
print(b2.author_surname)
#Checkpoint 4:
print(len(r2.email)   )
```

![Untitled](3%20INTRODUCTION%20TO%20FLASK-SQLALCHEMY%20922f2f699ad04b72ab1ce1f883e5d99e/Untitled%203.png)

### **Creating database entries: relationships**

Creating objects for tables that have foreign keys is not much different from the usual creation of Python objects.

Consider that we have the following objects already created:

```python
b1 = Book(id = 123, title = 'Demian', author_name = 'Hermann', author_surname = 'Hesse')
b2 = Book(id = 533, title = 'The stranger', author_name = 'Albert', author_surname = 'Camus')
r1 = Reader(id = 342, name = 'Ann', surname = 'Adams', email = 'ann.adams@example.com')
r2 = Reader(id = 312, name = 'Sam', surname = 'Adams', email = 'sam.adams@example.com')
```

To create an entry in the `Review` table, in addition to specifying a review text and a rating, we also need to specify which reader wrote the review, and for which book. In other words, we need to specify values for the review’s foreign keys `reviewer_id` and `book_id` that represent [primary keys](https://www.codecademy.com/resources/docs/sql/primary-keys) in `Reader` and `Book`, respectively.

```python
rev1 = Review(id = 435, text = 'This book is amazing...', stars = 5, reviewer_id = r1.id, book_id = b1.id)
```

In the example above we see that the review is written by `Reader` instance `r1` for `Book` instance `b1`. We again used Python’s dot notation to access the `id` attribute of `r1` and `b1` objects.

Note: in the future, when creating database entries you don’t need to specify the primary key value explicitly, if you don’t have a preference for the values. When adding entries to a database, a primary key value will be automatically generated, unless specified. In the next lesson, we will see how to add entries to our database.

create_objects.py

```python
#This is a separate Python script in which we practice creating database objects
#You can also perform these operations in command-line terminal
from app import Reader, Book, Review

b1 = Book(id = 123, title = 'Demian', author_name = 'Hermann', author_surname = 'Hesse')
b2 = Book(id = 533, title = 'The stranger', author_name = 'Albert', author_surname = 'Camus')
r1 = Reader(id = 342, name = 'Ann', surname = 'Adams', email = 'ann.adams@example.com')
r2 = Reader(id = 312, name = 'Sam', surname = 'Adams', email = 'sam.adams@example.com')

rev1 = Review(id = 435, text = 'This book is amazing...', stars = 5, reviewer_id = r1.id, book_id = b1.id)
print(rev1) #prints the rev1 object
print(rev1.text) #prints the text of the review rev1
print(rev1.book_id) #prints the id of the book for which the review was written

#Checkpoint 1: 
rev2 = Review(id = 450, text = "This book is difficult!", stars = 2, reviewer_id = r2.id, book_id = b2.id)

#Checkpoint 2:
print(len(rev2.text.split()))
```

![Untitled](3%20INTRODUCTION%20TO%20FLASK-SQLALCHEMY%20922f2f699ad04b72ab1ce1f883e5d99e/Untitled%204.png)