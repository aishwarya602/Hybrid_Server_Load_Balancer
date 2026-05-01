# Hybrid Load Balancer Project
This project implements a Hybrid Load Balancer that combines multiple scheduling strategies to optimize request distribution across backend servers. It utilizes traditional algorithms like Round Robin and Least Connections alongside Reinforcement Learning (RL) to adaptively improve performance based on feedback.  
## Features
- Hybrid Selection Logic: Integrates three distinct strategies to pick the best server.
    a. Weighted Round Robin (WRR): Rotates through servers based on assigned weights.
    b. Least Connections (LC): Prioritizes servers with the fewest active connections.
    c. Reinforcement Learning (RL): Uses an Epsilon-Greedy policy and Q-learning to exploit high-performing servers and explore others.
- Persistent Storage: Implementation available using SQLite to manage server metadata (weights, Q-values, connection counts) and log request history.
- Dynamic Server Management: Support for adding, removing, and monitoring server status.
- Asynchronous Logging: Ability to log rewards and request data without blocking the main load-balancing thread.
## System Architecture
The system consists of three main components:
1. Backend Servers: Simulated instances that handle incoming requests with variable processing times.
2. Hybrid Load Balancer: The core module that evaluates all three strategies (WRR, LC, RL) and selects a server by minimizing the Q-value among the candidates.
3. Feedback Loop: After a request is handled, a "reward" (e.g., based on response time) is fed back to the load balancer to update the server's Q-value in the database.
## Installation & Setup
1. Dependencies: 
The project primarily uses standard Python libraries. No external libraries required for core functionality. Optional: sqlite3 (included in Python), numpy
2. Database Initialization: The SQLite version automatically creates load_balancer.db with necessary tables for servers and request_logs upon instantiation.
## Core Implementation Details
### Hybrid Selection Algorithm
The get_server method determines candidates from each strategy and selects the final server based on the RL Q-table:  Pythondef get_server(self):
    
    # 1. Weighted Round Robin candidate
    rr_server = self.get_wrr_candidate()
    
    # 2. Least Connections candidate
    lc_server = self.get_lc_candidate()
    
    # 3. RL (Epsilon-Greedy) candidate
    rl_server = self.get_rl_candidate()
    
    # Final Selection: Minimize Q-value among strategy winners
    selected_server = min([rr_server, lc_server, rl_server], key=lambda s: self.q_table[s])
    return selected_server
### Reinforcement Learning Update
Q-values are updated using the standard Bellman equation approach to refine server selection over time:  
- Alpha ($\alpha$): Learning rate.
- Gamma ($\gamma$): Discount factor.
- Reward: A value (typically between -1 and 1) representing the efficiency of the handling server.
## Usage Example
Python

    from hybrid_lb import HybridLoadBalancer
    
    # Initialize with servers and their weights
    servers = ["Server1", "Server2", "Server3"]
    weights = {"Server1": 3, "Server2": 1, "Server3": 2}
    lb = HybridLoadBalancer(servers, weights)
    
    # Distribute a request
    assigned = lb.get_server()
    print(f"Request handled by: {assigned}")
    
    # Provide feedback (e.g., fast response = positive reward)
    lb.update_q_values(assigned, reward=0.8)

## Monitoring & Analysis
The project includes capabilities to plot performance metrics, such as:
- Server Comparison: Visualizing requests handled by each server.
- Reward Distribution: Tracking the distribution of feedback values over time.
- Rolling Average Reward: Monitoring the learning progress of the RL agent.
