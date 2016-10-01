---
title: 'Refactoring 4: Remove Middle Man'
date: 2014-01-27 19:34:00 Z
categories:
- source
layout: post
---

Today, in my latest journey on the [Refactor](http://martinfowler.com/books/refactoringRubyEd.html) Tractor, I'll be taking a look at a strategy called Move Method.

Move Method is used to help remove duplication. Sometimes, in the interest of encapsulation, it's a good idea to make an object as unaware of another object as possible. If the first needs to access the second, though, it may have to "reach through" a third object. If this reaching is minimal, it can sometimes be a worthwhile trade off. But when it becomes an ongoing pattern, you can end up with a lot of duplication. And a lot of what my teacher Franklin Webber calls "bad touching". The Refactoring book points out that "it's hard to figure out what the right amount of hiding is",  but it's easy to change your mind with refactoring. Using a combination of this strategy and Hide Delegate, you can always change it again later.

Today I'll be practicing Remove Middle Man on another example from the [Mancala](https://github.com/fluxusfrequency/mancala) app I was refactoring in my recent post: [Refactoring 2 - Replace Method With Method Object](fluxusfrequency.github.io/blog/2014/01/17/refactoring-2-replace-method-with-method-object/).

Here are the steps to Remove Middle Man:

```plain
1. Create an accessor for the delegate.

2. For each client use of a delegate method, remove the method from the server and replace the call in the client to call the method on the delegate.

3. Test after each method.

```

That's fairly cryptic. I think it means: just pass a reference to the distant object into the first object, and access it directly (instead of through object two).

## Mancala Game View

There are many poorly-built parts to Mancala. That's probably because I didn't test it at all as I was writing it. At the time, I thought I was taking a shortcut. In reality, I was making a big mess that was difficult to understand and deal with. I didn't realize it at the time, but ["TDD's primary benefit is to improve the design of our code"](http://blog.testdouble.com/posts/2014-01-25-the-failures-of-intro-to-tdd.html).

One of my poorly designed classes is called the MancalaGameView. Its job is to draw the beads in their respective pits, and print the winner to the screen, using [Ruby Processing](https://github.com/jashkenas/ruby-processing). Here's the problem with my design: I pass in the 'app' object, then call through it to get information about the 'rules' and 'model' objects. Here's what it looks like:

```ruby

class MancalaGameView

  attr_reader :app

  def initialize(app)
    @app = app
  end

  def draw_all_beads
    app.model.all_pits.each do |pit|
      draw_beads_in_pit(pit)
    end
    app.model.all_stores.each do |store|
      draw_beads_in_store(store)
    end
    print_winner if app.ruler.game_over?
  end

  def draw_beads_in_pit(pit)
    app.fill 225
    n = pit.count
    return if n == 0
    if n >= 1 && n <= 9
      draw_x_beads(pit, n)
    else
      draw_many_beads(pit)
    end
  end

  def draw_beads_in_store(store)
    fill_a_random_color(1)
    if store.id == 1
      draw_beads_in_player_one_store
    elsif store.id == 2
      draw_beads_in_player_two_store
    end
  end

  def print_winner
    app.text "Player one is the winner!", 640, 475 if app.ruler.winner == :player_1
    app.text "Player two is the winner!", 400, 250 if app.ruler.winner == :player_2
    app.text "Tie Game!", 640, 475 if app.ruler.winner == :tie
  end

end

```

The calls to `app.fill` and `app.text` are built into Ruby Processing. Outside of that, I'm not going to get into the details of what some of these methods, like `draw_x_beads` and `draw_beads_in_player_one_store`. Because the coupling in this app is so high, it would require pulling in too many methods.

## Setting Up the Test

I'm going to stub out a few things, so that we can build a test without having to drag in the rest of the app and the Ruby Processing Gem.

```ruby

class MancalaGameView

  attr_reader :app

  def initialize(app)
    @app = app
  end

  def draw_all_beads
    app.model.all_pits.each do |pit|
      draw_beads_in_pit(pit)
    end
    app.model.all_stores.each do |store|
      draw_beads_in_store(store)
    end
    print_winner if app.ruler.game_over?
  end

  def draw_beads_in_pit(pit)
    true
  end

  def draw_beads_in_store(store)
    true
  end

  def print_winner
    return "Player one is the winner!" if app.ruler.winner == :player_1
    return "Player two is the winner!" if app.ruler.winner == :player_2
    return "Tie Game!" if app.ruler.winner == :tie
  end

end

class App
  attr_reader :model, :ruler
  def initialize(model, ruler)
    @model = model
    @ruler = ruler
  end
end

class Model
  def all_pits
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
  end

  def all_stores
    [0, 1]
  end

end

class Ruler
  def winner
    winners = [:player_1, :player_2, :tie]
    @winner ||= winners[rand(3)]
  end

  def game_over?
    true
  end
end

```

And now, to build a test.

```ruby

class GameViewTest < Minitest::Test

  def setup
    @model = Model.new
    @ruler = Ruler.new
    @app = App.new(@model, @ruler)
    @view = MancalaGameView.new(@app)
  end

  def test_draw_all_beads_succeeds
    expected = ["Player one is the winner!", "Player two is the winner!", "Tie Game!"]
    result = @view.draw_all_beads
    assert expected.include?(result)
  end

end

```

{% img http://s15.postimg.org/uh17pf8ij/Green_Pattern.jpg 'All Green' 'The color green' %}

This test is green. I don't like to start from green. It's supposed to be: "red, green, refactor"! I could comment out some code or change `game_over?` to false, but that wouldn't prove anything about the code. Since this is an instance of Test After, and the code is already understood, I'm feel comfortable going forward from green here.

## Applying the Strategy

Now, to begin removing the middle man object, `app`. There are two classes I'm accessing through `app`: `Model` and `Ruler`. I'll extract them one at a time, beginning with `Model`. The first step is to "create an accessor for the delegate". I could just do this:

```ruby

class MancalaGameView
  def model
    app.model
  end
end

```

But I would rather pass the model into the MancalaGameView directly:

```ruby

class MancalaGameView

  attr_reader :app, :model

  def initialize(app, model)
    @app = app
    @model = model
  end

end

```

Step two is to change "client" calls so that they use the new accessor instead of calling through the "server" (which is `app` in this case). Step three is to test after each method. So here we go:

```ruby

# OLD VERSION
# def draw_all_beads
#   app.model.all_pits.each do |pit|
#     draw_beads_in_pit(pit)
#   end
#   app.model.all_stores.each do |store|
#     draw_beads_in_store(store)
#   end
#   print_winner if app.ruler.game_over?
# end

def draw_all_beads
  model.all_pits.each do |pit|
    draw_beads_in_pit(pit)
  end
  model.all_stores.each do |store|
    draw_beads_in_store(store)
  end
  print_winner if app.ruler.game_over?
end

```

{% img http://s30.postimg.org/va1jflych/red_pattern.jpg 'Red Tests' 'The color red' %}

The test fails with: `wrong number of arguments (1 for 2)`. The problem lies in the test setup. I now need to pass in the model when I create the `MancalaGameView`.

```ruby

class GameViewTest < Minitest::Test

  def setup
    # Code omitted
    @view = MancalaGameView.new(@app, @app.model)
  end

end

```

{% img http://s15.postimg.org/uh17pf8ij/Green_Pattern.jpg 'All Green' 'The color green' %}

Ahh, that's better. Now to repeat the process with the `ruler`. Step one: create the accessor.

```ruby

class MancalaGameView

  attr_reader :app, :model, :ruler

  def initialize(app, model, ruler)
    @app = app
    @model = model
    @ruler = ruler
  end

end

```

Steps two and three: replace the method calls to use the accessor instead of reaching through `app`. Test after each step.

```ruby

def draw_all_beads
  # Code omitted
  print_winner if ruler.game_over?
end


```

{% img http://s30.postimg.org/va1jflych/red_pattern.jpg 'Red Tests' 'The color red' %}

It fails, because I'm not passing `ruler` into the test.

```ruby

class GameViewTest < Minitest::Test

  def setup
    # Code omitted
    @view = MancalaGameView.new(@app, @app.model, @app.ruler)
  end

end

```

{% img http://s15.postimg.org/uh17pf8ij/Green_Pattern.jpg 'All Green' 'The color green' %}

That gets me back to green. Now to refactor the `print_winner` method:

```ruby

# OLD VERSION
# def print_winner
#   return "Player one is the winner!" if app.ruler.winner == :player_1
#   return "Player two is the winner!" if app.ruler.winner == :player_2
#   return "Tie Game!" if app.ruler.winner == :tie
# end

def print_winner
  return "Player one is the winner!" if ruler.winner == :player_1
  return "Player two is the winner!" if ruler.winner == :player_2
  return "Tie Game!" if ruler.winner == :tie
end

```

{% img http://s15.postimg.org/uh17pf8ij/Green_Pattern.jpg 'All Green' 'The color green' %}

Success! That was pretty easy.

## Further Refactoring

Now that I've cleaned up the MancalaGameView somewhat, I'm going to refactor a few other things. Along the way, I noticed that `print_winner` is pretty ugly, so I changed it to use a hash lookup instead.

```ruby

# OLD VERSION
# def print_winner
#   return "Player one is the winner!" if ruler.winner == :player_1
#   return "Player two is the winner!" if ruler.winner == :player_2
#   return "Tie Game!" if ruler.winner == :tie
# end

def print_winner
  win_messages[ruler.winner]
end

private

def win_messages
  { :player_1 => "Player one is the winner!",
    :player_2 => "Player one is the winner!",
    :tie      => "Tie Game!"}
end

```

I also decided to use extract method on the loops in `draw_all_beads`:

```ruby

# OLD VERSION
# def draw_all_beads
#   model.all_pits.each do |pit|
#     draw_beads_in_pit(pit)
#   end
#   model.all_stores.each do |store|
#     draw_beads_in_store(store)
#   end
#   print_winner if ruler.game_over?
# end

def draw_all_beads
  draw_pits
  draw_stores
  print_winner if ruler.game_over?
end

private

def draw_pits
  model.all_pits.each do |pit|
    draw_beads_in_pit(pit)
  end
end

def draw_stores
  model.all_stores.each do |store|
    draw_beads_in_store(store)
  end
end

```

That's much better. Here's the final result:

```ruby

class MancalaGameView

  attr_reader :app, :model, :ruler

  def initialize(app, model, ruler)
    @app = app
    @model = model
    @ruler = ruler
  end

  def draw_all_beads
    draw_pits
    draw_stores
    print_winner if ruler.game_over?
  end

  def draw_beads_in_pit(pit)
    true
  end

  def draw_beads_in_store(store)
    true
  end

  def print_winner
    win_messages[ruler.winner]
  end

  private

  def draw_pits
    model.all_pits.each do |pit|
      draw_beads_in_pit(pit)
    end
  end

  def draw_stores
    model.all_stores.each do |store|
      draw_beads_in_store(store)
    end
  end

  def win_messages
    { :player_1 => "Player one is the winner!",
      :player_2 => "Player one is the winner!",
      :tie      => "Tie Game!"}
  end

end

```

## Conclusion

Remove Middle Man was really easy to use. Maybe it's because I've heard Franklin harp on the evils of "bad touching" so much, or maybe it's just that I'm violating the Law of Demeter, but I start to feel pretty nervous when I see two dots in my method calls. This was a quick and easy refactor that I'm sure I'll use regularly. Since it sets you up for [Dependency Injection](http://codeulate.com/2013/01/a-criticism-of-dhhs-post-on-dependency-injection/), it also opens up the class's possibilities for [Duck Typing](http://en.wikipedia.org/wiki/Duck_typing) and easy mocking.

As I continue to code and refactor, Remove Middle Man will be a vital tool in my tool kit.
