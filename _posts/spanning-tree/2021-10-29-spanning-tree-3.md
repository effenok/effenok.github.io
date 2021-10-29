---
layout: post 
title:  "Spanning Trees: Breadth-First-Search Tree"
categories: spanning-tree
---

Spanning Trees:
<ul>
{% assign sortedPosts = site.categories['spanning-tree'] | sort: 'url' %}

{% for post in sortedPosts %} {% if post.title == page.title %}
<li>{{ post.title }}</li>
{% else %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endif %}
{% endfor %}
</ul>

## Breadth-First-Search Tree

Now, that we know how to construct any spanning tree, let's construct a breadth-first-search (_bfs_) tree. Well, this
version is somewhat different from the "traditional" breadth-first-search algorithm. Traditional algorithm visits nodes
in the specific order, such that nodes that are direct neighbors of the root are visited first, then their neighbors,
and so on. In a distributed version we want to construct a tree, such that each node's parent is visited before the node
in the bfs order. Well, this super smart statement basically says that each parent is closer to the root in number of
edges (sorry, again too smart). Anyways...

<div class="level">
    <div class="level-item">
    <div class="container">
        <p class="title is-6">this is not a bfs tree</p>

{% digraph a tree%} layout=dot; rankdir="BT"; edge [labelstyle="above, sloped"]; a [shape="doublecircle"]
b [label="b\nparent=a"]
c [label="c\nparent=b"]
d [label="d\nparent=c"]
b -> a [penwidth=2]
c -> b [penwidth=2]
d -> c [penwidth=2]
d -> a  [arrowhead="none"]
{% enddigraph %}

    </div> 
    </div> 

    <div class="level-item">
    <div class="container has-text-centered">
    <p class="title is-6">this is a bfs tree</p>

{% digraph BFS tree%} layout=dot; rankdir="BT"; edge [labelstyle="above, sloped"]; a [shape="doublecircle"]
b [label="b\nparent=a"]
c [label="c\nparent=b"]
d [label="d\nparent=a"]
b -> a [penwidth=2]
c -> b [penwidth=2]
d -> c [arrowhead="none"]
d -> a [penwidth=2]
{% enddigraph %}

    </div>
    </div>

</div>

So, how do we do it. Well, we introduce distance to the root, in number of edges traversed, have each node keep track of
its distance, have messages carry the distance, and have nodes correct their parent pointers, when they receive messages
with smaller distance. Pfeu, I use too many too smart words today. Let's try again.

So, how do we do it. We introduce distance to the root, that each node tracks of. Each node will save its distance to
its current parent. To calculate this distance, the `build-tree` message will carry sender's distance to the root. The
receiver, calculates its distance by adding one to the message. Also, each node will change its parent, if it sees a
message with smaller distance to the root.

Root obviously has distance zero. And root sends its `build-tree` message with distance zero to its neighbors. Each
neighbor will set its distance to the distance received in message plus one. Then it will send its `build-tree`
message with distance one to all neighbors. And so on. When a node receives `build-tree` message, it will check the
distance in the message, and, if the distance is smaller change its parent.


<div class="language-plaintext highlighter-rouge">
<pre class="highlight">
<code>
// Breadth-First Search Algorithm

variables = { uid: &lt;random unique id&gt;, parent: uid or null, initially null,
<em>distance: number or infinity, initially zero for root node and infinity for everyone else</em>
root: uid or null, initially node's id for root node and null for other nodes }

messages = build-tree {root: uid, <em>distance: number</em>}

start:
if /* node is root */ self.root != null send build-tree {root = self.uid, <em>distance = 0</em>} to all neighbors

on receive build-tree {root, distance} from neighbor j

    <em>is_better_parent := (build-tree.distance + 1) &lt; self.distance</em>

    <em>if is_better_parent</em>
        self.parent = j
        <em>self.distance = build-tree.distance + 1</em>
        self.root = build-tree.root
        send build-tree {root = self.root, <em>distance = self.distance</em>} to all neighbors except j

</code>
</pre>
</div>

Basically, what we did, was introduced metric, called _hop-count_, and calculate the tree based on this metric. One can
say that BFS is Bellman-Ford with *hop count* distance. 
{: .notification }

Here's our tree

<div class="container has-text-centered">
{% digraph BFS tree%}
layout=dot;
rankdir="BT";
edge [labelstyle="above, sloped"];
a [shape="doublecircle"]
b [label="b\nparent=a\nd=1"]
c [label="c\nparent=b\nd=2"]
d [label="d\nparent=a\nd=1"]
b -> a [penwidth=2]
c -> b [penwidth=2]
d -> c [arrowhead="none"]
d -> a [penwidth=2]
{% enddigraph %}

</div>


Ok, too much text, but here is a simple spanning tree. What have we achieved, and what not?

- **achieved**
    - construct a spanning tree, where each node knows its parent and its hop count to root
    - tree is the shortest path with respect to hop count
- **not achieved**
    - can only support hop-count.

### More determinism

There is one small issue, this tree is somewhat arbitrary, that is, if the node receives several build-tree messages
with the same distance, there is no way of knowing which one the node will select. Recall the tree from part 2. This
tree is still arbitrary in a sense that node *e* can select either *c* or *d* as its parent.

<div class="container has-text-centered">
{% digraph the tree could look like this%}
layout=dot;
rankdir="BT";
a [shape="doublecircle"]
b [label="b\nparent=a\nd=1"]
c [label="c\nparent=b\nd=2"]
d [label="d\nparent=b\nd=2"]
e [label="e\nparent=c\nd=3"]
b -> a [penwidth=2]
c -> b [penwidth=2]
d -> b [penwidth=2]
e -> c [penwidth=2]
e -> d [arrowhead="none"]
{% enddigraph %}
</div>

We can solve this by using some kind of tie-breaking. The common approach is to use tie-breaking based on node
identifiers. For this, node identifiers need to be comparable. To be more precise we need the set of identifiers to be
totally orderable, i.e., for every two elements one can say which one is smaller. Ok, this is not a definition, but you
can check wikipedia for that. Let's say that these unique ids are numbers. Numbers can be totally ordered, i. e., one
can say which number is smaller and which is larger. I think everything in the computer can be viewed as n-bit number
and can be compared based on it.

Let's change the code

<div class="language-plaintext highlighter-rouge">
<pre class="highlight"><code>
on receive build-tree {root, distance} from neighbor j

<em>my_distance_via := build-tree.distance + 1 is_better_parent := my_distance_via < self.distance || (my_distance_via
== self.distance && j < self.parent)</em>

<em> if is_better_parent </em>
    self.parent = j
    self.root = build-tree.root
    send build-tree to all neighbors except j
</code> </pre></div>

Now, under assumption that there is only one link between each pair of nodes, the tree is fully deterministic.

We haven't fixed non-determinism in this situation, but we will skip it for now.
<div class="container has-text-centered">
{% digraph %}
layout=dot;
rankdir="BT";
a [shape="doublecircle"]
b [label="b\nparent=a\nd=1"]
c [label="c\nparent=b\nd=2"]

b -> a [penwidth=2]
c -> b [penwidth=2]
c -> b [arrowhead=none]
{% enddigraph %}
</div>
