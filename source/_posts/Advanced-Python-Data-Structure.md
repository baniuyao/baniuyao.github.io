title: Advanced Python Data Structure
date: 2015-08-27 15:13:37
tags:
- python
---

# Colletions

`Collections` is a very useful data structure colletion in Python. In this post, I will introduct

## collections.Couter()
`Counter` is used to count something of iterable data structure such as list and tuple. Let's see example below:
``` python
from collections import Counter
name_list = ['frank', 'frank', 'tony', 'jack', 'judy', 'jack', 'jack']
name_counter = Counter(name_list) 
print counter # print couter of name_list
# Counter({'jack': 3, 'frank': 2, 'tony': 1, 'judy': 1})
print counter.most_common(1) # print top 1 count
# [('jack', 3)]
print counter.most_common(2) # print top 2 count
# [('jack', 3), ('frank', 2)]
```
Now I am going to update a Counter:
``` python
from collections import Counter
name_list_one = ['frank', 'frank', 'tony']
name_counter = Counter(name_list_one)
# name_counter: Counter({'frank': 2, 'tony': 1})
name_list_two = ['tony', 'jack']
name_counter.update(name_list_two)
# name_counter: Counter({'tony': 2, 'frank': 2, 'jack': 1})
name_counter.subtract(['frank']) # subtract is the reverse operation of update
# name_counter: Counter({'tony': 2, 'frank': 1, 'jack': 1})
name_counter.subtract(['jack', 'jack']) # value can be zero and negative counts
# name_counter: Counter({'tony': 2, 'frank': 1, 'jack': -1}) 
```

## collections.defaultdict()

`defaultdict` is almost the same as `dict` which can be assigned by this statement:
``` python
d = {'name': 'frank'}
```

`defaultdict` provides a function which will be invoked when we use a non-exists key to access the value in the `defaultdict`. When we want to get the value by a key which does not exist in the dict, `dict` will raise a `KeyError` Exception. We can use `d.get(KEY, DEFAULT_VALUE)` to define the value we will get if the key does not exist. It is okay but not an easy work-around. For example, I have a dict named `writer_book_dict` and its key is writers' name and value is books' name. When I get the value by a non-exist writer, it should return 'NO BOOKS'. Let's see how we can deal it with `dict` and `defaultdict`.

``` python
writer_book_dict = {'frank': 'book_1', 'tony': 'book_2'}
book_by_jack = writer_book_dict.get('jack', 'NO BOOKS')
book_by_jack = writer_book_dict.get('judy', 'NO BOOKS')

from collections import defaultdict
writer_book_defaultdict = default(lamda: 'NO BOOKS')
writer_book_defaultdict.get('jack') # return 'NO BOOKS'
```

We can use a complex function in initialize the defaultdict. Furthermore, we can also define the data structure of the value in the `defaultdict`. Let's see an example, we need to make a classification of a short paragraph by the first letter of each world. It is to say, 'I am a student and I like sports' -> `{'a': ['am', 'a', 'and'], 'i': ['it', 'is', 'i'], 'l': ['like'], 's': ['student', 'sports']}`. If we use `default` we must judge if the key exists and if it does not exist, we need to initialize with an empty list like below:
``` python
paragraph = 'I am a student and I like sports'
word_dict = {}
for word in paragraph:
    first_letter = word[0]
    if word not in word_dict:
        word_dict[first_letter] = [word]
    else:
        word_dict[first_letter].append(word)
```

Let's see how `defaultdict` works:
``` python
from collections import defaultdict
paragraph = 'I am a student and I like sports'
word_defaultdict = defaultdict(list)
for word in paragraph:
    word_defaultdict[word[0]].append(word)
```
If we want to `uniq` the element of `word_defaultdict`, we can simply initialize `word_defaultdict` by `defaultdict(set)`.

## bisect

`bisect` is used to insert element into an sorted list and keep the list sorted. `bisect` module have six methods:
1. bisect
2. bisect_left
3. bisect_right
4. insort
5. insort_left
6. insort_right

Methods start with `bisect` will return the index of the element you need to insert. Those start with `insort` will make change into list directly and return nothing. The difference between `left` and `right` will affect the result of operation only when the element you want to insert does exist in the list and `left` will return the index left of this existed-element and `right` return the right position. Please remark that `bisect` is the same as `bisect_right` and `insort` is the same as `insort_right`. Let's see some examples.
``` python
sorted_list = [1, 10, 100, 1000]
bisect(sorted_list, 20) # return 2
bisect(sorted_list, 10) # return 2
bisect_left(sorted_list, 10) # return 1
bisect_right(sorted_list, 10) # return 1
```