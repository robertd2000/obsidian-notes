Solved : Find Shortest Distance ie Shortest Path between two nodes in weighted graph

Although Golang is not designed to address complex business use cases, But gives you flexibility to easily code Data stuctures.

Lets find shortest distance path between nodes A and G

![](https://miro.medium.com/v2/resize:fit:579/1*UobmHNzNCrdiNiFdoLl4Eg.png)

By looking at figure we can easily say that shortest path between A and G is A -> F -> G since distance 4 is lowest between A and G.

How easily by looking at image we can answer , similarly dijkstra algorithm helps to have answer programatically

Algorithm

1. Create a Distance Map Storing Distance from source Node to Each Nodes, Initialize all the valuse to Infinity
2. Create a Vertex Map to mark vertexes as visited and non visited
3. For source Node mark distance as zero (weights should be non negative)
4. Enqueue the source Node in min Priority queue , Priority Queue is based on distance
5. Dequeue the Node from queue and visit all unvisited neighbours , calculate tentative distances through current node, if tentative distance is less than current assign distance , update the distance and assign the smaller one, enqueue that smallest distance vertex in priority queue
6. To Get shortest Path in step 5 store previous node name
7. Repeat this untill priority queue is empty

It’s that simple

## Get Rishabh Mishra’s stories in your inbox

Join Medium for free to get updates from this writer.

Subscribe

Remember me for faster sign in

==Implementation for Minimum Priority queue in Golang==

Create Graph

Dijkstra Algorithm

Happy learning !!!

Do checkout API to get shortest Path using Dijkstra Algorithm

[

## GitHub - rishabh625/graphs

### Contribute to rishabh625/graphs development by creating an account on GitHub.

github.com



](https://github.com/rishabh625/graphs?source=post_page-----7bf2722ba0c8---------------------------------------)

Please show some support