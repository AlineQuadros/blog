---
layout: post
title:  "Network analysis of bacterial communities"
author: aline
categories: [ ecology, data-science, dataviz]
image: assets/images/microbialnets.jpg
tags: [featured]
---


Network analysis has been increasingly used to understand relationships between entities. Although the method itself and all the network metrics have been around for a long time, recent improvements of *visualization techniques* really made this tool more popular.

In ecology, network analysis can be used to study the interactions between species, and to understand the organization of ecological communities.

Networks are composed of **nodes** (and their attributes) and **edges** (and their attributes). To build a network, these two datasets are necessary. For example, the project showed above, the nodes are the species (actually OTUs), and the edges represent associations between them. Attributes of **nodes** could be ID, species name, species function, habitat, etc. Attributes of **edges** are any  attribute indicating the strength of an association, or volume of or the nature of the association. In my case, they were "negative" (indicating a negative association), or "positive". More commonly, **edges** have a `weight` attribute that is used to determine the thickness of the lines. Based on the relationships between these nodes and edges, a series of **metrics** can be obtained to describe the network.

In this project I used *network analysis* to study the organization of microbial communities found in rice fields contaminated with pesticides. The goal was to identify, among thousands of OTUs, groups (networks) of co-occurring microorganisms, and groups of antagonistic groups. The identification of co-occurring groups indicate the formation of consortia and/or facilitation between the microorganisms. Here's one of the main results:

<img src="/blog/assets/images/net1.png">

The black lines indicate a strong association (Spearman correlation coefficient > 0.8), whereas the red lines indicate a strong negative association, i.e., groups of organisms that rarely co-occurred in a sample. Figure produced using Gephi (see below).

To build this visualization I used <a href="https://gephi.org/">GEPHI</a>, and *I highly recommend it*. Although there are packages that enable the visualization of networks inside R (and take my word, I tried all of them before adopting Gephi), none of them can produce figures as above in less than a few minutes and with minimum configuration.

>Gephi is the leading visualization and exploration software for all kinds of graphs and networks. Gephi is open-source and free. Runs on Windows, Mac OS X and Linux.


### How to organize the data for network analysis and export it to GEPHI using R

After calculating the Pearson correlation between all OTUs (correlation matrix), filtering (to keep only the strongest correlations, with Pearson coef. larger than 0.80 or  smaller than -0.8), and re-arranging it into a dataframe, this was how the data  `corr_dataset` looked like:

```
OTU_1     OTU_2     coef_correlation
16s_0001  16s_0001  0.8435
16s_0001  16s_0002  0.8908
16s_0001  16s_0003  -0.9866
16s_0001  16s_0004  0.9744
16s_0001  16s_0005  0.8033
16s_0001  16s_0005  0.8675
16s_0001  16s_0006  0.8035
```

To organize the `nodes` dataset, you'll need to get a list of your entities. This is generally the first column of your `nodes` dataset. The other columns (optional, but really useful) will store the `attributes` of each node. In my case, each entity (microbial "species") had the following attributes: `gross_id` (keeping the species identification at a coarser level, like Order), `fine_id` (keeping the species identification at a finer level, like Genus), and `label` containing a short string to be printed next to the nodes in the network.

```
# get the list of entities
nodes <- as.data.frame(unique(c(OTU_1,OTU_2)))

# set attributes by including different columns to the dataframe
nodes$label
nodes$gross_id
nodes$fine_id

```

Import the data into the formats used by `graph`:

```
# Create a graph object. Use simplify to ensure that there are no duplicated edges or self loops
gD <- simplify(graph.data.frame(edges, directed=FALSE))

# Print number of nodes and edges
vcount(gD)
ecount(gD)
```

Organize the `nodes` dataset:

```
# Set node attributes (aka vertex)
gD <- set.vertex.attribute(gD, "label", index = V(gD), value = nodes$label)
gD <- set.vertex.attribute(gD, "gross_id", index = V(gD), value = nodes$gross_id)
gD <- set.vertex.attribute(gD, "fine_id", index = V(gD), value = nodes$fine_id)

# Create a dataframe with node attributes
nodes_att <- data.frame(gross_id = V(gD)$gross_id,
                        label_short=V(gD)$label_short,
                        fine_id=V(gD)$fine_id)
```
Organize the `edges` dataset:

```
# Set edge attributes
gD <- set.edge.attribute(gD, "association", index = E(gD), value = edges$assoc)

# Create a dataframe with edges attributes
edges_att <- data.frame(TYPE = E(gD)$association)

```

To load your network data in to `Gephi` it's necessary to convert the `nodes`, `edges` and their `attributes` into a *"gefx."* file. It can be done easily with the package `rgexf` for R:

```
library("rgexf")
write.gexf(nodes = nodes[, c(1,3)], edges = edges[, 1:2],  
            nodesAtt = nodes_att, edgesAtt = edges_att, edgesWeight=edges[,3],
            defaultedgetype = "undirected", output = "_your_filename.gexf")
```

That's it! Once you import you *"gefx."* file into Gephi you'll have full control over t. Gephi lets you change the colors, edge thickness, and node sizes based on attributes (like the ones you exported from R), or based on network metricks such as `degree` and `betweenness`.
