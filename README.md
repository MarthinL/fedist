# FEderated DISTribution
A federated approach to process and data distribution


When a problem overwhelms us we [divide and conquer](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm) it's complexities to express the solution as work for a computer. When the volume of work we palm off to a computer overwhelms it's capacity we employ a simmilar set of tools to spread the workload over a processor's multiple cores ([concurrency](https://en.wikipedia.org/wiki/Parallel_computing)) or several networked computers ([distribution](https://en.wikipedia.org/wiki/Distributed_computing)).


```mermaid
flowchart TB
    INITIALISE["Divide into n pieces\ni=1"]
    MORE{"i<=n ?"}
    ACT["Process piece i\n---\ni = i + 1"]
    FINALISE("Combine results")
    INITIALISE --> MORE
    MORE --->|Yes|ACT
    ACT --> MORE
    MORE ---->|No|FINALISE
```
## Figure 1: Pseudo Algorithm
```mermaid
flowchart TB
    INITIALISE["Divide into n pieces"]
    ACT1["Process piece 1"]
    ACT2["Process piece 2"]
    ACT3["Process piece 3"]
    ACT4["Process piece 4"]
    ACT5["Process piece 5"]
    ACT6["Process piece 6"]
    subgraph ETC[" "]
        direction LR
        ETC1((" "))
        ETC2((" "))
        ETC3((" "))
    end
    ACTN1["Process piece n-1"]
    ACTN2["Process piece n"]
    FINALISE["Combine results"]
    INITIALISE --> ACT1
    ACT1 --> ACT2
    ACT2 --> ACT3
    ACT3 --> ACT4
    ACT4 --> ACT5
    ACT5 --> ACT6
    ACT6 --> ETC1
    ETC1 --> ETC2
    ETC2 --> ETC3
    ETC3 --> ACTN1
    ACTN1 --> ACTN2
    ACTN2 --> FINALISE
```
## Figure 2: Single Tasking
```mermaid
flowchart TB
    INITIALISE["Divide into n pieces"]
    ACT1["Process piece 1"]
    ACT2["Process piece 2"]
    ACT3["Process piece 3"]
    ACT4["Process piece 4"]
    ACT5["Process piece 5"]
    ACT6["Process piece 6"]
    subgraph ETC[" "]
        direction LR
        ETC1((" "))
        ETC2((" "))
        ETC3((" "))
    end
    ACTN1["Process piece n-1"]
    ACTN2["Process piece n"]
    FINALISE["Combine results"]
    INITIALISE --> ACT1 & ACT2 & ACT3 & ACT4 & ACT5 & ACT6 & ETC1 & ETC2 & ETC3 & ACTN1 & ACTN2
    ACT1 & ACT2 & ACT3 & ACT4 & ACT5 & ACT6 & ETC1 & ETC2 & ETC3 & ACTN1 & ACTN2 --> FINALISE
```
## Figure 3: Multitasking (Ideal)
```mermaid
flowchart TB
    INITIALISE["Divide into n pieces"]
    ACT1["Process piece 1"]
    ACT2["Process piece 2"]
    ACT3["Process piece 3"]
    ACT4["Process piece 4"]
    ACT5["Process piece 5"]
    ACT6["Process piece 6"]
    subgraph ETC[" "]
        direction LR
        ETC1((" "))
        ETC2((" "))
        ETC3((" "))
        ETC4((" "))
        ETC5((" "))
        ETC6((" "))
        ETC1-->ETC2
        ETC3-->ETC4
        ETC5-->ETC6
    end
    ACTN1["Process piece n-1"]
    ACTN2["Process piece n"]
    FINALISE["Combine results"]
    INITIALISE --> ACT1 & ACT3 & ACT5 & ETC1 & ETC3 & ETC5 & ACTN1
    ACT1-->ACT2
    ACT3-->ACT4
    ACT5-->ACT6
    ACTN1-->ACTN2
    ACT2 & ACT4 & ACT6 & ETC2 & ETC4 & ETC6 & ACTN2 --> FINALISE
```
## Figure 4: Multitasking (Constrained)
