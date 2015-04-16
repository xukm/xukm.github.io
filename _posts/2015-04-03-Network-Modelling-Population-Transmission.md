---
layout: post
title: Applications of networks
categories: 
  - personal
tags: 
  - code
  - modelling
  - python
description: Applications of different networks
published: true
---

## Introduction

Networks can be applied to various different problems that have far reaching real world applications. PopSci recently recognized Dr. Barabasi, one of the leaders of the field, as a "Man who could rule the world" http://www.popsci.com/science/article/2011-10/man-could-rule-world.

In this post we examine some of the basic theory behind the ideas, and look at how we can apply transmission models to them to investigate propagation of an idea/disease/product/trend throughout a population.

As the first step, the algorithm is quite simple:

1. Generate a network g: g(V, E).
2. Generate a seed network that gives the basis for generation
3. Each infected node at each time step checks to see if it transmits to its neighbour


## SI model

This is a basic epidemiological model where individuals exist as either one of two states, infected or susceptible. Through this model we assume that everyone in the population, so long as they are connected to another person will become infected eventually. In a static population without changes in the number of people inside, this model implies that every individual has the same probability to transfer the something (e.g., disease, innovation or information) to its neighbors (T. G. Lewis, 2011).

Given the transmission rate $\beta$, SI model can be described as:

$\frac{dS}{dt}=-\beta SI$

$\frac{dI}{dt}=\beta SI$

These two equations yield a logistic growth function when solved. The defining characteristic of the logistic growth function is its sigmoidal shape at which there is a critical point from which growth becomes much more rapid. This can be thought of as the tipping point for a trend on twitter, the adoption of a new generation of product, the point at which confidence in a market crashes, etc.

In this post, I use the below python code to generate a scale free network using Barabasi-Albert population growth to demonstrate how SI would interact with a population.

## Generate the network

	import random
	def BA_graphgen(nodes, edges, seed_size, seed_edge_density):

        graph = []
        numEdges = 0 # tracking variable to count total number of edges
        if edges < 1 or edges >= (seed_size + 1):
            raise Exception("Number of edges must be equal to greater than 1 and less than seed size + 1 number of nodes")
        if seed_edge_density <= 0 or type(seed_edge_density) != int: # check to see that the
            raise Exception("Seed edge density must be greater than zero and be an integer.")

        # GENERATE ZEROS SEED
        A=[] #initialize blank list
        for i in range(seed_size): # set column length
            for j in range(seed_size): # set row length
                    A.append(0) 
            graph.append(A) # add this as a new row
            A = [] # blank list A again

        # Generate edges between nodes for seed graph
        for i in range(len(graph[0])): 
            for j in range(len(graph[0])): 
                rand = random.randint(0,10) # determine connection probability
                if rand % (10 - seed_edge_density) != 0: 
                    graph[i][j] = graph[j][i] = 1 # set the connection to be 1, indicating an edge of weight 1
                    numEdges += 1 # total degree counter increase by 1
                if i == j: 
                    graph[i][j] = graph[j][i] = 1 # see above
                    break

        # Add new nodes and attach them
        for n in range(nodes-seed_size): 
            for i in range(len(graph)): # check the row length of the current graph
                graph[i].append(0) # add zeros to the end of every row
            graph.append([0]*len(graph[0])) # add another row of zeros
            graph[-1][-1] = 1 
            node = n + seed_size 
            NodeEdges = 0 # each node needs to have an "edges" number of edges
            while NodeEdges < edges: # while we have less edges than desired for that new node
                node_visit = random.randint(0,len(graph[0])-1) # randomly visit another node in the population
                p = sum(graph[node_visit])/numEdges
                if p > random.random() and graph[node][node_visit] != 1: 
                    graph[node][node_visit] = graph[node_visit][node] = 1 # set an edge
                    numEdges += 1 # increase number of total edges
                    NodeEdges += 1 # increas number of edges for the current node
        return graph # return the generated adjacency matrix

#### Visual
![](http://farm4.staticflickr.com/3672/12848749413_7f9da8b8c7_o.gif)

Image from [cheng-jun's post](http://chengjun.github.io/en/2014/03/simulate-network-diffusion-with-R/) published under [(CC) BY-NC-SA](http://creativecommons.org/licenses/by-nc-sa/3.0/)