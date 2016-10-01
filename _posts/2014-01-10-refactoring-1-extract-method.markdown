---
title: 'Refactoring 1: Extract Method'
date: 2014-01-10 09:07:00 Z
categories:
- source
layout: post
---

I'm reading [Refactoring Ruby](http://martinfowler.com/books/refactoringRubyEd.html) by Jay Fields, Sean Harvie, Martin Fowler and Kent Beck. As I'm heading into the last couple of months at gSchool, I want to clean up some of the code I wrote, and practice my refactoring skills. I'll be doing a series of articles detailing the strategies I learn. This is the first.

Today I'll be talking about the "Extract Method" strategy. Here's how the book describes it:

```plain

1. Create a new method, and name it after the intention of the method (name it by what it does, not by how it does it).
If the code you want to extract is very simple, such as a single message or function call, you should extract it if the name of the new method reveals the intention of the code in a better way. If you can’t come up with a more meaningful name, don’t extract the code.

2. Copy the extracted code from the source method into the new target method.

3. Scan the extracted code for references to any variables that are local in scope to the source method. These are local variables and parameters to the method.

4. See whether any temporary variables are used only within this extracted code. If so, declare them in the target method as temporary variables.

5. Look to see whether any of these local-scope variables are modified by the extracted code. If one variable is modified, see whether you can treat the extracted code as a query and assign the result to the variable con- cerned. If this is awkward, or if there is more than one such variable, you can’t extract the method as it stands. You may need to use Split Tempo- rary Variable and try again. You can eliminate temporary variables with Replace Temp with Query (see the discussion in the examples).

6. Pass into the target method as parameters local-scope variables that are read from the extracted code.

7. Replace the extracted code in the source method with a call to the target method.
If you moved any temporary variables over to the target method, look to see whether they were declared outside the extracted code. If so, you can now remove the declaration.

8. Test.

```

Here's some code from a Sinatra app I wrote early on in gSchool, called [IdeaBox](http://ideabox-flux.herokuapp.com). For this project, we couldn't use ActiveRecord or DataMapper, but had to hand build an ORM.

 I selected these methods from my Finders module. They work together to search through a YAML file for records that match a search string. The `find_raw` method looked really long and complex, so I thought this would be a good place to do some refactoring.

```ruby

module Finders

  def search_for(query="")
    query = query.downcase
    find_raw({'title' => query, 'description' => query, 'tags' => query})
  end

  def find_raw(query_hash)
    begin
      found = []
      query_hash.keys.each do |key|
        query_string = query_hash[key]
        database.transaction do
          result = database['ideas'].select do |idea|
            idea[key].downcase.include?(query_string)
          end
          found << result
        end
      end
      found = found.flatten.compact.collect do |raw|
        Idea.new(raw)
      end
      select_current_portfolio_only(found)
    rescue
      []
    end
  end
end

```

My first step was to check the tests for the method.

```ruby

class SearchingTest < Minitest::Test

  def test_it_can_search_for_ideas_with_title_description_or_tags
    IdeaStore.create({
      'title' => "Recreation",
      'description' => "Bicycles In The Park",
      'tags' => 'bike',
      })
    IdeaStore.create({
      'title' => "Sundays",
      'description' => "In the park with George",
      'tags' => 'gerschwin',
      })
    assert_equal 2, IdeaStore.search_for("park").length
    assert_kind_of Idea, IdeaStore.search_for("Gerschwin").first
  end

end

```

To be thorough, I added a a couple of assertions, one to test the title field search, one for cases where there's nothing in the database, and one for a nil search.

