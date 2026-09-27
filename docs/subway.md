# Subway Challenge: Beijing Edition

Beijing Subway is currently (late 2026) the longest subway system in the world, with 909 kilometers (565 miles) of tracks in total. You can get on line 5, take an hour long nap, and wake up on the other end of the city, still many stops away from the terminus. I love its scale and reach, so I decided to embark on a project to try to visit every single one of its stations in one day.

## Rules

First, the rules of the challenge. It turns out, I am not the first person to try to speedrun a metro system! [Subway Challenge](https://en.wikipedia.org/wiki/Subway_Challenge) (Class B in particular) is an 80 years old challenge where participants speedrun the entire New York subway system (MTA). I will be borrowing heavily from their rules.

I am allowed to start and end a run at any metro station, and I do *not* have to start and end at the same metro station. During a run, I am allowed to board / unboard trains and transfer between lines as much as I want. However, I am not allowed to exit and reenter the metro system to take a shortcut by car / bike / rocket (Thankfully all line transfers can be done without leaving the station). In other words, the run must be done with a single one-way ticket. 

![shortcut not allowed](assets/one_ticket.jpg "Unless the train itself takes a shortcut")

I will also count a station as visited if the train I am on stops at it. Borrowing from the MTA rules, if I take a [skip-stop](https://en.wikipedia.org/wiki/Skip-stop) train that flies through a station without stopping, that does not count as visiting it. On the flip side, I do not have to physically disembark and step in a station to visit it. This is so that I do not lose my hard-fought seat on a rush hour train, or worse, not be able to squeeze back onto the packed train at all.

Unfortunately for me (and fortunately for the subway workers), the Beijing Subway is not open 24 hours like the MTA. On most lines, the first trains depart from the termini at around 5:30 AM, and the last trains depart at around 11:30 PM. Once the last train has departed from a station, it will be cleared and closed for the night. This means that excluding extreme strategies like hiding in the bathroom, I will have a hard time limit of approximately 18 hours (the last train *leaves* at 11:30 PM, so stations on longer lines may not close for another hour).

![Hiding in a bathroom stall](assets/hiding_in_bathroom.jpeg "Beijing Metro Speedrun Bathroom%")

What this means is that visiting ALL 541 stations of the Beijing Subway in one run is almost certainly impossible. With significantly more length and more stations than the MTA, visiting every station of the Beijing Subway would almost certainly take more time than visiting every station of MTA, and the current record of the MTA Subway Challenge is [just over 24 hours](https://en.wikipedia.org/wiki/Subway_Challenge#472_stations), held by Kate Jones, already far over our 18 hour limit.

From another perspective, lets suppose we are to visit all 541 stations, and we will suppose we have 19 hours to do it. Then on average, we must visit one new station every 2.11 minutes. In comparison, line 10, a downtown line with stations relatively close to each other, takes around 105 minutes to hit all 45 stations, with an average of 2.33 minutes per station. Therefore, even in the hypothetical scenario where line 10 engulfs the entire Beijing Subway, and we do not need to ever backtrack or transfer, we still would not have enough time to visit all stations.

![line 10 subsuming the metro system](assets/line_10_expansion.jpeg "One ring to transport them all")

Therefore, we will change our objective slightly. Instead of visiting all stations in the shortest time possible, our goal would be to visit as many different stations as possible in a single run - a single day. From an optimization perspective, we are attempting the *dual* of the original challenge.

## The Plan

Here is the high level plan. We first need a sufficiently accurate mathematical model of the Beijing Subway, so that we can construct and verify routes through the system. To construct such a model, we first need to gather:

1. A timetable of when every train arrives at every station
2. A table of distance / time between platforms at every single transfer station (at every station in reality, since we sometimes need to disembark and go the other direction)


Then, we will use standard optimization algorithms to solve for a directed path through the graph that visits the highest number of different stations, taking into account additional factors such as transfer time uncertainties. Finally, we will verify the route in simulation, and give it a go in real life if it passes the test.

## Modeling the Metro

With these data gathered, we will construct a [graph](https://en.wikipedia.org/wiki/Graph) to represent the Beijing Subway, and to transform our problem into a graph theoretical problem. 

Here, we actually have choices as to how complex we want our model to be - how many real life variables we want to keep vs. how many we want to abstract away. At the bottom of the complexity ladder, we have model A [find a more descriptive pair of names of these two models], which represents the Beijing Subway as a simple undirected graph, where:

- Each vertex represents a station A.
- If station A and B are adjacent on the same line, then there is an edge between vertex A and B.
- The weight of edge AB is the travel time between station A and B, calculated from the timetable.

With this model, we assume that transferring at a transfer station - both the walking part and the waiting part - takes no time. This is not true, but not *too far* from reality either, given the high service frequency of Beijing Subway. In theory, the longest we have to wait for a train is [insert length of time, station, and time of day here, should be about 10 minutes], and the longest we have to walk is [insert length of time, station, distance here, should be around 9 minutes]. Back of envelope math says that if we make ten transfers during a run, each taking 5 minutes (including wait time), this would give us a total error of 50 minutes or around 4.6% of total run time - not too bad. Finally, we also assume that travel time between two stations is constant and equal in both directions, which is mostly true [check this one!].

[Insert histogram of transfer time and time between two consecutive trains here]

From another perspective, this is an optimistic model, since it almost always underestimates the time it takes to get from one station to another by ignoring transfer times (unless the train beats the schedule *significantly* in real life). This also means that the route that it produces serves as an upper-bound, score wise, to what could be achieved in practice.

At the top of the complexity ladder, on the other hand, we have model B, which uses a directed graph to represent the metro system:

- For each train X that stops at a specific station Y, we have a vertex represented by the pair (train X, station Y). If we are at this vertex on the graph, it means that we are aboard train X while it stops at station Y - either boarding the train there, disembarking from the train there, or staying on the train while it goes through there.
- There is an edge from vertex U to vertex V if we can travel from U to V by either staying on train and riding for one stop, or by transferring to another line on foot. This means that:
    - If train X stops at Y, and its not the final stop, then is an edge from (X, Y) to (X, Next Station from Y in X's direction). Its weight would be the travel time of the train, as calculated from the timetable.
    - If Y is a transfer station, and train X and X' stop there, then there is an edge from (X, Y) to (X', Y), if there is enough time between their arrival to transfer from X to X'. Its weight would be the time between the arrival of X and X'.

Thus, we have explicitly taken into account both the walking and waiting time of transfers. In theory, paths through this graph should translate exactly to real life routes, assuming the datasets are accurate. However, this comes at the cost of expanding each station into 300+ vertices, increasing our total vertex count by two orders of magnitude, and the edge count similarly. We can prune the graph with some heuristics, but it will certainly remain more complex and expensive than model A. Further, any error in the transfer time data would poison this model more than model A.




