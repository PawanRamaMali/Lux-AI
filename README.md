# Lux-AI 
Kaggle Lux AI - Gather the most resources and survive the night!

This is not the official representation of the Lux AI Challenge. 

The purpose of this repository is to discuss open source algorithms which can be implemented to solve the challenge.

## Introduction

The night is dark and full of terrors. Two teams must fight off the darkness, collect resources, and advance through the ages. Daytime finds a desperate rush to gather the resources that can carry you through the impending night whilst growing your city. Plan and expand carefully -- any city that fails to produce enough light will be consumed by darkness.

### Welcome to the Lux AI Challenge Season 1!

The Lux AI Challenge is a competition where competitors design agents to tackle a multi-variable optimization, resource gathering, and allocation problem in a 1v1 scenario against other competitors. In addition to optimization, successful agents must be capable of analyzing their opponents and developing appropriate policies to get the upper hand.

Visit on Kaggle Link - https://www.kaggle.com/c/lux-ai-2021/overview 

## Lux AI Season 1 Specifications

A copy of these rules can also be found at https://www.lux-ai.org/specs-2021

For documentation on the API, see this document. To get started developing a bot, see our Github

### Background
The night is dark and full of terrors. Two teams must fight off the darkness, collect resources, and advance through the ages. Daytime finds a desperate rush to gather and build the resources that can carry you through the impending night. Plan and expand carefully -- any city that fails to produce enough light will be consumed by darkness.

### Environment
In the Lux AI Challenge Season 1, two competing teams control a team of Units and CityTiles that collect resources to fuel their Cities, with the main objective to own as many CityTiles as possible at the end of the turn-based game. Both teams have complete information about the entire game state and will need to make use of that information to optimize resource collection, compete for scarce resources against the opponent, and build cities to gain points.

Each competitor must program their own agent in their language of choice. Each turn, your agent gets 3 seconds to submit their actions, excess time is not saved across turns. In each game, you are given a pool of 60 seconds that is tapped into each time you go over a turn's 3-second limit. Upon using up all 60 seconds and going over the 3-second limit, your agent freezes and can no longer submit additional actions.

The rest of the document will go through the key features of this game.

### The Map

The world of Lux is represented as a 2d grid. Coordinates increase east (right) and south (down). The map is always a square and can be 12, 16, 24, or 32 tiles long. The (0, 0) coordinate is at the top left.