```ruby

  def test_search_for_can_find_ideas_by_title_description_or_tags
    IdeaStore.create({
      'title' => "Recreation",
      'description' => "Bicycles In The Park",
      'tags' => 'bike',
      })
    IdeaStore.create({
      'title' => "Sundays",
      'description' => "In the park with George",
      'tags' => 'gerschwin',
      })
    assert_equal 1, IdeaStore.search_for("Recreation").length
    assert_equal 2, IdeaStore.search_for("park").length
    assert_kind_of Idea, IdeaStore.search_for("Gerschwin").first
  end

  def test_search_for_doesnt_blow_up_on_nil
    IdeaStore.create({})
    assert_equal [], IdeaStore.search_for("anything")
    assert_equal [], IdeaStore.search_for(nil)
  end

```

This last test broke on `undefined method `downcase' for nil`, so I updated the search_for method to protect against nil, and also return empty searched don't yield any results.

```ruby

  def search_for(query)
    return [] if query.to_s == ""

    query = query.downcase
    find_raw({'title' => query, 'description' => query, 'tags' => query})
  end


```

With tests in place to make sure that I wouldn't break any of the existing functionality, I began to make changes. My first step was to rename the find_raw method to find_yaml_data, reflecting its real intention. I then had to change the method call in `search_for`.

Beginning the extraction, I created a new method called find_in_database_for_mutliple_queries (step 1) and pulled the code from find_yaml_data into it (step 2).

```ruby

  def search_for(query)
    return [] if query.to_s == ""

    query = query.downcase
    find_yaml_data({'title' => query, 'description' => query, 'tags' => query})
  end

  def find_yaml_data(query_hash)
    begin

      found = found.flatten.compact.collect do |raw|
        Idea.new(raw)
      end
      select_current_portfolio_only(found)
    rescue
      []
    end
  end

  def find_multiple_queries_in_database
    found = []
    query_hash.keys.each do |key|
      query_string = query_hash[key]
      database.transaction do
        result = database['ideas'].select do |idea|
          idea[key].downcase.include?(query_string)
        end
        found << result
      end
    end
    found
  end

```

In step 3, I looked at the original method and found that the result of the code now living in `find_multiple_queries_in_database` was being used on the next line:

```ruby
  found = found.flatten.compact.collect do |raw|
    Idea.new(raw)
  end
```

However, I didn't see any variables used only in the new code that needed to be moved over (step 4), because I'd already moved `found = []`. I could therefore safely skip step 5, which helps deal with any such variables.


Turning my attention to the new method, I identified the `query_hash` variable as a problem, because the new method didn't know about it. Step 6 has you turn local variables like these into parameters to the new method. Here, I decided to rename it from `query_hash` to `queries`, because the fact that it's a hash is an implementation detail.

```ruby

  def find_multiple_queries_in_database(queries)
    found = []
    queries.keys.each do |key|
      query_string = query_hash[key]
      database.transaction do
        result = database['ideas'].select do |idea|
          idea[key].downcase.include?(query_string)
        end
        found << result
      end
    end
    found
  end

```

Finally, in step 7, I made a call to the new method from the body of the old, and assigned it to the local variable needed to pass it to the next line.

```ruby

  def find_yaml_data(query_hash)
    begin
      found = find_multiple_queries_in_database(query_hash)
      found = found.flatten.compact.collect do |raw|
        Idea.new(raw)
      end
      select_current_portfolio_only(found)
    rescue
      []
    end
  end

```

At this point, I went ahead and ran the tests (step 8). They were failing, but I couldn't tell why because of the begin/end block. I decided to remove it. I wrote it when I was still pretty new to programming, and was afraid of nil.

```ruby

  def find_yaml_data(query_hash)
    found = find_multiple_queries_in_database
    found = found.flatten.compact.collect do |raw|
      Idea.new(raw)
    end
    select_current_portfolio_only(found)
  end

```

Running the tests again, I could now see the error: `undefined local variable or method `query_hash'`. I saw that I had forgotten to rename the variable in the new method from `query_hash` to `queries`, so I fixed it.

```ruby

  def find_multiple_queries_in_database(queries)
    found = []
    queries.keys.each do |key|
      query_string = queries[key]
      database.transaction do
        result = database['ideas'].select do |idea|
          idea[key].downcase.include?(query_string)
        end
        found << result
      end
    end
    found
  end

```

