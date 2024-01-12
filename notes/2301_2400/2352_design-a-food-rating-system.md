# 2353. Design a Food Rating System

Design a food rating system that can do the following:

* **Modify** the rating of a food item listed in the system.
* Return the highest-rated food item for a type of cuisine in the system.

Implement the `FoodRatings` class:

* `FoodRatings(String[] foods, String[] cuisines, int[] ratings)` initializes the system. The food items are described by `foods`, `cuisines` and `ratings`, all of which have a length of `n`.
    - `foods[i]` is the name of the $i$-th food,
    - `cuisines[i]` is the type of cuisine of the $i$-th food, and
    - `ratings[i]` is the initial rating of the $i$-th food.
* `void changeRating(String food, int newRating)` changes the rating of the food item with the name `food`.
* *String highestRated(String cuisine)* returns the name of the food item that has the highest rating for the given type of `cuisine`. If there is a tie, return the item with the **lexicographically smaller** name.

Note that a string `x` is lexicographically smaller than string `y` if `x` comes before `y` in dictionary order, that is, either `x` is a prefix of `y`, or if `i` is the first position such that `x[i] != y[i]`, then `x[i]` comes before `y[i]` in alphabetic order.

**Example:**

> **Input**
>
> `["FoodRatings", "highestRated", "highestRated", "changeRating", "highestRated", "changeRating", "highestRated"]`
> `[[["kimchi", "miso", "sushi", "moussaka", "ramen", "bulgogi"], ["korean", "japanese", "japanese", "greek", "japanese", "korean"], [9, 12, 8, 15, 14, 7]], ["korean"], ["japanese"], ["sushi", 16], ["japanese"], ["ramen", 16], ["japanese"]]`
> 
> **Output**
>
> `[null, "kimchi", "ramen", null, "sushi", null, "ramen"]`
> 
> **Explanation**
>
> `FoodRatings foodRatings = new FoodRatings(["kimchi", "miso", "sushi", "moussaka", "ramen", "bulgogi"], ["korean", "japanese", "japanese", "greek", "japanese", "korean"], [9, 12, 8, 15, 14, 7]);`
> 
> `foodRatings.highestRated("korean");     // return "kimchi"`
> 
> "kimchi" is the highest `rated korean food with a rating of 9.
> 
> `foodRatings.highestRated("japanese");   // return "ramen"`
> 
> `foodRatings.changeRating("sushi", 16);  // "sushi" now has a rating of 16.`
> 
> `foodRatings.highestRated("japanese");   // return "sushi"japanese food with a rating of 16.`
> 
> `foodRatings.changeRating("ramen", 16);  // "ramen" now has a rating of 16.`
> 
> `foodRatings.highestRated("japanese");   // return "ramen"`
> 
> Both "sushi" and "ramen" have a rating of 16. However, "ramen" is lexicographically smaller than "sushi".


## Brute force

Create a dict. of cuisines. For each cuisines, save the foods and their ratings in a list of `[{food name: rating}]`. Then sort the list by the rating (descending) and food name (ascending), and finally return the first food name.

## Optimise the sorting by priority queue

Sorting a list can be slow, so we can somehow maintain the order of the rating list when we construct it, using stuff like [`SortedList`](https://grantjenks.com/docs/sortedcontainers/) or [priority queue](https://docs.python.org/3/library/heapq.html#priority-queue-implementation-notes). We use the built-in implementation `heapq` for priority queue here.

We need a dict. to store the basic info. of foods, like:

```python
foods = {
    'kimchi': {
        'cuisine': 'korean',
        'rating': 1
    },
    'sushi': {
        'cuisine': 'japanese',
        'rating': 2
    }
}
```
Then for each cuisine, we create a priority queue, which looks like

```python
ratings = {
    'korean': [(1, 'kimchi'), (2, 'bulgogi')],
    'japanese': [(3, 'ramen'), (4, 'sushi'), (5, 'ramen')]
}
```

**NB:**

* priority queues are by default sorted ascendantly, i.e. higher ratings will be ranked later, so we need to take the opposite of the rating to have a descending order
* we can have duplicated entries for the same food in the priority queue, bc. when we change its rating, we push it to the queue, but never/cannot delete the previous one
* need to handle non-existing cuisine, e.g. the top food for `'italian'` in the above example will be `None`


## Sol.

```python
from collections import defaultdict
import heapq


class FoodRatings:
    def __init__(self, foods: List[str], cuisines: List[str], ratings: List[int]):
        self.foods = defaultdict(dict)    # Basic food info.
        self.ratings = defaultdict(list)  # Heap for each cuisine
        for i in range(len(foods)):
            heapq.heappush(self.ratings[cuisines[i]], (-ratings[i], foods[i]))
            self.foods[foods[i]]['cuisine'] = cuisines[i]
            self.foods[foods[i]]['rating'] = ratings[i]

    def changeRating(self, food: str, newRating: int) -> None:
        self.foods[food]['rating'] = newRating
        heapq.heappush(self.ratings[self.foods[food]['cuisine']], (-newRating, food))

    def highestRated(self, cuisine: str) -> str:
        while self.ratings[cuisine][0]:
            # Get the top food in the queue for the given cuisine
            top_in_queue = self.ratings[cuisine][0]

            # The top food may be obsolete, if we have increased or decreased its rating. So check
            # if the rating of top food in queue is equal to its current rating store in foods'
            # basic info.
            if top_in_queue[0] != -self.foods[top_in_queue[1]]['rating']:
                # If not, delete this entry in the heap
                heapq.heappop(self.ratings[cuisine])
            else:
                return top_in_queue[1]

        # If we get nothing from above, then we don't have any food for the given cuisine, so
        # return `None`
        return None
```
