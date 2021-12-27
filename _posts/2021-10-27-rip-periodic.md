---
layout: post 
title:  "Distance-Vector and Periodic Broadcast?"
categories: notes
---

I am trying to finally start the blog, and unless I start with something small, I will never start at all. So, here it is. 
{: class="note"}

I think everyone knows that RIP periodically sends routing updates. True!

However, it seems to me, that a lot of times this statement is taught like it is a ground truth for distance-vector
protocols in general. And this is not true! There is nothing, absolutely nothing, requiring a distance vector protocol
to send messages periodically. RIP does this, which is a design choice of RIP.

EIGRP does not!. EIGRP does neighbor discovery, reliable transmissions, and only sends triggered updates on changes.
