---
description: Introduction
---

# 🆕 DOD Introduction

### What is DOD?

DOD stands for DePHY Official Delegation, which delegates (stakes) officially held stPHY tokens to community nodes based on their contribution. Created to maximize the decentralization, reliability, and performance of the DePHY network, DOD expands the number of nodes with diverse pledge sources while maintaining a large and representative mainnet-beta network. The program incentivizes community participation to ensure network robustness and security, laying the groundwork for a sustainable DePHY ecosystem.

### What are the benefits of DOD?

#### Increased decentralization of the network

After launching the staking campaign, we observed that a small number of nodes gained significantly more than ordinary nodes through high stakes. This issue, reported by the community, caused the network to favor a very small number of nodes, reducing decentralization. Consequently, a minority of nodes earned higher mining revenue while most community members earned less.

We're now taking action to balance the amount and weight of staked tokens in the network, allowing well-performing nodes to receive free pledges and increase decentralization.

#### Balancing node gains

Once DOD launches, the system's Gini coefficient will be greatly reduced, effectively mitigating the ultra-high gains of single nodes. Low-staking nodes can receive official stake delegation for free, dramatically increasing the earnings of individual participants who only need to contribute to the network.

#### Priority participation in testing services

We will soon launch the first round of testing for additional services that provide decentralized data transmission for third parties, including AI, DePIN, and RWA projects. Nodes participating in DOD will receive priority access to these testing services and additional revenue opportunities.

### How to participate in DOD?

#### Basic Participation Conditions

| Standard Description   | Standard Requirements            | Comments                                                                                                |
| ---------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Last 120h online hours | >90% of the time                 |                                                                                                         |
| Running Node Version   | See latest official requirements |                                                                                                         |
| Number of self-stake   | >5000                            | Staking other than DOD and other third party delegation is considered as self-stake, in terms of stPHY. |
| Blacklisted            | Not blacklisted                  | Malicious nodes will be blacklisted                                                                     |

#### Technical Indicators

| Standard Description                 | Comments                                    | Abbreviation |
| ------------------------------------ | ------------------------------------------- | ------------ |
| Last 120h online hours               | Hourly                                      | UP120        |
| Last 2160h online                    | Hourly                                      | UP2160       |
| Number of self-collateralized stPHYs | 7-day average number of self-pledged stPHYs | SS           |
| Number of total encumbrances         | 7-day average total stPHY                   | TS           |

#### Scoring Method

Score is calculated based on a combination of the following metrics that determine a node's ranking and allocation in the DOD program:

$$
Score = UP120(\%) \times 0.4 + UP2160(\%) \times 0.3 + \frac{SS}{TS} \times 0.3
$$

Among them:

* _**UP120**_ is weighted at 40% to ensure recent stability of the node
* _**UP2160**_ is weighted at 30% to ensure long-term stability of the node
* Self-stake ratio (_**SS**_/_**TS**_) is weighted at 30% to encourage node operators to stake in themselves.

This scoring considers the short-term and long-term stability of the node, as well as the percentage of the operator's own investment, rather than relying solely on pledge numbers. This ensures network decentralization and overall health while preventing overweighting of nodes with large stakes.

As an example:

Node A has participated in the network for 2160 hours and has only 5000 self-stake, then

Score = 1×0.4 + 1×0.3 + 5000/5000×0.3 = 1.0

After node A gets the official 20000 DOD delegation stake, then

Score = 1×0.4 + 1×0.3 + 5000/25000×0.3 = 0.76

Node B has only been running for 500 hours in total, 495 hours of active time, and has 50000 self-stake.

Score = 495/500×0.4 + 495/2160×0.3 + 50000/50000×0.3 = 0.76475

After node B gets the official 20000 DOD delegation stake, then

Score = 495/500×0.4 + 495/2160×0.3 + 50000/70000×0.3 = 0.6790357143

In this example, we can see that a well-participated node will rank higher than nodes with high stakes but short uptime.

#### Sorting method

All nodes are sorted in descending order according to their Score, and the top 5000 nodes will receive DOD delegation.

#### Calculation Period

Calculated every 7 days (168 hours), with minor time variations possible.

#### Delegation Cancellation Criteria

Criteria Cancellation:

After all nodes are sorted by Score in descending order, nodes ranked within the top 7000 will continue to receive DOD delegation. If a node's ranking falls below 7000, its delegation will be canceled.

Immediate Cancellation:

1. Nodes that have been blacklisted will have their pledges canceled immediately.
2. Pledges will be canceled immediately if a node's accumulated offline time exceeds 24 hours.
3. Other behaviors that are detrimental to the network's operation

Nodes affected by Criteria Cancellation can still participate in the next round of ranking.

Nodes subject to Immediate Cancellation from DOD, except those blacklisted, can still participate in the next round of ranking.

### DOD Vault Quantity

Currently, the initial vault quantity is 200 million or 200,000,000 stPHY, which may be adjusted later.
