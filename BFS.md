# Breadth-first search
Breadth-first search is a technique that comes up in a multitude of BIO Round 1 Question 3's. It's a way of searching a graph.

A graph is a collection of *nodes* or *vertices* (points) connected by *edges*.

Consider the below graph:

![A central blue circle, which is connected to four red circles. Each red circle is connected to two green circles, so there are 8 green circles in total on the outside](http://u.cubeupload.com/jokebookservice1/bfs.png)

The grey lines are the edges, and the circles are the nodes.

Suppose we're trying to find a *specific* circle. All of the circles are different -- we can imagine they have some kind of number inscribed into them (which isn't pictured in the above diagram).

In an actual Question 3 problem, we might need to generate the circles on the fly, and so we may be forced to start at a particular circle, such as the blue circle in the middle. We then have two approaches. We could try to go as 'deep' as possible after generating a new circle. So for example, we might generate the red circle in the top-left corner, and then we would immediately try to generate a green circle connected to that red circle. We would then go back and generate the other green circle; and then we could generate another red circle (where we would repeat the process). This is an algorithm called depth-first search.

The other approach would be to generate all of the red circles in one go, and then generate all of the green circles in one go (rather than alternating). This is called breadth-first search.

The advantage of one of these approaches becomes very clear when our graph becomes a little more messy. Consider a Question 3 which asks for the length of the shortest path from the blue node to a particular node given to you; and then consider this graph, where there is no longer a clear sense of 'colour' (depth):

![The same graph, but with one of the green circles in the outer layer being connected directly to the central blue circle with a purple line, without an intermediate red circle. The green circle is also connected to the central blue circle via an alternative path which *does* contain an intermediate red circle.](http://u.cubeupload.com/jokebookservice1/bfsmessy.png)

The previous graph was also a *tree*. This graph is *not* a tree, which is why it's more messy.

The colour of this red-green circle (connected via a purple-coloured edge) is *red* if we're thinking only of the shortest path. However, if we used depth first search and it decided to generate the red circle first, which then led to the red-green circle, we wouldn't be able to say whether or not it was the shortest path (in this case, it wasn't). To be absolutely certain, we'd probably have to explore all of the other paths through the graph, and if we're unlucky, the path involving the purple edge will be the last to be checked.

If we instead generated all of the red circles first (i.e., all nodes which are distance 1 from the blue circle), we would immediately *know* that the shortest path is 1.

BFS and DFS won't necessarily be faster than one another, especially in their worst cases (which is all that matters in the BIO, because the test cases will try to make your program run as slow as desired), but BFS will tell you when you can stop searching for paths, because it will always find the shortest path first.

Let's start by writing a function which could output the nodes attached directly to a given node. This is the section of code which will actually depend on the particular Question 3 problem you're solving.

```python
def get_children(parent):
  return [parent * 2, parent // 2, parent + 1, parent - 1]
```

Then, let's write our breadth-first search algorithm. Our grey lines don't have a direction (formally, we've drawn an *undirected graph*), and especially with a graph that isn't a tree, we need to be very careful that we don't revist a node we've already seen; because recalculating its children is time-consuming.

Let's store our current "layer" in a list, where all elements in a "layer" have the same shortest distance to our origin node. We can initally start with the "blue" layer -- i.e., just the origin node itself.

```python
origin_node = 12
target_node = 7

layer = [origin_node]
seen = set(layer)

distance = 0
```

Next, we can try to build the next layer. As soon as we generate a child, we should mark it as seen, so that the same element does not appear twice in our layer (as that would mean we'd need to calculate children twice). To give an example of where this could happen, consider our current `get_children` function. To get to a child, we can add 1, subtract 1, double, or halve. However, if our parent is 1, then 1 + 1 = 2, but 1 * 2 = 2 also.

Also, it's probably a good idea to exit out of the loop as soon as we realise we've found our target. If we ran the check at the top of our `while` loop, we would generate the entire layer even if we found the target half-way through.

The BFS implementation will probably have lots of nested loops, so I like to wrap it in a function so that I can `return` in order to break out of all of them at once. This does mean we need to list any global variables that we plan on changing.

```python
def bfs():
  global seen, layer, distance
  
  if target_node == origin_node:
    return 0
  
  while True:
    next_layer = []
    for parent in layer:
      for child in get_children(parent):
        if child in seen:
          continue
        
        if child == target_node:
          return distance + 1
        
        next_layer.append(child)
        seen.add(child)
    layer = next_layer
    distance += 1
```

Notice that we add 1 to `distance`, because `distance` is the distance to the parent, not the child.

Our final code is hence

```python
def get_children(parent):
  return [parent * 2, parent // 2, parent + 1, parent - 1]

origin_node = 12
target_node = 7

layer = [origin_node]
seen = set(layer)

distance = 0

def bfs():
  global seen, layer, distance
  
  if target_node == origin_node:
    return 0
  
  while True:
    next_layer = []
    for parent in layer:
      for child in get_children(parent):
        if child in seen:
          continue
        
        if child == target_node:
          return distance + 1
        
        next_layer.append(child)
        seen.add(child)
    layer = next_layer
    distance += 1

print(bfs())
```

Indeed, this outputs 2 (which is correct, (÷ 2), (+ 1)).

This code could be improved by detecting when it is not possible to get from the parent to the `origin_node` to the `target_node`. If `layer` becomes empty (because all possible children are in `seen`), we know we have explored all of the layers. We could detect this by changing the while loop so that it starts

```python
while layer:
```

we could then place, under the while loop, `return "IMPOSSIBLE"`.

However, this is not usually needed, because most BIO questions will tell you that inputs are guaranteed to be valid.