All of the tests now passed. Looking at my code, I now had:

```ruby

  def search_for(query="")
    return [] if query.to_s == ""

    query = query.downcase
    find_yaml_data({'title' => query, 'description' => query, 'tags' => query})
  end

  def find_yaml_data(query_hash)
    found = find_multiple_queries_in_database(query_hash)
    found = found.flatten.compact.collect do |raw|
      Idea.new(raw)
    end
    select_current_portfolio_only(found)
  end

  def find_multiple_queries_in_database(queries)
    found = []
    queries.keys.each do |key|
      query_string = queries[key]
      database.transaction do
        result = database['ideas'].select do |idea|
          idea[key].downcase.include?(query_string)
        end
        found << result
      end
    end
    found
  end

```

I still wasn't happy with the complexity, so I decided to extract some more methods. I went after this bit in `find_yaml_data`:

```ruby

  found = found.flatten.compact.collect do |raw|
    Idea.new(raw)
  end

```

I extracted it, following the same process:

```ruby

  # Step 1: Create a new method
  def build_ideas_from_raw_data
  end


  # Step 2: Copy over the code
  def build_ideas_from_raw_data
    found.flatten.compact.collect do |raw|
      Idea.new(raw)
    end
  end

  # Step 3: Look for local variables in the source method
  # Here, there's one called 'found'
  found = found.flatten.compact.collect do |raw|
    Idea.new(raw)
  end

  # Step 4: Look for temporary variables in the extracted code.
  # There aren't any.

  # Step 5: If any temps are modified, replace them with queries.
  # Again, there aren't any.

  # Step 6: Pass locals into the new method as parameters.

  def build_ideas_from_raw_data(data)
    data.flatten.compact.collect do |raw|
      build_idea_from_data(raw)
    end
  end

  # Step 7: Replace extracted code with a call to
  # the new method in the source method.
  # I also renamed the temp from 'found' to 'ideas' for clarity.

  def find_yaml_data(query_hash)
    found = find_multiple_queries_in_database(query_hash)
    ideas = build_ideas_from_raw_data(found)
    select_current_portfolio_only(ideas)
  end

```

Step 8 is to test. All green! Taking another look at the whole thing, I now had this:

```ruby

   def search_for(query="")
     return [] if query.to_s == ""

     query = query.downcase
     find_yaml_data({'title' => query, 'description' => query, 'tags' => query})
   end

   def find_yaml_data(query_hash)
     found = find_multiple_queries_in_database(query_hash)
     ideas = build_ideas_from_raw_data(found)
     select_current_portfolio_only(ideas)
   end

   def build_ideas_from_raw_data(data)
    data.flatten.compact.collect do |raw|
      build_idea_from_data(raw)
    end
  end

   def find_multiple_queries_in_database(queries)
     found = []
     queries.keys.each do |key|
       query_string = queries[key]
       database.transaction do
         result = database['ideas'].select do |idea|
           idea[key].downcase.include?(query_string)
         end
         found << result
       end
     end
     found
   end

```

My next thought was to clean up `find_multiple_queries_in_database`. I started by replacing the #each call with a #map instead. This let me remove all references to the `found` local variable, and cleaned the whole thing up considerably.

```ruby

  def find_multiple_queries_in_database(queries)
    queries.keys.map do |key|
      query_string = queries[key]
      database.transaction do
        result = database['ideas'].select do |idea|
          idea[key].downcase.include?(query_string)
        end
      end
    end
  end

```

Reflecting further, I saw that the hash I was creating in the `search_for` method was unneccesary, as all of the values were the same. I moved the keys out into an array, and adjusted the `find_yaml_data` method to accept a string rather than a hash.

```ruby

  def search_for(query="")
    return [] if query.to_s == ""

    query = query.downcase
    find_yaml_data(query)
  end

  def fields_to_search
    ["title", "description", "tags"]
  end

  def find_yaml_data(query)
    found = find_multiple_queries_in_database(query)
    ideas = build_ideas_from_raw_data(found)
    select_current_portfolio_only(ideas)
  end

```

