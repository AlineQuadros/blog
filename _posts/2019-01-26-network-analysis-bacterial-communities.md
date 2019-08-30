---
layout: post
title:  "Network analysis of bacterial communities"
author: Aline
categories: [ ecology, data-science, dataviz]
image: assets/images/rice-network.png
tags: [featured]
---


Network analysis has been increasingly used to understand relationships between entities. Although the method itself and all the network metrics have been around for a long time, recent improvements of *visualization techniques* really

In ecology,

Networks are composed of **nodes** (and their attributes) and **edges** (and their attributes). To build a network, these two datasets are necessary. For example, the project showed above, the nodes are the species (actually OTUs), and the edges represent associations between them. Attributes of **nodes** could be ID, species name, species function, habitat, etc. Attributes of **edges** are any  attribute indicating the strength of an association, or volume of or the nature of the association. In my case, they were "negative" (indicating a negative association), or "positive". More commonly, **edges** have a `weight` attribute that is used to determine the thickness of the lines.

This was how my original `corr_dataset` looked like:
```
OTU_1   OTU_2   coef_correlation


```

It is necessary that the nodes should be unique
```
nodes <- as.data.frame(unique(c(a1,a2)))
```

```
# Create a graph. Use simplify to ensure that there are no duplicated edges or self loops
gD <- simplify(graph.data.frame(edges, directed=FALSE))
# Print number of nodes and edges
vcount(gD)
ecount(gD)
nodes$gross_id <- as.character(nodes$gross_id)
nodes$fine_id <- as.character(nodes$fine_id)
# Add new node attributes
gD <- set.vertex.attribute(gD, "gross_id", index = V(gD), value = nodes$gross_id)
gD <- set.vertex.attribute(gD, "label_short", index = V(gD), value = nodes$label_short)
gD <- set.vertex.attribute(gD, "fine_id", index = V(gD), value = nodes$fine_id)
gD <- set.vertex.attribute(gD, "top", index = V(gD), value = nodes$top)
gD <- set.vertex.attribute(gD, "presence", index = V(gD), value = nodes$presence)
# Create a dataframe with node attributes
nodes_att <- data.frame(gross_id = V(gD)$gross_id, label_short=V(gD)$label_short, fine_id=V(gD)$fine_id, top = V(gD)$top, presence = V(gD)$presence)
 # Add new edge attributes
gD <- set.edge.attribute(gD, "association", index = E(gD), value = edges$assoc)
# Create a dataframe with edges attributes
edges_att <- data.frame(TYPE = E(gD)$association)


```

<img src="blog/assets/images/net1.png">

> Networks of associated microorganisms. The black lines indicate a strong association (Spearman correlation coefficient > 0.8), whereas the red lines indicate a strong negative association, i.e., groups of organisms that rarely co-occurred in a sample. Figure produced using Gephi.

To build this visualization I used <a href="https://gephi.org/">GEPHI</a>, and *I highly recommend it*. Although there are packages that enable the visualization of networks inside R (and take my word, I tried all of them before adopting Gephi), none of them can produce figures as above in less than a few minutes and with minimum configuration.

>Gephi is the leading visualization and exploration software for all kinds of graphs and networks. Gephi is open-source and free. Runs on Windows, Mac OS X and Linux.

it's necessary to convert the `nodes` and `attibutes` dataframes into a *"gefx."* file. It can be done easily with the package `rgexf` for R:

```
library("rgexf")
write.gexf(nodes = nodes[, c(1,3)], edges = edges[, 1:2],  nodesAtt = nodes_att, edgesAtt = edges_att, edgesWeight=edges[,3], defaultedgetype = "undirected", output = "_your_filename.gexf")
```





```  

That's it! Once you import you *"gfx."* file into Gephi
