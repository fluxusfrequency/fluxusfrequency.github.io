---
title: Refactoring 3 - Introduce Named Parameter / Introduce Class Annotation
date: 2014-01-18 13:22:00 Z
categories:
- source
layout: post
---

I've been trying to get better at refactoring lately. To do that, I'm digging into my old code from the first months of gSchool and applying strategies from [Refactoring Ruby](http://martinfowler.com/books/refactoringRubyEd.html) by Jay Fields, Shane Harvie, Martin Folwer, and Kent Beck.

Just a couple of days ago I posted about trying out [Replace Method with Method Object](http://fluxusfrequency.github.io/blog/2014/01/17/refactoring-2-replace-method-with-method-object/). Almost immediately afterwards, I read about two more strategies that seemed like they would fit well with the same refactoring example, so I thought I'd give them a try: Introduce Named Parameter and Introduce Class Annotation.

## Introduce Named Parameter

According to the book, naming your parameters is a good thing to do if you want to improve the readability of a method. Using it prevents the work of having to go searching through dependency objects to figure out what's going on. It works easily with the new `BeadDistributor` class I created in my last article. Here's that code again:

```ruby

class BeadDistributor
  attr_reader :rules,
              :app,
              :first_pit_id,
              :player,
              :count

  def initialize(rules, app, first_pit_id, player, count)
    @rules = rules
    @app = app
    @first_pit_id = first_pit_id
    @player = player
    @count = count
  end

  def compute
    fix_landing_on_spot_12
    take_both_if_own_empty
    distribute_beads
  end

  def distribute_beads
    (count - 1).times do |i|
      rules.execute_store_logic(next_pit_id, player, count)
      app.add_bead_to_model(next_pit_id)
    end
  end

  def next_pit_id
    app.find_next_pit_in_model(first_pit_id)
  end

  def take_both_if_own_empty
    rules.take_both_sides(landing_spot) if landed_on_players_own_empty_pit?
  end

  def landed_on_players_own_empty_pit?
    rules.landed_on_empty?(landing_spot) && app.check_if_pit_belongs_to_player(player, landing_spot)
  end

  def landing_spot
    @landing_spot ||= next_pit_id - 1
  end

  def fix_landing_on_spot_12
    @landing_spot = 12 if landing_spot == 0
  end

end


```

The most important parts here will be the `attr_reader`s and the `initialize` method.

Here are the steps for Introduce Named Parameter:

```plain
1. Choose the parameters that you want to name. If you are not naming all of the parameters, move the parameters that you want to name to the end of the parameter list.

2. Test

3. Replace the parameters in the calling code with name/value pairs.

4. Replace the parameters with a Hash object in the receiving method. Modify the receiving method to use the new Hash.

5. Test.

```

As always, I started with the tests. They were still passing after my last refactoring adventure:

{% img http://s5.postimg.org/iav0trapz/All_green_start.png 'All Green to Start' 'A set of passing Ruby tests' %}

For the purposes of this example, I wanted to make all of the parameters required, so I skipped steps 1 and 2.

Here are the relevant parts of the code again:

```ruby

class BeadDistributor
  attr_reader :rules,
              :app,
              :first_pit_id,
              :player,
              :count

  def initialize(rules, app, first_pit_id, player, count)
    @rules = rules
    @app = app
    @first_pit_id = first_pit_id
    @player = player
    @count = count
  end

  # Code omitted for brevity.

end

class MancalaKalahRules

  def distribute_beads_for_player(app, first_pit_id, player, count)
    BeadDistributor.new(self, app, first_pit_id, player, count).compute
  end

end

```

Step 3: Replace the params in the calling method with key/value pairs.

```ruby

class MancalaKalahRules

  def distribute_beads_for_player(app, first_pit_id, player, count)
    BeadDistributor.new({
      :rules => self,
      :app => app,
      :first_pit_id => first_pit_id,
      player => :player,
      :count => count}).compute
  end

end

```

Step 4: Modify the params taken into the receiving object to take a hash.

```ruby

class BeadDistributor
  attr_reader :rules,
              :app,
              :first_pit_id,
              :player,
              :count

  def initialize(params)
    @rules = params[:rules]
    @app = params[:app]
    @first_pit_id = params[:first_pit_id]
    @player = params[:player]
    @count = params[:count]
  end

  # Code omitted for brevity.

end

```

Step 5: Test.

{% img http://s5.postimg.org/ypoy6bsp3/Failing_after_first_refactor.png 'Failing After First Refactor' 'A set of passing Ruby tests' %}

The test is failing, because I haven't modified it to send a hash instead of the individual params. After fixing this, it passes again.

```ruby

class BeadDistributorTest < Minitest::Test

  def test_compute_returns_the_correct_count
    rules = MancalaKalahRules.new
    app = App.new
    id = 2
    player = nil
    count = 10
    distributor = BeadDistributor.new({
      :rules => rules,
      :app => app,
      :id => id,
      :player => player,
      :count => count})

    expected = 9
    result = distributor.compute
    assert_equal expected, result
  end

end

```

{% img http://s5.postimg.org/6e7zn0y6f/Passing_after_test_fix.png 'Passing After Test Fix' 'A set of passing Ruby tests' %}

At this point, everything's working. But why did I do it? If anything, it just added duplication to what was already there. Here's why: you can combine it with the Introduce Class Annotation strategy to DRY up object initialization.

## Introduce Class Annotation

Although I've heard from many developers more experienced than I am that metaprogramming is generally a bad idea, I also get the impression that many of them consider doing it anyway a guilty pleasure, particularly in non-production code.

However you feel about it, gird yourself, because we're heading into a little bit of `define_method` territory.

I'm going to use the same annotation the book gives as an example. It's a way of defining a helper along the lines of `attr_reader`, `attr_writer`, and `attr_successor`. In it, you wrap all of the object initialization code into a neat new method called `hash_initializer`.

Here are the steps:

```plain
1. Decide on the signature of your class annotation. Declare it in the appropriate class.

2. Convert the original method to a class method. Make the apppropriate changes so that the method works at the class scope.

3. Test.

4. Consider using Extract Module on the class method to make the annotation more prominent in your class definition.

```

Here we go.

Step 1: Declare the new class annotation.

```ruby

class BeadDistributor

  def hash_initializer(*attribute_names)
    define_method(:initialize) do
      data = args.first || {}
      attribute_names.each do |attribute_name|
        instance_variable_set "@#{attribute_name}", data[attribute_name]
      end
    end
  end

  attr_reader :rules,
              :app,
              :first_pit_id,
              :player,
              :count

  def initialize(params)
    @rules = params[:rules]
    @app = params[:app]
    @first_pit_id = params[:first_pit_id]
    @player = params[:player]
    @count = params[:count]
  end

  # Code omitted for brevity.


end

```

Step 2: Convert the method to a class method. I also called the `hash_initializer` at the top of the `BeadDistributor` class.

```ruby

class BeadDistributor
  def self.hash_initializer(*attribute_names)
    define_method(:initialize) do |*args|
      data = args.first || {}
      attribute_names.each do |attribute_name|
        instance_variable_set "@#{attribute_name}", data[attribute_name]
      end
    end
  end

  hash_initializer :rules,
                   :app,
                   :first_pit_id,
                   :player,
                   :count

  attr_reader :rules,
              :app,
              :first_pit_id,
              :player,
              :count

  # Code omitted for brevity.

end

```

Step 3: Test.

All green!

{% img http://s5.postimg.org/lqrqh1xc7/Passing_after_metaprogramming.png 'Passing After Metaprogramming' 'A set of passing Ruby tests' %}

Step 4: Extract the class annotation into a module.

```ruby

module CustomInitializers

  def hash_initializer(*attribute_names)
    define_method(:initialize) do |*args|
      data = args.first || {}
      attribute_names.each do |attribute_name|
        instance_variable_set "@#{attribute_name}", data[attribute_name]
      end
    end
  end

end

class BeadDistributor
  extend CustomInitializers

  hash_initializer :rules,
                   :app,
                   :first_pit_id,
                   :player,
                   :count

  attr_reader :rules,
              :app,
              :first_pit_id,
              :player,
              :count

  # Code omitted for brevity.

end

```

To me, the module you get at the end is like the gift that keeps on giving. Now I have a cool utility modult that I can use in other objects that take a hash of parameters. Right away, I can think of a program where I can use it in at least six classes.

Running the tests once more, everything is still cool.

{% img http://s5.postimg.org/i8fqknwg7/Everything_is_still_cool.png 'Everything is Still Cool' 'A set of passing Ruby tests' %}

## Conclusion

As I got toward the end of the fifth chapter, Refactoring got pretty deep into some `method_missing` and `define_method` metaprogramming madness. While I can't say I'm very excited to use these things in my day-to-day refactoring, it's nice to have them there as a reference in case I ever need them. If nothing else, tricks like the ones I tried out here help me think about the relationships between classes in new ways. Next week, I'll try some strategies from the next chapter: "Moving Features Between Objects".