![image](https://user-images.githubusercontent.com/11299574/138728346-eff2fc85-c7b1-47c5-9973-8ebd4f6ad944.png)

The map has various features including Resources (Wood, Coal, Uranium), Units (Workers, Carts), CityTiles, and Road.

In order to prevent maps from favoring one player over another, it is guaranteed that maps are always symmetric by vertical or horizontal reflection.

Each player will start with a single CityTile and a single worker on that CityTile


### Resources

There are 3 kinds of resources: Wood, Coal, and Uranium (in order of increasing fuel efficiency). These resources are collected by workers, then dropped off once a worker moves on top of a CityTile to then be converted into fuel for the city. Some resources require research points before they are possible to collect.

Wood in particular can regrow. Each turn, every wood tile's wood amount increases by 2.5% of its current wood amount rounded up. Wood tiles that have been depleted will not regrow. Only wood tiles with less than 500 wood will regrow.

![image](https://user-images.githubusercontent.com/11299574/138728524-6a9c1668-3b7f-4c0b-afb9-a93d250a8478.png)

### Collection Mechanics

At the end of each turn, Workers automatically receive resources from all adjacent (North, East, South, West, or Center) resource tiles they can collect resources from according to the current symmetric formula:

* Iterating over uranium, coal, then wood resources:
  * Each unit makes resource collection requests to collect an even number of resources from each adjacent tile of the current iterated resource such that the collected amount takes the unit's cargo above capacity. E.g. worker with 60 wood adjacent to 3 wood tiles asks for 14 from each, receives 40 wood, and wastes 2.
  * All tiles of the current iterated resource then try to fulfill requests, if they can't they make sure all unfulfilled requests get an equal amount, the leftover is wasted. E.g. if 4 workers are mining a tile of 25 wood, but one of them is only asking for 5 while the others are asking for 20 wood each, then first all workers get 5 wood each, leaving 5 wood left over for 3 more workers with space left. This can evenly be distributed by giving 1 wood each to the last 3 workers, leaving 2 wood left that is then wasted.
Workers cannot mine while on CityTiles. Instead, if there is at least one Worker on a CityTile, that CityTile will automatically collect adjacent resources at the same rate as a worker each turn and directly convert it all to fuel. The collection mechanic for a CityTile is the same as a worker and you can treat a CityTile as an individual Worker collecting resources.

### Actions

Units and CityTiles can perform actions each turn given certain conditions. In general, all actions are simultaneously applied and are validated against the state of the game at the start of a turn. The next few sections describe the Units and CityTiles in detail.

### CityTiles
A CityTile is a building that takes up one tile of space. Adjacent CityTiles collectively form a City. Each CityTile can perform a single action provided the CityTile has a Cooldown < 1.

Actions

* Build Worker - Build Worker unit on top of this CityTile (cannot build a worker if the current number of owned workers + carts equals the number of owned CityTiles)
* Build Cart - Build Carts unit on top of this CityTile (cannot build a cart if the current number of owned workers + carts equals the number of owned CityTiles)
* Research - Increase your team’s Research Points by 1

### Units
There are two unit types, Workers, and Carts. Every unit can perform a single action once they have a Cooldown < 1.

All units can choose the move action and move in any of 5 directions, North, East, South, West, Center. Moreover, all units can carry raw resources gained from automatic mining or resource transfer. Workers are capped at 100 units of resources and Carts are capped at 2000 units of resources.

Whenever a unit moves on top of a friendly CityTile, the City that CityTile forms converts all carried resources into fuel.

There can be at most one unit on tiles without a CityTile. Moreover, units cannot move on top of the opposing team’s CityTiles. However, units can stack on top of each other on a friendly CityTile.

If two units attempt to move to the same tile that is not a CityTile, this is considered a collision, and the move action is canceled.

### Workers
Actions

Move - Move the unit in one of 5 directions, North, East, South, West, Center.
Pillage - Reduce the Road level of the tile the unit is on by 0.5
Transfer - Send any amount of a single resource-type from a unit's cargo to another (start-of-turn) adjacent Unit, up to the latter's cargo capacity. Excess is returned to the original unit.
Build CityTile - Build a CityTile right under this worker provided the worker has 100 total resources of any type in their cargo (full cargo) and the tile is empty. If building is successful, all carried resources are consumed and a new CityTile is built with 0 starting resources.

### Carts
Actions

Move - Move the unit in one of 5 directions, North, East, South, West, Center.
Transfer - Send any amount of a single resource-type from a unit's cargo to another (start-of-turn) adjacent Unit, up to the latter's cargo capacity. Excess is returned to the original unit.

### Cooldown

CityTiles, Workers and Carts all have a cooldown mechanic after each action. Units and CityTiles can only perform an action when they have < 1 Cooldown.

At the end of each turn, after Road have been built and pillaged, each unit's Cooldown decreases by 1 and further decreases by the level of the Road the unit is on at the end of the turn. CityTiles are not affected by road levels and cooldown always decreases by 1. The minimum Cooldown is 0.

After an action is performed, the unit’s Cooldown will increase by a Base Cooldown.

![image](https://user-images.githubusercontent.com/11299574/138730448-e4728847-c437-4ccd-90ad-70df00570d49.png)


### Roads

As Carts travel across the map, they start to create roads that allow all Units to move faster (see Cooldown). At the end of each turn, Cart will upgrade the road level of the tile it ends on by 0.75. The higher the road level, the faster Units can move and perform actions. All tiles start with a road level of 0 and are capped at 6.

Moreover, CityTiles automatically have the max road level of 6.

Roads can also be destroyed by Workers via the pillage action which reduces road level by 0.5 each time.

If a City is consumed by darkness, the road level of all tiles in the City's CityTiles will go back to 0.

### Day/Night Cycle

The Day/Night cycle consists of a 40 turn cycle, the first 30 turns being day turns, the last 10 being night turns. There are a total of 360 turns in a match, forming 9 cycles.

During the night, Units and Cities need to produce light to survive. Each turn of night, each Unit and CityTile will consume an amount of fuel, see table below for rates. Units in particular will use their carried resources to produce light whereas CityTiles will use their fuel to produce light.

Workers and Carts will only need to consume resources if they are not on a CityTile. When outside the City, Workers and Carts must consume whole units of resources to satisfy their night needs, e.g. if a worker carries 1 wood and 5 uranium on them, they will consume a full wood for 1 fuel, then a full unit of uranium to fulfill the last 3 fuel requirements, wasting 37 fuel. Units will always consume the least efficient resources first.

Lastly, at night, Units gain 2x more Base Cooldown

Should any Unit during the night run out of fuel, they will be removed from the game and disappear into the night forever. Should a City run out of fuel, however, the entire City with all of the CityTiles it owns will fall into darkness and be removed from the game.

![image](https://user-images.githubusercontent.com/11299574/138730573-a00e7380-0ec4-40d9-88a7-a001d5839767.png)

### Game Resolution order

To help avoid confusion over smaller details of how each turn is resolved, we provide the game resolution order here and how actions are applied.

Actions in the game are first all validated against the current game state to see if they are valid. Then the actions, along with game events, are resolved in the following order and simultaneously within each step

CityTile actions along with increased cooldown
Unit actions along with increased cooldown
Roads are created
Resource collection
Resource drops on CityTiles
If night time, make Units consume resources and CityTiles consume fuel
Regrow wood tiles that are not depleted to 0
Cooldowns are handled / computed for each unit and CityTile
The only exception to the validation criteria is that units may move smoothly between spaces, meaning if two units are adjacent, they can swap places in one turn.

Otherwise, actions such as one unit building a CityTile, then another unit moving on top of the new CityTile, are not allowed as the current state does not have this newly built city and units cannot move on top of other units outside of CityTiles.


### Win Conditions

After 360 turns the winner is whichever team has the most CityTiles on the map. If that is a tie, then whichever team has the most units owned on the board wins. If still a tie, the game is marked as a tie.

A game may end early if a team no longer has any more Units or CityTiles. Then the other team wins.

Note on Game Rule Changes will be visible on https://www.kaggle.com/c/lux-ai-2021/rules 
(This is also in the competition rules) 
