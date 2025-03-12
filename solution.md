
## Solution 

### Approach & Analysis

I looked at the plotted distribution of the queries, and noticed that the vast majority of queries fall below 10, and there are very little queries above 30. Queries below 5 made up more than half of the total queries. This falls in line with an exponential distribution. With this information, I decided to focus on fast search of these lower query values while largely ignoring extremely large query values. Since the probability of each query decreases exponentially as the value increases, I decided to split the queries I would search into two sections, with different strategies for each. 

### Optimization Strategy

[Explain your optimization strategy in detail]
I focused on ensuring that the most likely query values are all connected. Although the parameter for the exponential distribution that queries are sampled from is given in the constants file, for robustness, I calculated the maximum likelihood estimate (MLE) of the lambda parameter. This ensures that given queries sampled from a different distribution, my graph could still function well. This can be solved for by setting the derivative of the log-likelihood to 0 and solving in terms of the samples. This yields that the MLE for an exponential distribution is the inverse of the sample mean. I utilized the query targets given in results to calculate the MLE. 

Using this estimate, I was able to calculate thresholds for where approximately a certain percentage of the data would lie. I split the data into two sections: one large threshold that aims to capture a very large percentage of the data and a smaller threshold that calculates the most likely query values. The values in the smaller threshold are searched more rigourously while the larger threshold is searched more stochastically since it encompasses a larger range. 

Any query above the large threshold is redirected towards nodes under the large threshold. This aims to reduce the size of the path needed to reach query targets that are more likely.

### Implementation Details

[Describe the key aspects of your implementation]

I defined a function to first calculate an estimate of the lambda parameter using result data. This calculation yielded an estimate of 0.11, which is very close to the true parameter of 0.1. Using this parameter, I used the CDF of an exponential distribution with respect to the lambda estimate to find thresholds for where 0.9 of the queries would be under and for where 0.99 of the queries would be under. These were denoted as the lower and upper thresholds respectively.

For data under the lower threshold, I connected them linearly with one outgoing edge each s.t. 0 -> 1 -> 2 ... -> lower_threshold. This ensures that any query in this range can be found as long as a node that is less than the query in value is reached. This search may also extend for longer since each node only has one outgoing node, so the lower threshold ensures that only the most likely nodes are searched in this manner. It also ensures that we can search all values in this range once the 0 node is reached, which since the majority of the data falls in this range (90% approximately), searching all values is extremely valuable.

For data between the lower and upper threshold, nodes are connected similarly to those in the lower threshold. Every n steps where n > 1, there is an edge added to 0 such that the lower range can be searched with some probabiliy. In this implementation, n=5. For every m steps where m > 1, there is an edge added to the query target that is two less so that the reverse direction could be searched (since this range is fairly large).

For nodes above the upper threshold, many were added such that there were two edges, one connecting to the lower range and one connecting to the range between the lower and upper thresholds, with a greater weight placed on the edge to the lower range since it is more probable that the query is in that range and I wanted to have a high probability of at least one walker reaching the lower range. Nodes above a certain range only connected to the lower threshold in order to ensure that the graph had less than 1000 edges.

### Results

[Share the performance metrics of your solution]

My solution achieved 99% success, which is a 19% improvement. It also achieved an average path length of 8, which is a 98.7% improvement. The score was calculated to be 527.31. 

### Trade-offs & Limitations

[Discuss any trade-offs or limitations of your approach]

A trade-off is that since the nodes above the upper threshold are not connected, a query extending to those values will be guaranteed to be unsucessful. Another disadvantage is that my graph contains cycles in the range between the lower and upper threshold, so a random walk could get trapped in a cycle, which has the potential to dramatically increase path length. Edge weights were also finetuned very little, so they are likely to be suboptimal.

### Iteration Journey

[Briefly describe your iteration process - what approaches you tried, what you learned, and how your solution evolved]

I began by writing a scaffold of my current graph, where arbitrary thresholds were chosen based off intuition from looking at plots of query data. I played around with different strategies, including adding much denser connections between the lower range and the upper-lower range. In the early stages, I also had a lot more random edges, where a random next edge would be sampled from a range. 

I refined this version such that edges were no longer randomly sampled. I realized that the most important node to be able to reach is node 0, and all nodes in the lower range would be guaranteed to be found as long as node 0 was reached with the sequential setup. Therefore, for larger nodes I hardcoded it such that there was a connection to node 0 with at least high probability. I also added edges connecting to the lower threshold to search that range as well. This also made my results more consistent due to the reduced randomness. 

I also made the thresholds less arbitrary. Since I wanted my graph to be robust to different exponential distribution parameters, I decided to find the MLE and use it to calculate thresholds. Only values below the 99% threshold (which was found to be 41) were connected to the graph, which may be the cause for the 99% accuracy rate. Only values below the 90% threshold (which was found to be 20) were searched sequentially, which provided higher guarantees of finding the node at the cost of potentially longer path lengths. Since in the exponential distribution, the data heavily leans left, the 90% threshold was about half of the 99% threshold, helping make the graph less dense.

I also did some minor finetuning of the edge weights by hand. Given more time, I would use a more rigorous finetuning scheme, perhaps using binary search, to find more optimal edge weights. From experimenting with weights, I learned that they have a large effect on the final results, so more finetuning would be very valuable. In this implementation, I resorted to using the pdf of the exponential distribution as edge weights, which weighted more towards lower value queries, which were more probable. This fitted my goal of having walkers have an increased chance of visiting more likely queries, but the exponential distribution changes drastically so I added a constant to stabilize it (i.e. I didn't want exponential_pdf(0) to result in an almost guaranteed chance of traveling to 0). 

---

### Note on time:
I spent about 20 minutes removing Git LFS from this repo since I didn't realize LFS didn't work for public forks. 

* Be concise but thorough - aim for 500-1000 words total
* Include specific data and metrics where relevant
* Explain your reasoning, not just what you did
