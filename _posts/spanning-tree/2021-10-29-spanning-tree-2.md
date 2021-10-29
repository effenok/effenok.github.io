---
layout: post title:  "Spanning Trees: Basic Tree"
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

## Basic Tree

The first algorithm constructs *any* spanning tree, rooted at a pre-defined node. This is a very simple algorithm. Root
node sends a special message, let's call it `build-tree` to all its neighbors. Each neighbor receives the message, sets
its parent node to the sender of the message and then sends its own `build-tree` message to all of its neighbors except
for the parent. Note, that nodes (ehm, i mean processes) can receive multiple `build-tree` messages. In the simple
algorithm, each node selects the sender of the first received  `build-tree` message to be its parent.

Let's define the algorithm more formally. First, each node has a unique identifier. Second, each node wants to know the
current root. Well, let's assume that it needs to, although it is not strictly necessary for this algorithm. Third, each
node saves its parent node. The root node starts with root equal to itself, and null parent. This will never change. All
other nodes do not know the values of either root or parent, thus they start with these values being null and fill them
later.

Nodes exchange the `build-tree`
message. This message needs to carry the id of the root node.

```
// Basic spanning tree algorithm

variables = { 
    uid: <random unique id>, 
    parent: uid or null, initially null, 
    root: uid or null, initially node's id for root node and null for other nodes
}

messages = build-tree {root: uid}

start: 
    if /* node is root */ self.root != null
        send build-tree to all neighbors

on receive build-tree {root} from neighbor j
    if self.parent == null
        self.parent = j
        self.root = build-tree.root
        send build-tree to all neighbors except j
```

Ok, too much text, but here is a simple spanning tree. What have we achieved, and what not?

- **achieved**
    - construct a spanning tree, where each node knows its parent
- **not achieved**
    - tree is not a shortest path tree or anything, it can be suboptimal.
    - tree is arbitrary, and there is no control on the structure of the tree

{: .notification } One thing to note about this algorithm, is that each node knows when it achieves a quiescence state.
Informally, quiescence state is a state in which internal state of the node does not change. Here it means that each
node knows when the construction of the tree has stabilized. Yes, it is trivial here, but this is something that is not
that trivial when constructing trees based on distance.

- **achieved**
    - each node knows when the tree is stable

<div class="container has-text-centered">
{% digraph the tree could look like this%}
layout=dot;
rankdir="BT";
a [shape="doublecircle",xlabel="root"]
b [label="b\nparent=a"]
c [label="c\nparent=b"]
d [label="d\nparent=b"]
e [label="e\nparent=c"]
b -> a [penwidth=2]
c -> b [penwidth=2]
d -> b [penwidth=2]
e -> c [penwidth=2]
e -> d [arrowhead="none"]
{% enddigraph %}
<p> resulting spanning tree</p>
</div>

### For state machine lovers

If you like state machines, I can also define states and variables that make sense in these states. Although these
states will not make much sense in the next two algorithms below. So, root can have a special state `root`. For each
other node something happens for the first received `build-tree` message. Let's define two states - `unmarked` and
`marked`, for nodes that have not yet received or have already received their first `build-tree` message. Since there
are states, i will attach state variables to states. This can be done in Rust.

```
// Basic spanning tree algorithm with states

states = {root, unmarked, marked }

variables = { 
    uid: <random unique id>, 
    
    in state marked {
        root: uid of root node
        parent: uid of parent node
    }
}

messages = build-tree {root: uid}

start: 
    if in state root: 
        send build-tree to all neighbors

on receving build-tree {root} from neighbor j
    if in state unmarked
        self.parent = j
        self.root = build-tree.root
        send build-tree to all neighbors except j
    else 
        ignore the message
```

### How it looks

The algorithm does something like this

<div class="container has-text-centered">
{% digraph %}
node[shape="circle"]
a [shape="doublecircle",xlabel="root"]
b -> a [arrowhead="none",label="m->"]
c -> b [arrowhead="none"]
d -> b [arrowhead="none"]
e -> c [arrowhead="none"]
e -> d [arrowhead="none"]
{% enddigraph %}

{% digraph %} node[shape="circle"]
a [shape="doublecircle",xlabel="root"]
b [xlabel="parent=a"]
b -> a [penwidth=2]
c -> b [arrowhead="none",label="m->"]
d -> b [arrowhead="none",label="m->"]
e -> c [arrowhead="none"]
e -> d [arrowhead="none"]
{% enddigraph %}

{% digraph %} node[shape="circle"]
a [shape="doublecircle",xlabel="root"]
b [xlabel="parent=a"]
c [xlabel="parent=b"]
d [xlabel="parent=b"]
b -> a [penwidth=2]
c -> b [penwidth=2]
d -> b [penwidth=2]
e -> c [arrowhead="none",label="m->"]
e -> d [arrowhead="none",label="m->"]
{% enddigraph %}

{% digraph %} node[shape="circle"]
a [shape="doublecircle",xlabel="root"]
b [xlabel="parent=a"]
c [xlabel="parent=b"]
d [xlabel="parent=b"]
e [xlabel="parent=c"]
b -> a [penwidth=2]
c -> b [penwidth=2]
d -> b [penwidth=2]
e -> c [penwidth=2]
e -> d [arrowhead="none"]
{% enddigraph %}

</div>