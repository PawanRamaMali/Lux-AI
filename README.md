# Lux-AI 
Kaggle Lux AI - Gather the most resources and survive the night!

This is not the official representation of the Lux AI Challenge. 

The purpose of this repository is to discuss open source algorithms which can be implemented to solve the challenge.

## Introduction

The night is dark and full of terrors. Two teams must fight off the darkness, collect resources, and advance through the ages. Daytime finds a desperate rush to gather the resources that can carry you through the impending night whilst growing your city. Plan and expand carefully -- any city that fails to produce enough light will be consumed by darkness.

## Welcome to the Lux AI Challenge Season 1!

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
