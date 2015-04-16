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

import random

def BA_graphgen(nodes, edges, seed_size, seed_edge_density): # function to generate a population adjacency graph in B-A distribution

	graph = []
	numEdges = 0 # tracking variable to count total number of edges
	if edges < 1 or edges >= (seed_size + 1): # check to see we are generating edges and that there aren't more edges than nodes initially in seed
		raise Exception("Number of edges must be equal to greater than 1 and less than seed size + 1 number of nodes")
	if seed_edge_density <= 0 or type(seed_edge_density) != int: # check to see that the
		raise Exception("Seed edge density must be greater than zero and be an integer.")

	# GENERATE ZEROS SEED
	A=[] #initialize blank list
	for i in range(seed_size): # set column length
		for j in range(seed_size): # set row length
				A.append(0) # add zeros equivalent to seed size desired, this generates the number of columns
		graph.append(A) # add this as a new row
		A = [] # blank list A again

	# Generate edges between nodes for seed graph
	for i in range(len(graph[0])): # for each element of the seed graph (row or col, doesn't matter due to symmetry)
		for j in range(len(graph[0])): # for each element of the seed graph (row or col, doesn't matter due to symmetry)
			rand = random.randint(0,10) # determine connection probability
			if rand % (10 - seed_edge_density) != 0: # seed edge density is an arbitrary number that determines how much linkage goes on, lower numbers indicate lower density
				graph[i][j] = graph[j][i] = 1 # set the connection to be 1, indicating an edge of weight 1
				numEdges += 1 # total degree counter increase by 1
			if i == j: # when col and row indicies are the same, set the connection to 1 as a node must be connected to itself
				graph[i][j] = graph[j][i] = 1 # see above
				break

	# Add new nodes and attach them
	for n in range(nodes-seed_size): # add nodes equal to total node wanted subtract the nodes already in the seed graph
		for i in range(len(graph)): # check the row length of the current graph
			graph[i].append(0) # add zeros to the end of every row
		graph.append([0]*len(graph[0])) # add another row of zeros
		graph[-1][-1] = 1 # set the bottom right most element to 1, this is because all connections are connected to themselves
		node = n + seed_size # set current node position to be n, which is the index of new node, plus the original seed size index
		NodeEdges = 0 # each node needs to have an "edges" number of edges
		while NodeEdges < edges: # while we have less edges than desired for that new node
			node_visit = random.randint(0,len(graph[0])-1) # randomly visit another node in the population
			p = sum(graph[node_visit])/numEdges # the probability of attachment is equal to the number of edges the visited node has over the total number of edges
			if p > random.random() and graph[node][node_visit] != 1: # if there isn't already a connection and the generated number is less than the probability of attachment, create an edge
				graph[node][node_visit] = graph[node_visit][node] = 1 # set an edge
				numEdges += 1 # increase number of total edges
				NodeEdges += 1 # increas number of edges for the current node
	return graph # return the generated adjacency matrix
	


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


![](http://farm4.staticflickr.com/3675/12826584484_7c6f35380c_o.gif)

![](http://farm8.staticflickr.com/7432/12826173045_ef3548ec04_o.gif)

![](http://farm4.staticflickr.com/3672/12848749413_7f9da8b8c7_o.gif)