Running the tests, this wasn't working, as the `find_multiple_queries_in_database` method still expected a hash. I changed it as well:

```ruby

  def find_mutliple_queries_in_database(query)
    fields_to_search.collect do |field|
      database.transaction do
        find_query_in_database(query, field)
      end
    end
  end

```

I didn't like the name anymore, so I changed it to search_fields_for_query. This last refactoring was a sweeping change, and removing the hash cleaned up a lot of things. All of the tests were still green. The code now looked like this:

```ruby

  def search_for(query)
    return [] if query.to_s == ""

    query = query.downcase
    find_yaml_data(query)
  end

  def fields_to_search
    ["title", "description", "tags"]
  end

  def find_yaml_data(query)
    found = search_fields_for_query(query)
    ideas = build_ideas_from_raw_data(found)
    select_current_portfolio_only(ideas)
  end

  def build_ideas_from_raw_data(data)
    data.flatten.compact.collect do |raw|
      Idea.new(raw)
    end
  end

  def search_fields_for_query(query)
    fields_to_search.collect do |field|
      database.transaction do
        database['ideas'].select do |idea|
          idea[field].downcase.include?(query.downcase)
        end
      end
    end
  end

```

I could see that `search_fields_for_query` still had another candidate for extraction, so I went ahead with it, following the same steps:

```ruby

  def search_fields_for_query(query)
    fields_to_search.collect do |field|
      database.transaction do
        find_query_in_database(query, field)
      end
    end
  end

  def find_query_in_database(query, field)
    database['ideas'].select do |idea|
      idea[field].downcase.include?(query)
    end
  end

```

I then realized that I could push the #downcase call from `search_for` fown into the database query. This cleaned up the former nicely:

```ruby

  def search_for(query)
    return [] if query.to_s == ""
    find_yaml_data(query)
  end


  def find_query_in_database(query, field)
    database['ideas'].select do |idea|
      idea[field].downcase.include?(query.downcase)
    end
  end

```

Since search_for was now just delegating to `find_yaml_data`, I deleted the latter and pulled its code into `search_for`.

```ruby

  def search_for(query)
    return [] if query.to_s == ""

    found = search_fields_for_query(query)
    ideas = build_ideas_from_raw_data(found)
    select_current_portfolio_only(ideas)
  end

```

As a finishing touch, I pulled the transformation of raw data into an object out of `build_ideas_from_raw_data` and into a new method. This increased the lines of code, but I thought it was clearer, and the new method could potentially be reused elsewhere.

```ruby

  # Before
  def build_ideas_from_raw_data(data)
   data.flatten.compact.collect do |raw|
     Idea.new(raw)
   end
  end

  # After
  def build_ideas_from_raw_data(data)
    data.flatten.collect do |raw|
      build_idea_from_data(raw)
    end
  end

  def build_idea_from_data(data)
    Idea.new(data)
  end

```

All tests still green. Here's the final result:


```ruby

  def search_for(query)
    return [] if query.to_s == ""

    found = search_fields_for_query(query)
    ideas = build_ideas_from_raw_data(found)
    select_current_portfolio_only(ideas)
  end

  def search_fields_for_query(query)
    fields_to_search.collect do |field|
      database.transaction do
        find_query_in_database(query, field)
      end
    end
  end

  def fields_to_search
    ["title", "description", "tags"]
  end

  def find_query_in_database(query, field)
    database['ideas'].select do |idea|
      idea[field].downcase.include?(query.downcase)
    end
  end

  def build_ideas_from_raw_data(data)
    data.flatten.collect do |raw|
      build_idea_from_data(raw)
    end
  end

  def build_idea_from_data(data)
    Idea.new(data)
  end

```

Ahh, much better. Thanks, Extract Method strategy!
