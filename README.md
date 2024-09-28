# Football Players Network

This project explores the relationships between football players by constructing a network (graph) based on the players' shared experiences in teams. It uses web scraping to gather data from Transfermarkt and employs graph theory to visualize and analyze the connections between players, providing insights into how players are linked through their careers.

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#key-features)
- [Data Source](#data-source)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Graph Building Process](#graph-building-process)
- [Future Projects](#future-projects)
- [Visualization](#visualization)
- [Contributing](#contributing)
- [Authorship](#authorship)

## Project Overview

Football is a highly interconnected sport, with players moving between clubs and national teams throughout their careers. The goal of this project is to build a network where players are nodes, and an edge between two players exists if they have played in the same team at some point in their career.

### Key Features

- **Web Scraping:** The project utilizes Python libraries such as `Requests` and `BeautifulSoup` to scrape data from Transfermarkt, specifically targeting player and team pages.
- **Graph Construction:** Using `networkx`, players are represented as nodes, and their shared team experiences are used to build edges between them.
- **Visualization:** The network is visualized using `matplotlib`, and potential future work includes implementing interactive visualizations with `Plotly` or `Bokeh`.
- **Data Cleaning:** Data is preprocessed and cleaned to ensure that relationships between players are unique and accurately represented.

## Data Source

Data is scraped from [Transfermarkt](https://www.transfermarkt.com/), a popular website that provides detailed information on football players, teams, transfers, and more.

## Requirements

The project uses the following Python libraries:

- `requests`
- `beautifulsoup4`
- `networkx`
- `plotly`
- `pandas`

Ensure you have these dependencies installed before running the project.

## Installation

To get started with this project, clone the repository and install the required dependencies:

```bash
git clone https://github.com/veronicamars73/Football-Players-Network.git
cd Football-Players-Network
pip install -r requirements.txt
```

## Usage

1. Open the Jupyter notebook accessing-player-data.ipynb.
2. Run the notebook to start the web scraping process and build the player network.
3. Explore the graph visualizations and the relationships between football players.

## Graph Building Process

The process involves the following steps:

1. Scraping Player Data: Web scraping is used to gather data on each player's teammates from their Transfermarkt profile.
2. Handling Pagination: To ensure all data is captured, pagination is handled during the scraping process.
3. Constructing Relationships: A graph is built where each player is a node, and an edge is created between two players if they have been teammates.

## Future Projects

Using the Graph we have built a number of analysis can be made. Like the shortest path between players, Cluster Analysis, degree of centrality, this and more can be add to the project in the future.

## Visualization

The graph is visualized using NetworkX and Plotly. Nodes represent players, and edges show the teammate connections between them. You can explore clusters, identify key players with many connections, and see how players from different teams and countries are connected.

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request for any changes.

## Authorship

Repository developed by Lisandra Melo (<lisandramelo34@gmail.com>).
