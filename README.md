# Mini Metro Optimization

The goal of this program is to find the optimal design of public transport. 

Properties  of optimal solution:
* passengers are transported to their destination as fast as possible
* the design results in as few congestions as possible

What is available to the metro designer explicitly:
* location of stations and their distances between each other
* each station is marked on map by a symbol
* each station has only one symbol
* number of passengers waiting at a station and their individual destinations (denoted by a list station symbols)
* number of metro lines
* number of trains
* number of cars
* number of interchanges
* number of tunnels

What the metro designer may be able to infer:
* how fast trains travel
* how fast new passengers arrive to their initial stations
* distribution of stations by their symbol (squares, triangles and circles are much more common than stars, crosses, droplets)
* distribution of stations on map by their symbol (may be completely random)
* distribution of passengers across station(may be completely random)
* distribution of passenger destinations at individual stations (may be completely random)

Should be noted:
* passengers appear intelligent - they are capable of switching lines (if those lines intersect) to get to destinations that are not situated on the line(s) that their origin stations have available. it is not known whether the passengers are able to decide their path based on its travel time or whether they choose first available train that will shorten distance between the passenger and their destination. it is also not known if passengers can get lost and travel forever.

## Project plan

Before trying to solve the game with all its nuances I will first attempt to intuitively solve parts of the problem that lead to the most robust solution. At the beginning I will try to only minimize total distance of one line. Then I will try to minimize sum of distances of multiple lines. This adds the issue of passengers not being able to reach a destination that lies on a different line that doesn't intersect with the line they're currently on. I will add this new constraint to my algorithm.

After that I will attempt to minimize the distance between any two types of stations. This is because there are usually multiple stations denoted by the same symbol and passengers don't care at which specific station they finish their journey - as long as the station has passenger's symbol.

Finally I will introduce optimization of train and train car counts on each line, their timing and direction - to minimize the average time it takes to get from one station to another.

I will not consider tunnels and interchanges at this point.

There are certain times when the game allows users to choose one new addition to their inventory from given options. They can be tunnels, new lines (up to the maximum line limit), trains and train cars. The algorithm should be able to tell the user which of the items will maximize their metro plan efficiency better.

At this point I will have an algorithm that produces working solution leading to high scores in the endless mode of the game where the passengers don't overcrowd the stations (which in normal mode would cause the game to terminate) and where it is possible to redesign the metro plan at any point from the bottom up.

It would seem that the only way how I could refine solutions for the normal game mode would be to test them on accurate simulations of the game. The only problem is that I don't have access to the game source code so I can't know some of the very important details of the game. There's no way of telling in what order the new stations show up on the map, with which symbols, in what intervals. And there's no way of telling what is the function that distributes new passengers across their stations of origin.

However, a working accurate simulation could not only help find better solutions, it could also do so by different methods e.g. (deep) reinforcement learning. I have yet to implement such simulation.


### Stage 1: 
* only 1 metro line is considered
* only 1 train (with its 1 car) is considered
* metro can be redesigned any time additional stations are added
* travelling salesman problem - can be optimized with simulated stochastic annealing
* visualization of single-line solution

### Stage 2: 
* more than 1 metro line are considered
* only 1 train (with its 1 car) per metro line is considered
* metro can be redesigned any time additional stations are added
* visualization of multi-line solution

### Stage 3: 
* more than 1 metro line are considered
* more than 1 train per metro line are considered
* additional cars for trains are considered
* metro can be redesigned any time additional stations are added

### Stage 4:
* tell user which item they should choose from the set of items when given the option

## Implementation questions

