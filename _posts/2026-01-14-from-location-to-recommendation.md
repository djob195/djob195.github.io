---
layout: post
title: "Engineering Geographic Customer Recommendations at Scale"
date: "2026-01-14 09:00:00"
categories: cases
tags: clustering distances unsupervised supervised learning
---
{% include mathjax.html %}

You have probably browsed delivery applications and noticed features such as finding the nearest restaurants to your location in just a few seconds. Behind this apparent simplicity lies a combination of artificial intelligence and geolocation techniques that make this experience possible.

In this blog, I will explain in a simple way how some of these techniques work, focusing mainly on supervised and unsupervised learning, applied to the restaurant use case.

## Supervised learning

One of the most common approaches in machine learning is supervised learning, which is based on previously labeled data.

### Data Labeling

Data labeling is a fundamental process, although it is often time-consuming since, in many cases, it is performed manually. It consists of assigning meaning to the data so that models can learn patterns and make decisions.

Following the restaurant example, a common form of labeling occurs when users assign a rating from 1 to 5 stars, indicating how good they consider a restaurant to be. These labels allow models to be trained to recommend places based on the experiences of other users.

![data labeling](/assets/posts/2026-01-14/learning_supervised.png "data labeling")

## Unsupervised learning

However, it is not always possible to label data directly. In such cases, unsupervised learning techniques are used.

### Clustering

Clustering is a technique that allows data to be grouped based on shared attributes, without the need for prior labels. This helps uncover hidden patterns and create new decision-making models.

In the context of restaurants, one of the most important attributes is geolocation. Based on the user’s location and the locations of businesses, it is possible to group nearby restaurants using distance and geographic proximity techniques.

![clustering](/assets/posts/2026-01-14/unsupervised.png "clustering")

## Key Geolocation Concepts

To better understand these processes, it is important to be familiar with some fundamental geolocation concepts:

* **GeoJSON:** A standard JSON-based format used to represent geographic data, such as points, lines, and polygons.
* **Geographic Point:** Represents a specific location on the map, defined by two values: latitude and longitude.
* **Polygon:** A set of three or more geographic points that form a closed area, starting and ending at the same point.

### Geographic Metrics Used in Models
When working with latitude and longitude, it is possible to compute metrics that are very useful for recommendation models:

* **Haversine Distance:** Calculates the linear distance between two geographic points on the Earth’s surface.
$$
d = 2 \cdot 6371 \,\text{km} \cdot 
\arcsin\left(
\sqrt{
\sin^2\left(\frac{\Delta lat}{2}\right) +
\cos(lat_1)\cos(lat_2)\sin^2\left(\frac{\Delta lon}{2}\right)
}
\right)
$$

$$
\Delta lat = lat_2 - lat_1
$$

$$
\Delta lon = lon_2 - lon_1
$$


* **Bearing:** Calculates the angle or direction between two geographic points relative to the north axis, helping determine the relative orientation between locations.
$$
\theta = \operatorname{atan2}\left(
\sin(\Delta lon)\cdot \cos(lat_2),
\cos(lat_1)\cdot \sin(lat_2) -
\sin(lat_1)\cdot \cos(lat_2)\cdot \cos(\Delta lon)
\right)
$$

$$
\Delta lon = lon_2 - lon_1
$$

$$
\text{Bearing} =
\left(
\theta \cdot \frac{180}{\pi} + 360
\right) \bmod 360
$$

## Case: Customer Recommendation for Sellers in a Marketplace Platform
While working on a marketplace platform, I was involved in the design of a feature focused on recommending customers to sellers based on geographic proximity. The platform supported multiple entrepreneurs by centralizing products and customer information, creating a unified ecosystem that enabled data-driven decision making.

One of the key challenges was to help sellers efficiently identify which customers to visit, prioritizing those located nearby. To address this, the platform leveraged location-based data to recommend the most relevant customers according to the seller’s current position, improving operational efficiency and customer coverage.

### Data Resources and Location Strategy

