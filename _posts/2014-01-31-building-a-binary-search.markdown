---
title: Building a Binary Search
date: 2014-01-31 10:34:00 Z
categories:
- source
layout: post
---

A couple of days ago I had an interview with a big startup in Boulder. I was really excited about the possibility of working there, because they solve some *really* hard problems on a daily basis, and because I've heard great things about their office culture.

It was just supposed to be a quick interview, but it ended up taking a few hours. First, I met up with the CTO. It all started off innocent enough. We went to a restaurant on Pearl Street for lunch. He asked me about my background and skills, and I asked him about the company. We shared personal stories about the flood last summer. After hearing about the way he treats his workers (like adults), and some of the technical details of what they were working on, I was even more excited about the company. We headed went to a coffee shop for a drink, then went back to the office.

When we got there, he introduced me to the head of the team that the best fit for me if I worked there. The team lead, one of his coworkers and I headed back to the coffee shop. We chatted about Rails, [Exercism](http://www.exercism.io), TDD, and Ember.js, and life at their company. Pretty soon, the CTO came in again. Seeing that we were still talking, he asked the team lead to take me back and "dig in technically" a little bit.

We went back to the office again, and found a conference room. He asked me to solve [FizzBuzz](http://en.wikipedia.org/wiki/Fizz_buzz) in JavaScript. I fired up Jasmine-Node and built it with tests the whole way. That went fairly well, I thought. Then he asked me a couple of CSS questions. I answered them, and he said that I answered them better than many of the people he interviews.

Finally, we came to some algorithm questions. Although I've learned a lot at gSchool in the last five months, algorithms are something that I haven't spent as much time on, yet. I've only had enough programming context to begin understanding them for the last two of those months. I've played around with them as much as I could, solving Linked List, the Luhn Algorithm, Nth Prime Factor, Sum of Squares, and Binary Search Tree. I've also had a go at writing the MD5 algorithm. But compared to someone with a four year CS degree, I haven't had much time to get comfortable with them.

My interviewer drew an array on the board, and asked me how I would find a given element's position within 2-3 queries. I thought about it, made a guess that it had to do with bit operators, and finally conceded that I had no idea. My heart sunk. Things were going so well. When I asked for the solution, he told me that it was to use a Binary Search. I got excited, because I *had* done a Binary Search Tree. But no, this was just plain old Binary Search.

Bitten that I didn't know how to do Binary Search, I set out to build it on my own. This rest of this post is a walkthrough of how I solved it.

## Research

First, I visited Wikipedia and read up on [Binary Search](http://en.wikipedia.org/wiki/Binary_search_algorithm). It was pretty easy to follow. So easy, in fact, I decided to lift some of the text verbatim and translate it into a set of tests to drive my implementation. Here's what I found:

```
Searching a sorted collection is a common task. A dictionary is a sorted list of
 word definitions. Given a word, one can find its definition. A telephone book
 is a sorted list of people's names, addresses, and telephone numbers. Knowing
 someone's name allows one to quickly find their telephone number and address.

If the list to be searched contains more than a few items (a dozen, say) a
binary search will require far fewer comparisons than a linear search, but it
imposes the requirement that the list be sorted.



In computer science, a binary search or half-interval search algorithm finds
the position of a specified input value (the search "key") within an array
sorted by key value.

In each step, the algorithm compares the search key value with the key value of
the middle element of the array.

If the keys match, then a matching element has been found and its index, or
position, is returned.

Otherwise, if the search key is less than the middle element's key, then the
algorithm repeats its action on the sub-array to the left of the middle element
or, if the search key is greater, on the sub-array to the right.

If the remaining array to be searched is empty, then the key cannot be found in
the array and a special "not found" indication is returned.

A binary search halves the number of items to check with each iteration, so
locating an item (or determining its absence) takes logarithmic time. A binary
search is a dichotomic divide and conquer search algorithm.
```

I decided to take this step-by-step description and turn each step into a test.

## Setting Up the Binary Search Test & Class

The first step was to create an object that knew about "an array sorted by key value", or as I like to call it, a list. I wrote a test:

```ruby
class BinarySearchTest < MiniTest::Test

  def test_it_has_list_data
    binary = BinarySearch.new([1, 3, 4, 6, 8, 9, 11])
    assert_equal [1, 3, 4, 6, 8, 9, 11], binary.list
  end

end
```

And a class to make it pass:

```ruby
class BinarySearch
  attr_reader :list

  def initialize(data)
    @list = data
  end

end
```

That was easy! Next, it needed to be able to check whether the list was actually sorted, so I wrote a test to make sure it would throw an error if it wasn't:

```ruby
def test_it_raises_error_for_unsorted_list
  assert_raises ArgumentError do
    BinarySearch.new([2, 1, 4, 3, 6])
  end
end
```

To make it pass, I implemented the following check: compare the input with itself in sorted form. Not that elegant, but it worked:

```ruby
class BinarySearch
  attr_reader :list

  def initialize(data)
    raise ArgumentError unless data.sort == data
    @list = data
  end

end
```

According to the Wikipedia article, the algorithm was going to have to find "the key value of the middle element of the array". So I wrote a test to make sure that it knew where the middle of its list was:

```ruby
def test_it_finds_position_of_middle_item
  binary = BinarySearch.new([1, 3, 4, 6, 8, 9, 11])
  assert_equal 3, binary.middle
end
```

This was fairly trivial to solve:

```ruby
class BinarySearch
  attr_reader :list

  # Code omitted

  def middle
    list.length/2
  end

end
```

With the class all set up, I turned my attention to the actual search algorithm itself.

## Building the Search Algorithm

The goal was to search for the index of a specific element, so I wrote a test for a method that did that:

```ruby
def test_it_finds_position_of_search_data
  binary = BinarySearch.new([1, 3, 4, 6, 8, 9, 11])
  assert_equal 5, binary.search_for(9)
end
```

Next came the hard part. I took the text from Wikipedia and broke it down into pseudo-code:

- Get the search value
- Find the middle element in the list
- Check if they are equal
- If they are equal, return the position of the middle element in the list
- If they are not equal, branch:

If the search key is less than the middle index:

- Cut the list we're searching in half
- Instantiate a new BinarySearch
- Pass the first half of the list into it
- Call the search method on the new object, with the same search key
- Repeat until the keys are equal
- If they're never equal, raise "Not Found"

If the search key is more than the middle index:

- Cut the list we're searching in half
- Instantiate a new BinarySearch
- Pass the second half of the list into it
- Call the search method on the new object, with the same search key
- Repeat until the keys are equal
- If they're never equal, raise "Not Found"

With this plan in place, I started to build the method. First, I created the method and passed in the search key:

```ruby
def search_for(key)

end
```

Then I checked it against the middle element in the list, returning the index if they were equal:

```ruby
def search_for(key)
  return middle if list[middle] == key
end
```

The next thing on my to do list was to branch for the other two conditions: whether the key was less than or greater than the middle value:

```ruby
def search_for(key)
  return middle if list[middle] == key

  if list[middle] > key
    # ???
  else
    # ???
  end

end
```

Then I filled in the branches some more of my pseudo-code:

```ruby
def search_for(key)
  return middle if list[middle] == key

  if list[middle] > key
    # Cut the list we're searching in half
    # Instantiate a new BinarySearch
    # Pass the first half of the list into it
    # Call the search on the new object, with the same search key
  else
    # Cut the list we're searching in half
    # Instantiate a new BinarySearch
    # Pass the second half of the list into it
    # Call the search on the new object, with the same search key
  end

end
```

I filled them in with real code, one step at a time:

```ruby
def search_for(key)
  return middle if list[middle] == key

  if list[middle] > key
    # Cut the list we're searching in half
    sublist = list[0..middle]

    # Instantiate a new BinarySearch
    # Pass the first half of the list into it
    search = BinarySearch.new(sublist)

    # Call the search on the new object, with the same search key
    return search.search_for(key)
  else
    # Cut the list we're searching in half
    sublist = list[middle..-1]

    # Instantiate a new BinarySearch
    # Pass the first half of the list into it
    search = BinarySearch.new(sublist)

    # Call the search on the new object, with the same search key
    return search.search_for(key)
  end

end
```

Seeing some duplication here, I went ahead and refactored:

```ruby
def search_for(key)
  return middle if list[middle] == key

  if list[middle] > key
    sublist = list[0..middle]
  else
    sublist = list[middle..-1]
  end

  return BinarySearch.new(sublist).search_for(key)

end
```

Time to run the tests.

```bash
Expected: 5
  Actual: 2
```

Fail.

Why? I did some `pry` action and discovered that I was getting back the index of the *new* list's middle element, not the *original* list's middle element. To fix this, I added the current list's middle to the recursive call. That meant undoing my refactor. I guess I refactored too soon.

```ruby
def search_for(key)
  return middle if list[middle] == key

  if list[middle] > key
    sublist = list[0..middle]
    return BinarySearch.new(sublist).search_for(key)
  else
    sublist = list[middle..-1]
    return BinarySearch.new(sublist).search_for(key) + middle
  end

end
```

The tests were now passing. Good. Time to try with another example. This time looking for keys on both sides of the middle.

```ruby
def test_it_finds_position_in_a_larger_list
  binary = BinarySearch.new([1, 3, 5, 8, 13, 21, 34, 55, 89, 144])
  assert_equal 1, binary.search_for(9)
end

```

Whoops! `stack level too deep`! I forgot to change value of the search on line 3. I was getting an infinite loop when I searched for a bad value. I skipped this test and wrote one for the bad search condition, instead:

```ruby
def test_it_raises_error_for_data_not_in_list
  assert_raises RuntimeError do
    BinarySearch.new([1, 3, 6]).search_for(2)
  end
end

```

To make this pass, I had to validate that the search wasn't stuck looking through the same sublist over and over:

```ruby
def search_for(key)
  return middle if list[middle] == key

  if list[middle] > key
    sublist = list[0..middle]
    raise "Not Found" if sublist == list
    return BinarySearch.new(sublist).search_for(key)
  else
    sublist = list[middle..-1]
    raise "Not Found" if sublist == list
    return BinarySearch.new(sublist).search_for(key) + middle
  end

end
```

It passed! I went back and rewrote the other test:

```ruby
def test_it_finds_position_in_a_larger_list
  binary = BinarySearch.new([1, 3, 5, 8, 13, 21, 34, 55, 89, 144])
  assert_equal 1, binary.search_for(3)
  assert_equal 7, binary.search_for(55)
end
```

Passed again! How about one more for good measure? I also wanted to check if the middle was still coming out right when I passed in a list with an even number of elements. All the others had been of an odd length.

```ruby
def test_it_finds_correct_position_in_a_list_with_an_even_number_of_elements
  binary = BinarySearch.new([1, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377])
  assert_equal 5, binary.search_for(21)
  assert_equal 6, binary.search_for(34)
end
```

Ok! Green!

## Conclusion

Even if I didn't know my stuff in the conference room at that moment, I'm glad that my interviewer asked me about Binary Search. It was fun to learn, and it turned out to be a little easier than I expected. I always expect algorithms to be really hard to grasp, but maybe now that I've done a few algorithms that use recursion, it's not such a jump for me to understand it.

At any rate, it's pretty cool to be able to find an element in a list with so few queries. Lately, I've really been enjoying pure math programming problems like this one. I'm going to try to find a few more like this to do in the near future.

## Footnote

Since I had already done all of the work, I went ahead and submitted this as a new exercise for [Exercism.io](http://www.exercism.io). So, if you use Exercism, and you enjoy doing algorithms, I'd suggest giving Binary Search a go yourself. If you get stuck, you know where to find the answer!
