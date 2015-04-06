---
layout: post
title: Applications of networks
categories: [personal]
tags: [outdoors, hiking, travel, trip]
description: Applications of different networks
---
*modified repost from chengjun's original post*

## Introduction

Networks can be applied to various different problems that have far reaching real world applications. PopSci recently recognized Dr. Barabasi, one of the leaders of the field, as a "Man who could rule the world" http://www.popsci.com/science/article/2011-10/man-could-rule-world.

In this post we examine some of the basic theory behind the ideas, and look at how we can apply transmission models to them to investigate propagation of an idea/disease/product/trend throughout a population.

As the first step, the algorithm is quite simple:

1. Generate a network g: g(V, E).
2. Randomly select one or n nodes as seeds.
3. Each infected node influences its neighbors with probability p (transmission rate, $$\beta$$).


## SI model

Actually, this is the most basic epidemic model (SI model) which has only two states: Susceptible (S) and Infected (I). However, we will extend it to networks. 

SI model describes the status of individuals switching from susceptible to infected. In this model, every individual will be infected eventually. Considering a close population without birth, death, and mobility, and assuming that each agent is homogeneous mixing,  SI model implies that each individual has the same probability to transfer the something (e.g., disease, innovation or information) to its neighbors (T. G. Lewis, 2011).

Given the transmission rate $$\beta$$, SI model can be described as:

$$\frac{dS}{dt}=-\beta SI$$

$$\frac{dI}{dt}=\beta SI$$

Note that I + S = 1, the equation $$\frac{dI}{dt}=\beta SI$$ can be simplified as: 

$$\frac{dI}{dt}=\beta I(1-I)$$

Solve this equation, we can get a logistic growth function featured by its s-shaped curve. The logistic curve increases fast after it crosses the critical point, and grows much slower in the late stage. It can be used to fit the curve of diffusion of innovations. 

Note that the SI model is quite naive. In the real case of epidemic spreading, we have to consider how the status of the infected change: the infected can recover and become susceptible again (SIS model), or the infected can recover and get immune (SIR, $$\gamma$$ denotes the removal or recovery rate). 

In this post, I intend to bring the network back into the simulation of SI model using R and the package igraph.

## Generate the network

	require(igraph)
	# generate a social graph
	size = 50
	
	# regular network
	g = graph.tree(size, children = 2); plot(g)
	g = graph.star(size); plot(g)
	g = graph.full(size); plot(g)
	g = graph.ring(size); plot(g)
	g = connect.neighborhood(graph.ring(size), 2); plot(g) # 最近邻耦合网络
	
	# random network
	g = erdos.renyi.game(size, 0.1)

	# small-world network
	g = rewire.edges(erdos.renyi.game(size, 0.1), prob = 0.8 )
    # scale-free network
	g = barabasi.game(size) ; plot(g)
	


## Initiate the diffusers
	seeds_num = 1
	set.seed(2014); diffusers = sample(V(g),seeds_num) ; diffusers
	infected =list()
	infected[[1]]= diffusers

	# for example, set percolation probability 
	p = 0.128
	coins = c(rep(1, p*1000), rep(0,(1-p)*1000))
	n = length(coins)
	sample(coins, 1, replace=TRUE, prob=rep(1/n, n))
	


## Update the diffusers

	# function for updating the diffusers
	update_diffusers = function(diffusers){
	  nearest_neighbors = neighborhood(g, 1, diffusers)
	  nearest_neighbors = data.frame(table(unlist(nearest_neighbors)))
	  nearest_neighbors = subset(nearest_neighbors, !(nearest_neighbors[,1]%in%diffusers))
	  # toss the coins
	  toss = function(freq) {
	    tossing = NULL
	    for (i in 1:freq ) tossing[i] = sample(coins, 1, replace=TRUE, prob=rep(1/n, times=n))
	    tossing = sum(tossing)
	    return (tossing)
	  }
	  keep = unlist(lapply(nearest_neighbors[,2], toss))
	  new_infected = as.numeric(as.character(nearest_neighbors[,1][keep >= 1]))
	  diffusers = unique(c(diffusers, new_infected))
	  return(diffusers)
	  }
	

	
## Start the contagion!
R you Ready? Now we can start the contagion!

	i = 1
	while(length(infected[[i]]) < size){ 
	  infected[[i+1]] = sort(update_diffusers(infected[[i]]))
	  cat(length(infected[[i+1]]), "\n")
	  i = i + 1
	}

Let's look at the diffusion curve first:

![](http://farm8.staticflickr.com/7299/12845959103_e19cd9cd99_n.jpg)

To visualize the diffusion process, we label the infected nodes with the red color.

	E(g)$color = "blueviolet"
	V(g)$color = "white"
	set.seed(2014); layout.old = layout.fruchterman.reingold(g) 
	V(g)$color[V(g)%in%diffusers] = "red"
	plot(g, layout =layout.old)

I make the animated gif using the package animation developed by Yihui Xie.

	library(animation)
	
	saveGIF({
	  ani.options(interval = 0.5, convert = shQuote("C:/Program Files/ImageMagick-6.8.8-Q16/convert.exe"))
	  # start the plot
	  m = 1
	  while(m <= length(infected)){
	    V(g)$color = "white"
	    V(g)$color[V(g)%in%infected[[m]]] = "red"
	    plot(g, layout =layout.old)
	    m = m + 1}
	})


![](http://farm4.staticflickr.com/3806/12826172695_368a6f50a2_o.gif)

![](http://farm3.staticflickr.com/2848/12826237753_d8c97b1019_o.gif)

![](http://farm4.staticflickr.com/3729/12826584654_c84452f397_o.gif)

![](http://farm3.staticflickr.com/2851/12826173505_34649f488d_o.gif)

![](http://farm8.staticflickr.com/7391/12826173255_574e471023_o.gif)

![](http://farm4.staticflickr.com/3675/12826584484_7c6f35380c_o.gif)

![](http://farm8.staticflickr.com/7432/12826173045_ef3548ec04_o.gif)

![](http://farm4.staticflickr.com/3672/12848749413_7f9da8b8c7_o.gif)

