# 1436. Destination City

You are given the array `paths`, where <code>paths[i] = [cityA<sub>i</sub>, cityB<sub>i</sub>]</code> means there exists a direct path going from <code>cityA<sub>i</sub></code> to <code>cityB<sub>i</sub></code>. *Return the destination city, that is, the city without any path outgoing to another city*.

It is guaranteed that the graph of paths forms a line without any loop, therefore, there will be exactly one destination city.

Example:

> **Input:** `paths = [["London","New York"],["New York","Lima"],["Lima","Sao Paulo"]]`
> 
> **Output:** `"Sao Paulo"`
> 
> **Explanation:** Starting at "London" city you will reach "Sao Paulo" city which is the destination city. Your trip consist of: "London" -> "New York" -> "Lima" -> "Sao Paulo".


## Hash set

The question is in fact very simple: given a directed graph (edge list), find the nodes with zero out-deg. That is, nodes that never appear as the departure city in any journey.


## Sol.

```python
def destCity(paths: List[List[str]]) -> str:
    # All departure and destination city
    departure, destination = map(set, zip(*paths))

    # Destination city that never appears as departure city
    # As it's guaranteed to have one and only one answer, so just pop that one
    return (destination - departure).pop()
```
