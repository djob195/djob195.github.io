---
layout: post
title: "The Data Structures Behind Smarter Product Suggestions"
date: "2026-01-26 09:00:00"
categories: cases
tags: tree multipath supervised categorization
---

**Have you ever searched for a specific product in your favorite store only to realize that its price is far beyond your budget?** Almost instantly, the platform suggests a set of similar products, and you end up finding one that meets the same requirements at a much lower price.

This is not a coincidence. Behind that experience lies a well-defined architecture, and in this post, I will explain how it works.

## Trees as a Data Structure

To begin with, we need to talk about one of the most fundamental data structures in computer science: trees.
A tree is composed of entities called nodes, which store all the metadata required to build the structure. Within a tree, we can identify the following types of nodes:

* Root node: the main node that acts as the entry and exit point of the data.
* Non-leaf nodes: nodes that have one or more child nodes.
* Leaf nodes: terminal nodes that do not have any children.

![example tree](/assets/posts/2026-01-26/tree-example.png "example tree")

In general terms, a tree is a collection of one or more nodes, and each node may contain one or more keys that act as decision factors for organizing the structure.

## Multipath Trees

Building on this idea, we can define our records using a technique used in supervised learning: labeling.
Each record can be assigned one or more labels organized hierarchically, starting from macro-level categories (more general) and progressing down to micro-level categories (more specific), where the essence of the product is defined.

As a result, we end up constructing a categorization tree, where:

* Non-leaf nodes represent labels or categories.
* Leaf nodes contain the final records that share common characteristics.


## Design Considerations and Trade-offs

At first glance, this structure may seem easy to implement—and it is. However, its stability strongly depends on defining all labels and hierarchical levels from the beginning.

Adding, removing, or modifying a hierarchical level later can require a significant redesign, as the tree may need to be rebuilt from scratch, adapting all records to the new structure.

This leads to high space complexity when dealing with large datasets and can also impact the user experience.

## Real-World Case: Food Delivery Platform

A practical example of this approach comes from my experience working at a company focused on food delivery services, primarily for restaurants. The company identified an opportunity to implement an e-commerce solution that would allow each restaurant to upload its own product catalog and enable customers to browse those products from a web or mobile platform, ultimately placing orders for home delivery.

The solution was designed in a way similar to platforms like Shopify or WooCommerce, where:

* Each food product could configure all required metadata, such as name, price, image, and more.
* Each item was categorized using three hierarchical labels: macro category, category, and micro category.

Since all restaurants shared the same core requirements, the same module was replicated across the platform. The only difference was the addition of an extra hierarchical label placed before the macro category, representing the restaurant itself.

![case tree](/assets/posts/2026-01-26/tree-case.png "case tree")


This design made it possible to centralize all customers within a single platform while still allowing each restaurant to maintain its own categorization tree. By leveraging the leaf nodes of each tree, the platform could surface similar products and enhance the overall product discovery experience.

## Resourses
* [Categorizarion tree simulation](https://www.kaggle.com/code/djorellanab/tree-based-product-recommendation-using-market-seg)