### How to represent solutions?
* a list representing in what order stations are connected
* what about metro lines that do not form a cycle? significant design decision that will influence the impact of multiple trains on one line and their direction
* a more general design is path generation without enforced cycles but with a chance of cycles occuring anyway
* different measures of single line distance: length of the whole line OR sum of distance between every station and every other station = average distance from any point of origin to every destination OR take into account train direction too
* metro game doesn't permit lines to visit the same station twice = simplification of what the representation must fulfill to be accurate - so it is strictly either a LINE or a CYCLE - nothing else or inbetween
* so the representation may or may not include a cycle breaker - a symbol in line that represents that the two lines stations are not line neighbours
* there may be at most ONE cycle breaker
* thinking whether there is a simpler representation that need not contain cycle breakers
* do there even need to be cycle breakers? are there any guarantees that LINEs or CYCLEs necesarily perform better than the other (for all cases or only some cases)?
* a cycle with two trains going in opposite directions should on average have the best performance
* can't think of a single data structure that could be manipulated with elemental operations that would not need to explicitly add or remove line breakers
* how is neighbourhood of a solution defined - if I want to use a steepest descent search algorithm?
* we can actually define it however we want, the only question is what breadth of neighbourhood do we find optimal?
* without cycle there are two directions the train goes during runtime, changing between runs - with cycle there is one and only direction
* meaning that with cycles it is necessary to make a decision which way the train should go - this could influence performance of only one line but also performance of the whole system - the two different performances are not guaranteed to change together with changes to one line
* I decided to represent a single-line solution as a list of stations representing a cycle and an index (if specified) indicating between which two stations there isn't a line connection (thus breaking the cycle)

### How to compare performance of cyclic and non-cyclic lines?
* cyclic solutions move only in one direction
* non-cyclic solutions move in each direction half of the time
* simulate average performance in both directions in both cases
* while performance of non-cyclic solution will be evaluated twice - once in each direction - the real performance will be an average of the two
* performance of cyclic solution will be evaluated twice - once in each direction - the real performance will be the better of the two
* this applies ONLY TO SINGLE-LINE SOLUTIONS

### What search strategies to use?
* evaluate all permutations of stations to find the best solution
* if I design structure of search space I can use steepest ascent/descent heuristic
* stochastic search algorithm such as simulated annealing

### How to compare performance of different strategies?
* exhaustive: measure how many solution evaluations it takes to find the best solution
* non-exhaustive: measure number of evaluations it takes to find solution that is close enough to the exhaustive best solution
* meta-optimization: trying to find the optimal maximal number of evaluations required to find optimal solution

### How to deal with growing complexity of search heuristics?
* exhaustive evaluation of all solutions will always be a problem
* with growing number of lines my design of neighbourhood relationship creates very large neighbourhoods - all neighbours must be evaluated for steepest descent search
* on the other hand large neighbourhoods provide much finer search of optimal paths of descent
* at some point the performance tax of steepest descent will be too high
* my plan is to create another implementation of my algorithms in a compiled, statically typed language - C, C++, Java or Rust

## Progress
- [x] representation of a single-line solution
- [x] one_solution class for solutions with only one line
- [x] gameplan class for representing the problem
- [x] draw gameplan
- [x] draw solution
- [x] fitness measure using only line length
- [x] exhaustive search for single-line solution
- [x] steepest descent search for single-line solution
- so far simulations are showing that steepest descent yields solutions equally good as exhaustive search - no need for simulated annealing? May be a good idea to implement it anyway in case greedy algorithm starts getting stuck in local minima at more complex problems (or different fitness function)
- [x] simulated annealing for single-line solution
- slightly faster than steepest descent but solutions can sometimes be slightly less optimal than the best possible solutions
- [x] STAGE 1 COMPLETE
- [x] fitness measure using reachability of station from other stations
- [x] representation of n-line solution
- [x] solution class for solutions with any number of lines
- [x] draw solution with multiple lines
- [ ] exhaustive search for n-line solution - only total length considered
- [ ] steepest descent for n-line solution - only total length considered
- [x] simulated annealing for n-line solution - only total length considered
- [x] simulated annealing for n-line solution - total length AND solution validity considered
- [ ] exhaustive search for n-line solution - fitness of reachibility
- [ ] steepest descent for n-line solution - fitness of reachibility
- [ ] simulated annealing for n-line solution - fitness of reachibility
