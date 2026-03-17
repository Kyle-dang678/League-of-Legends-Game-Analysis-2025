Project for DSC80 at UCSD | Kyle Dang

## Introduction

League of Legends is a competitive multiplayer online battle arena (MOBA) game, where two teams of five players aim to destroy the opposing team's base. In professional esports, understanding what drives a team to victory is a key question for analysts, coaches, and fans.

This project analyzes the **2025 League of Legends Esports Match Data** from Oracle's Elixir, which contains detailed statistics for professional matches played in 2025. The dataset contains **120,636 rows** and **165 columns**, where each row represents either a single player's performance or a team's aggregate performance in a game.

The central question of this project is: **Does having a gold advantage at 15 minutes predict whether a team will win the game?**

Gold is one of the most important resources in League of Legends — it allows teams to buy items that makes individual champions stronger. The benchmark of 15 minutes is appropriate since it is at this phase of the game where players begin contesting for map objectives and largely marks the end of the "laning" or "leveling" phase. A gold lead at 15 minutes indicates early game dominance, but does it actually translate to winning?

The relevant columns for our analysis are:

| Column             | Description                                                                                                                                          |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `result`           | Game outcome: 1 = win, 0 = loss                                                                                                                      |
| `golddiffat15`     | Gold difference between the team and their opponent at 15 minutes                                                                                    |
| `xpdiffat15`       | Experience difference at 15 minutes - XP is gained by being near creature deaths, and determines a champion's level which unlocks stronger abilities |
| `csdiffat15`       | Creep score difference at 15 minutes - CS measure the number of creatures killed, which is the primary way players earn gold through farming         |
| `golddiffat10`     | Gold difference at 10 minutes                                                                                                                        |
| `killsat15`        | Number of kills at 15 minutes                                                                                                                        |
| `firstblood`       | Whether the team secured first blood                                                                                                                 |
| `firstdragon`      | Whether the team secured first dragon                                                                                                                |
| `datacompleteness` | Whether the game data is complete or partial                                                                                                         |
