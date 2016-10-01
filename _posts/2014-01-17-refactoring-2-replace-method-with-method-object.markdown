---
title: 'Refactoring 2: Replace Method with Method Object'
date: 2014-01-17 09:06:00 Z
categories:
- source
layout: post
---

As I mentioned last week, I'm doing a series of posts on refactoring. As gSchool wraps up, I'm working on my refactoring skills, so that I'll be a better developer when I get my first job. I'm trying some of the strategies outlined in the book [Refactoring Ruby](http://martinfowler.com/books/refactoringRubyEd.html).

Today I'll be trying out a strategy called Replace Method with Method Object. This is a good approach to take when you have a long method with many variables in it, and using [Extract Method](http://fluxusfrequency.github.io/blog/2014/01/10/refactoring-1-extract-method/) wouldn't be practical. Essentially, it consists of removing the method from its class and turning it into a class of its own. From there, you can more easily refactor using Extract Method and other simple strategies.

Here are the steps as outlined in the book:

```plain
1. Create a new class, name it after the method.

2. Give the new class an attribute for the object that hosted the original method (the source object) and an attribute for each temporary vairable and each parameter in the method.

3. Give the new class a constructor that takes the source object and each parameter.

4. Give the new class a method named "compute".

5. Copy the body of the original method into "compute". Use the source object instance variable for any invocations of methods on the original subject.

6. Test.

7. Replace the old method with one that creates the new method and call "compute".

```

## Mancala

I took a look around my Github repositories, and found this little wonder from a [Mancala](https://github.com/fluxusfrequency/mancala) app I wrote in Ruby Processing during the first month of gSchool.

```ruby

class MancalaKalahRules

  def distribute_beads_for_player(app, first_pit_id, player, count)
    next_pit_id = app.model.find_next_pit_by_id(first_pit_id)

    @i = 0
    while @i <= count do
      execute_store_logic(next_pit_id, player, count)
      break if @i == count
      app.model.add_bead_to_pit(next_pit_id)
      next_pit_id = app.model.find_next_pit_by_id(next_pit_id)
      if @i == count - 1
        landing_spot = next_pit_id - 1
        landing_spot = 12 if landing_spot == 0
        if landed_on_empty?(landing_spot) && app.model.find_all_pit_ids_on_players_side(player).include?(landing_spot)
          take_both_sides(landing_spot)
          break
        end
      end
      @i += 1
    end
  end

end

```

What a mess.

## Setting Up Tests

To begin the refactoring process, I had to make sure the tests were in place first. This was fairly difficult, as this method is highly coupled to other methods throughout the app. For testing purposes, I added a couple of stubbed methods to `KalahRules`, and created dummy `App` and `Model` classes with stubbed methods of their own. Also, `distribute_beads_for_player` did not return anything meaningful, so I changed it to do return the value of `@i`.

In the end, here's the code I used to make the test pass:


```ruby

require 'minitest/autorun'
require 'minitest/pride'
require './kalah_rules'

class KalahRulesTest < Minitest::Test

  def test_distribute_beads_for_player_returns_the_correct_count
    rules = MancalaKalahRules.new
    app = App.new
    id = 2
    player = nil
    count = 10

    expected = 9
    result = rules.distribute_beads_for_player(app, id, player, count)
    assert_equal expected, result
  end
end


class MancalaKalahRules

  def distribute_beads_for_player(app, first_pit_id, player, count)
    next_pit_id = app.model.find_next_pit_by_id(first_pit_id)

    @i = 0
    while @i <= count do
      execute_store_logic(next_pit_id, player, count)
      break if @i == count
      app.model.add_bead_to_pit(next_pit_id)
      next_pit_id = app.model.find_next_pit_by_id(next_pit_id)
      if @i == count - 1
        landing_spot = next_pit_id - 1
        landing_spot = 12 if landing_spot == 0
        if landed_on_empty?(landing_spot) && app.model.find_all_pit_ids_on_players_side(player).include?(landing_spot)
          take_both_sides(landing_spot)
          break
        end
      end
      @i += 1
    end
    @i
  end

  def execute_store_logic(pit_id, player, count)
  end

  def take_both_sides(spot)
  end

  def landed_on_empty?(spot)
    true
  end

end

class App
  def model
    Model.new
  end
end

class Model
  def find_next_pit_by_id(pit_id)
    3
  end

  def add_bead_to_pit(pit_id)
  end

  def find_all_pit_ids_on_players_side(player)
    [1, 2, 3, 4, 5, 6]
  end
end

```

All green! Time to start refactoring!

{% img http://s5.postimg.org/540zghpmv/All_green_before_refactor.png 'All Green Before Refactor' 'A set of passing Ruby tests' %}


## Applying the Strategy

Step 1: Create a new class named after the method:

``` ruby

class BeadDistributor
end


```

Step 2: Give the new class an attribute of the source object and its parameters.

``` ruby

class BeadDistributor

  def initialize(rules, app, first_pit_id, player, count)
  end

end

```

Step 3: Give the new class a constructor that takes the new attributes. I chose to add attr_readers as well:

``` ruby

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

end

```

Step 4: Give the new class a *compute* method:

``` ruby

class BeadDistributor

  # Code omitted for brevity

  def compute
  end

end

```

Step 5: Copy the original method into *compute*. I also copied in the stubbed methods for testing purposes.


``` ruby

class BeadDistributor

  # Code omitted for brevity

  def compute
    next_pit_id = app.model.find_next_pit_by_id(first_pit_id)

    @i = 0
    while @i <= count do
      execute_store_logic(next_pit_id, player, count)
      break if @i == count
      app.model.add_bead_to_pit(next_pit_id)
      next_pit_id = app.model.find_next_pit_by_id(next_pit_id)
      if @i == count - 1
        landing_spot = next_pit_id - 1
        landing_spot = 12 if landing_spot == 0
        if landed_on_empty?(landing_spot) && app.model.find_all_pit_ids_on_players_side(player).include?(landing_spot)
          take_both_sides(landing_spot)
          break
        end
      end
      @i += 1
    end
    @i
  end

  def execute_store_logic(pit_id, player, count)
  end

  def take_both_sides(spot)
  end

  def landed_on_empty?(spot)
    true
  end

end

```

Step 6: Test. I tested the newly created `BeadDistributor` class:

```ruby

class BeadDistributorTest < Minitest::Test

  def test_compute_returns_the_correct_count
    rules = MancalaKalahRules.new
    app = App.new
    id = 2
    player = nil
    count = 10
    distributor = BeadDistributor.new(rules, app, id, player, count)

    expected = 9
    result = distributor.compute
    assert_equal expected, result
  end

end

```

All green!

{% img http://s5.postimg.org/s81g92sxz/All_green_after_refactor.png 'All Green After Refactor' 'A set of passing Ruby tests' %}

Step 7: Replace the old method by instantiating the new object, passing the old object into it, and calling *compute*.

``` ruby

class MancalaKalahRules

  def distribute_beads_for_player(app, first_pit_id, player, count)
    BeadDistributor.new(self, app, first_pit_id, player, count).compute
  end

```

At this point, I ran the test2 again. The original one passed! Yay!

{% img http://s5.postimg.org/f55tpt2pz/Still_green.png 'Still Green' 'Another set of passing Ruby tests' %}



After completing the object extraction, I was free to refactor the new BeadDistributor using more common strategies. Here's what I ended up with:

``` ruby

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



class MancalaKalahRules

  def distribute_beads_for_player(app, first_pit_id, player, count)
    BeadDistributor.new(self, app, first_pit_id, player, count).compute
  end

  def execute_store_logic(pit_id, player, count)
  end

  def take_both_sides(spot)
  end

  def landed_on_empty?(spot)
    true
  end

end

class App
  def model
    Model.new
  end

  def add_bead_to_model(pit_id)
    model.add_bead_to_pit(pit_id)
  end

  def find_next_pit_in_model(pit_id)
    model.find_next_pit_by_id(pit_id)
  end

  def check_if_pit_belongs_to_player(player, landing_spot)
    model.find_all_pit_ids_on_players_side(player).include?(landing_spot)
  end
end

class Model
  def find_next_pit_by_id(pit_id)
    3
  end

  def add_bead_to_pit(pit_id)
  end

  def find_all_pit_ids_on_players_side(player)
    [1, 2, 3, 4, 5, 6]
  end
end


```

Running the tests one more time, I saw that everything was still green. Although I could have gone on to refactor some of the other methods in the original class that `distribute_beads_for_player` was coupled to, I decided to call it a day.

{% img http://s5.postimg.org/8fza9shdz/Final_green.png 'Final Green' 'Another set of passing Ruby tests' %}

## Conclusion

I enjoyed the Replace Method with Method Object strategy. It's a pretty cool way to think about the microcosm/macrocosm relationship between methods and objects. Although it's not probably useful for every refactoring case, I imagine it would come in really handy when cleaning up a hairy method like `distribute_beads_for_player`. I'm learning a lot from Refactoring Ruby, and I'm looking forward to seeing what other strategies it offers.
