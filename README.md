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


