---
layout: post
title: "Palindromes, Sets and XOR"
date: 2014-05-01 15:10:48 -0400
comments: true
categories: ruby
---

My Flatiron School classmate Arielle Sullivan posted [her solution](https://medium.com/p/ab6f4ef849d) to an interesting programming puzzle: 
>Determine whether the letters in a given string can be arranged to form a palindrome.

Her solutions is below:
```ruby
def possible_palindrome?(string)
  uniq_chars = []
  string.chars.each do |next_char|
    if uniq_chars.include?(next_char)
      uniq_chars.delete(next_char)
    else
      uniq_chars << next_char
    end
  end
  uniq_chars.length <=1 ? true : false
end
```
The gist of it is this: Iterate through the string to make sure at most one character is unpaired. So for instance with `lloo`, every character is paired up, so we can make the palindrome `lool`. The same with `llo` making the palindone `lol`.

If two or more characters are unpaired, this breaks down. So for instance `lllo` has an unpaired `l` and therefore can't form a palindrome. 

Looking at line 1, my first throught was that this was a good case for the ruby `inject` method (or `reduce`).

```ruby
def possible_palindrome?(string)
  string.chars.inject([]) do |uniq_chars,next_char|
    if uniq_chars.include?(next_char)
      uniq_chars.delete(next_char)
    else
      uniq_chars << next_char
    end
  end.length <=1
end
```

Looking good! But something was bothering me. We're essentially doing the following:
- If the array of uniques has a character, remove it.
- If it doesn't have a character, add it.

This is essentailly the `XOR` or [_exclusive or_](http://en.wikipedia.org/wiki/Exclusive_or) operator! In ruby the xor operator is the `^` character. In boolean expressions, it returns true if one of two expressions is true but not both.

```ruby
true ^ true   #=>false
true ^ false  #=>true
false ^ true  #=>true
false ^ false #=>false
```
How does this translate to sets and characters? Well, if we have two sets of characters, we can xor them together to get the characters in either but not in both. This is exactly what we want. Unforunately, you can't natively xor two arrays in ruby, but you can with the [Set](http://www.ruby-doc.org/stdlib-2.1.1/libdoc/set/rdoc/Set.html) class. Let's give it a try:
```ruby
require 'set'

def possible_palindrome?(string)
  string.chars.inject(Set.new []) do |uniq_chars,next_char|
    uniq_chars ^ [next_char]
  end.length <=1
end
```
Even better! If we wanted to do this without the Set class, we can xor two arrays ourselves:
```ruby
arr1 = [1,2,3]
arr2 = [3]

(arr1 + arr2) - (arr1 & arr2) #=> [1,2,3,3] - [3] => [1,2]
```
Let's break this down. `(arr1 + arr2)` returns an array that combines the two arrays. `(arr1 & arr2)` returns an array of items in both. Subtracting the latter from the former results in items that are in either one but not in both. 

So our final solution, sans sets would be:
```ruby
def possible_palindrome?(string)
  string.chars.inject([]) do |uniq_chars,next_char|
    (uniq_chars + [next_char]) - (uniq_chars & [next_char])
  end.length <=1
end
```