Before login access to the platform, customers were required to complete an onboarding process. This step was essential for collecting general information such as name, phone number, and address. The registered address was then transformed into geographic coordinates (latitude and longitude) through a geocoding process, enabling spatial analysis within the system.

Stores within the platform were treated as static entities, which simplified their association with predefined geographic zones. Each entrepreneur operated one or more distribution warehouses within these zones.

From a data perspective, this setup enabled a supervised learning labeling flow, where the system assigned a warehouse label to each customer based on their geocoded location. The predefined relationship between geographic zones and warehouses acted as labeled training data, allowing the system to consistently classify customers and determine the warehouse they should be served from.

This labeling process ensured alignment between customer recommendations, logistical operations, and seller workflows, providing sellers with actionable, location-aware insights when prioritizing which customers to visit.

### Onboarding Data Flow and Data Model
The following section presents the data flow derived from the onboarding process, as well as the entity–relationship diagram that supports the customer recommendation logic for sellers.

![Onboarding flow](/assets/posts/2026-01-14/onboarding_flow.png "Onboarding flow")

![ER Diagram](/assets/posts/2026-01-14/er_diagram.png "ER Diagram")

### Data Pipeline for Feature Engineering and Customer Segmentation
Before building the customer recommendation algorithm, it was necessary to enrich the existing data with additional attributes. To achieve this, a data pipeline was designed and implemented:

* For each customer assigned to a specific warehouse, the system retrieved the latitude and longitude derived from the customer’s address. Using these coordinates, an integer bearing was calculated relative to the geolocation of the assigned warehouse.
* A reference angle was then defined to segment the 360-degree circumference into directional clusters. Selecting this parameter was critical:
  * A reference angle that is too large would result in high intra-cluster dispersion, reducing the precision of recommendations.
  * Conversely, a reference angle that is too small would generate many clusters with insufficient or no customers, limiting the system’s ability to produce meaningful recommendations.
* Each customer was then assigned to a directional cluster based on the reference angle, using an inclusive lower bound and an exclusive upper bound. For example, with a reference angle of 5 degrees, the system would generate 72 directional clusters, following the pattern: 0–5°, 5–10°, 10–15°, and so on.
* Once constructed, each cluster was materialized and persisted in a file-based storage system, such as Google Cloud Storage buckets. To keep storage lightweight and decoupled from sensitive or frequently changing attributes, the stored files contained only customer indices (or unique identifiers) rather than full customer records. This design allowed downstream systems to efficiently resolve customer details when needed while minimizing storage and recomputation costs.
  
The objective of this pipeline was to enable customer grouping using direction based metrics, applying principles of unsupervised learning to segment customers into meaningful clusters.

Additionally, it was essential for this pipeline to run according to the data update frequency. Given the high volume of new customer registrations, the pipeline was designed to execute periodically (e.g., on a weekly basis) to ensure that recommendations remained accurate and up to date.

### Customer Recommendation Algorithm

Finally, a dedicated service was built to host the customer recommendation algorithm. The service took the seller’s current latitude and longitude as input and returned the three closest customers, following the workflow described below:

1. Using the seller’s latitude and longitude, the system computed the corresponding integer reference bearing.
2. Based on this bearing, the service queried the file-based storage system to locate the most relevant directional cluster.
3. The cluster data was then deserialized, retrieving the list of customer indices associated with that segment.
4. Using these indices, the service queried the primary database to fetch the customers’ latitude and longitude coordinates.
5. For each selected customer, the system calculated the geodesic distance between the seller and the customer.
5. Each customer was then inserted into a min-heap (priority queue), where the priority key was the computed distance.
6. Finally, the algorithm extracted the top three customers with the smallest distance values from the heap and returned them as the recommendation result.

This approach allowed the system to efficiently narrow down the search space using directional clustering, while still ensuring precise distance-based ranking at query time.


## resources

* [My customer Recommendation Algorithm](https://github.com/djob195/brainyBits/tree/master/cases/customer-recommendation)