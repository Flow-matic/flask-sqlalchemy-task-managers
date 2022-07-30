Now that we have the basic Flask application set up and running, it's time to start creating
our database schema by defining our models.
We'll start by creating a new file called models.py within our taskmanager package.
Since we will be defining the database, we obviously need to import db from the main taskmanager package.
For the purposes of these videos, we will be creating two separate tables, which will
be represented by class-based models using SQLAlchemy's ORM.
The first table will be for various categories, so let's call this class 'Category', which
will use the declarative base from SQLAlchemy's model.
The second table will be for each task created, so we'll call this class 'Task', also using the default db.Model.
We want each of these tables to have a unique ID, acting as our primary key, so let's create
a new column variable of 'id' for both of these tables.
These will be columns on our tables, and each will be Integers, with primary_key set to
True, which will auto-increment each new record added to the database.
If you recall from the last few lessons, we had to specifically import each column type at the top of the file.
However, with Flask-SQLAlchemy, the 'db' variable contains each of those already, and we can
simply use dot-notation to specify the data-type for our columns.
For each model, we also need to create a function called __repr__ that takes itself as the argument,
similar to the 'this' keyword in JavaScript.
This is a standard Python function meaning 'represent’, which means to represent the class objects as a string.
Another function that you might see out there, is the __str__ function that behaves quite
similar, and either should be suitable to use.
For now, let's just return self for those, but we'll come back to these momentarily.
Within our Category table, let's add a new column of 'category_name', which will be set to a standard db.Column().
This will be the type of 'string', with a maximum character count of 25, but you can
make these larger or smaller if needed.
We also want to make sure each new Category being added to the database is unique, so we'll set that to True.
Then, we also need to make sure it's not empty or blank, so by setting nullable=False this
enforces that it's a required field.
Now that we have a category_name, we can simply return self.category_name in our function.
For our Task table, we'll add a new column of 'task_name', which will also be a standard db.Column().
This will also be the type of 'string', with a maximum character count of 50, which should
also be unique for each record, and required, so nullable=False.
Then, we'll create a new column for 'task_description', and this time we'll use db.Text instead of
string, which allows longer strings to be used, similar to textareas versus inputs.
Next, we'll create a new column for 'is_urgent', but this will be a Boolean field, with a default
set to False, and nullable=False.
Then, the next column will be 'due_date', and this data-type will be db.Date with nullable=False,
but if you need to include time on your database, then db.DateTime or db.Time are suitable.
By the way, you can see a full list of column and data types from the SQLAlchemy docs, which
include Integer, Float, Text, String, Date, Boolean, etc.
The final column we need for our tasks is a foreign key pointing to the specific category, which will be our category_id.
This will use db.Integer of course, and for the data-type we need to use db.ForeignKey
so our database recognizes the relationship between our tables, which will also be nullable=False.
The value of this foreign key will point to the ID from our Category class, and therefore
we need to use lowercase 'category.id' in quotes.
In addition to this, we are going to apply something called ondelete="CASCADE" for this foreign key.
Since each of our tasks need a category selected, this is what's known as a one-to-many relationship.
One category can be applied to many different tasks, but one task cannot have many categories.
If we were to apply many categories to a single task, this would be known as a many-to-many relationship.
Let's assume that we have 10 tasks on our database, and 2 categories, with these two
categories being assigned to 5 tasks each.
Later, we decide to delete 1 of those categories, so any of the 5 tasks that have this specific
category linked as a foreign key, will throw an error, since this ID is no longer available.
This is where the ondelete="CASCADE" comes into play, and essentially means that once
a category is deleted, it will perform a cascading effect and also delete any task linked to it.
If these aren't deleted, and a task contains an invalid foreign key, then you will get errors.
In order to properly link our foreign key and cascade deletion, we need to add one more field to the Category table.
We'll call this variable 'tasks' plural, not to be confused with the main Task class, and
for this one, we need to use db.relationship instead of db.Column.
Since we aren't using db.Column, this will not be visible on the database itself like
the other columns, as it's just to reference the one-to-many relationship.
To link them, we need to specify which table is being targeted, which is "Task" in quotes.
Then, we need to use something called 'backref' which establishes a bidirectional relationship
in this one-to-many connection, meaning it sort of reverses and becomes many-to-one.
It needs to back-reference itself, but in quotes and lowercase, so backref="category".
Also, it needs to have the 'cascade' parameter set to 'all, delete' which means it will find
all related tasks and delete them.
The last parameter here is lazy=True, which means that when we query the database for
categories, it can simultaneously identify any task linked to the categories.
The final thing that we need to do with our class-based models, is to return some sort of string for the Task class.
We could simply return self.task_name, but instead, let's utilize some of Python's formatting to include different columns.
We'll use placeholders of {0}, {1}, and {2}, and then the Python .format() method to specify
that these equate to self.id, self.task_name, and self.is_urgent.
Go ahead and save the file, and let's return back to our routes.py file now.
At the top of the routes, we need to import these classes in order to generate our database next.
Save this file, but don't worry if you get any linting warnings about unused values.
We'll come back to these in the next lessons, but it's required to have our database schema
created within Postgres.
If you recall from our environment variables, we've specified that our database will be
called 'taskmanager', not to be confused with our 'taskmanager' Python package.
This would be similar to the 'chinook' database we created in the previous lessons, so we
will need to also create this 'taskmanager' database.
Let's navigate to the Terminal, and login to the Postgres CLI by typing 'psql' and hitting enter.
To create the database, we can simply type:
CREATE DATABASE taskmanager;
Once that's created, we'll switch over to that connection by typing:
\c taskmanager;
We don't need to do anything else within the Postgres CLI now that we have the database
created, so let's come out of the CLI by typing \q.
Next, we need to use Python to generate and migrate our models into this new database.
This will take the models that we've created for Category and Task, and build the database
schema using the details we've provided.
It's important to note, that if you were to modify your models later, then you'll need
to migrate these changes each time the file is updated with new context.
For example, if you added a new column on the Task table for 'task_owner', once the
file is saved, you'll need to make your migrations once again, so the database knows about these changes.
Let's go ahead and access the Python interpreter by typing "python3" and enter.
From here, we need to import our 'db' variable found within the taskmanager package, so type:
from taskmanager import db
Now, using db, we need to perform the .create_all() method:
That's it, pretty simple enough, our Postgres database should be populated with these two
tables and their respective columns and relationships.
Let's exit the Python interpreter by typing exit().
Congratulations, you now have your database models and schema migrated into the database.
You should go ahead and push your changes to GitHub now for version control.
However, if you'd like to confirm that these tables exist within your database, then follow
these instructions listed on the screen.
I'll see you in the next lesson, where we are going to start creating some of the front-end
templates with the Materialize CSS framework!
