2025/05/27  -  17:47
status: 

Tags: [[AI]] [[Federated learning]] [[machine learning]]

1. A shared model is sent from the server to some selected clients. 
2. The involved clients further train the model with their local data and submit the updated model or parameters to the server, respectively. 
3. The updates are collected by the server for aggregation, and the server will broadcast the aggregated model to all clients. 
4. The previous three steps will be repeated until we converge to a global model or reach a given number of rounds decided beforehand.

The key innovation of federated learning is that the server only requires parameters instead of collecting raw data from the clients.

# References
