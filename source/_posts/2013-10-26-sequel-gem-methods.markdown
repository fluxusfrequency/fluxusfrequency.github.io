---
layout: post
title: "Sequel Gem Methods"
date: 2013-10-26 13:28
---

Information is from the [Jumpstart Lab Sequel Tutorial](http://tutorials.jumpstartlab.com/topics/sequel.html)



Here is the [official Sequel documentation](sequel.rubyforge.org/rdoc/classes/Sequel/Dataset.html).

<br />

##Setting Up A Database
1. Create a Gemfile requiring ‘sequel’ and ‘sqlite3’.
2. run: bundle exec rib; Bundler.require
3. Assign a new database like this:
database = Sequel.sqlite(‘dbname.sqlite3’) Sqlite creates the db automatically. Postgres will not do this.

<br />

##Database Methods:
\#run "SQL query" - runs an SQL query

\#fetch “SQL query” - runs an SQL query and returns the results (need to call to_a to see) as a dataset. Can be combined with a block.

These two methods alone will allow you to interact with the db via SQL.

<br />

##Dataset Methods:

\#enumerable - can run enumerable methods on a dataset returned by database#fetch.

\#inspect - shows the database/dataset obj.



###Table manipulation:

\#create_table :table_name do
     primary key :id
     String :line_1, :size => 255
     Integer :person_id
end
creates a table

\#schema(:table) - shows the existence and column definitions of a table. Returns a nested array.

\#add_column
\#rename_column
\#drop_column
\#set_column_default
\#set_column_type


###Selection of records:

chaining - chain methods:
e.g. dataset.where(:name => 'George').or([[:id, [2,3]]])

\#from(:column) - equivalent to SELECT * FROM column. Returns a dataset. Doesn’t run search until you call to_a on it.
After assigning it like this:
dataset = database.from(:people)
Do things like this:

\#select(:column or [:column1, :column2, etc…]) - selects column(s) from table

\#where/#filter(:column => filter) - returns SELECT * FROM table WHERE column = filter. Selecting multiple items requires an extra set of brackets.
eg.: dataset.where(:id => 1) or
dataset.where([[:id, [2,3]]])

\#grep(:column, ‘search') - finds all rows that match the grep string. “%" is a 0+ wildcard, “_" matches a single character.

\#limit(int) - limits the number of returned records to the integer passed as an argument.

\#order(:column) - returns SELECT * FROM :table ORDER BY :column. Can add descending sort as follows:
dataset.order(Sequel.desc(:name))

\#exclude - works like Enumerable#reject.



###Working With Rows

\#insert ({:column => “value", :column => “value") - inserts a new row with the given hash values. Not a delayed method; runs immediately. Verify results using select.

\#join(:other_table, :other_column, :foreign_key) - Performs an INNER JOIN with another table. The column name order goes: other table’s column first, local table’s column second.

\#update(:column => “value”) - updates one or more existing rows. Combine it with where to select a record first. Runs immediately.
e.g. addresses.where(:id => 1).update(:zipcode => "20500")

\#delete(:column => “value) - similar to update, but deletes. Returns number of records deleted.

\#count - returns the number of records in the dataset. Useful in chaining.



###Math Utilities:

\#avg
\#sum
\#max
\#min

