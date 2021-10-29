---
layout: post title:  "Spanning Trees: Introduction"
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

## Introduction

Constructing spanning trees is a basic building block in all distance-vector routing protocols as well as STP/RSTP. All
these protocols can be described as based distributed Bellman-Ford. Well, STP is not distance-vector protocol, since
there are no vectors. I think of this series as a bridge between distributed algorithms for constructing spanning trees
and how they are used in network protocols.

### Distributed Algorithm

Here, I will use some typical notation commonly used in distributed systems. The system is modelled as a graph G=(V, E).
V is a set of vertices, and vertices are interconnected by edges, where each edge connects two vertices. Each vertex is
associated with a *process* and each edge is associated with a *channel*. Informally, a *process* has some state
variables, and is communicating with other processes by sending and receiving *messages* on channels. Channel delivers
messages from the sending process to receiving process.

{: .notification .is-info} Keep in mind, that network protocols work on different assumptions about the underlying
network. I will talk about the model later.

<div class="container has-text-centered">
{% graph %}
    node[shape="circle"]
    a [xlabel="a is a process, residing\nat graph node"]
    a -- b
    b -- c
    c -- d [label="channel"]
    d -- b
{% endgraph %}

<p> distributed system</p>
</div>

There are different models that describe properties of channels, depending on what channels can do. Typical cases are
message loss, message duplication, reordering, etc. For now, I will assume that the channels are reliable and are not
reordering (also known as *reliable FIFO channel*). That is, every sent message is guaranteed to be delivered and
messages are delivered in the same order in which they are sent.

Also, at the beginning I will assume that one node is configured to be the root of the spanning tree. I also assume that
each process has a unique id. It is an arbitrary number.

Ok, let's build some trees, shall we.