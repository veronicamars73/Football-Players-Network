# Building a Football Players Network: A Practical Introduction to Graph Theory

## What is a Graph?

In 1736, mathematician Leonard Euler tackled a famous problem known as the **Seven Bridges of Königsberg**. The problem arose in Königsberg, a city now called Kaliningrad (formerly in Germany), where two islands were connected by seven bridges to two mainland areas.¹ A representation of the city and the bridges is shown by the image below.

![Wikipédia](https://prod-files-secure.s3.us-west-2.amazonaws.com/0a3b1b74-615a-438a-8b23-07b9188540d8/577d49d1-f73a-42bb-bd41-8d1f62d1798f/image.png)

Wikipédia

The question posed was whether one could find a path through the city that crossed every bridge exactly once and returned to the starting point. Euler proved that no such path exists, and in doing so, laid the foundations of **graph theory**.¹
Graph theory, a branch of mathematics concerned with networks of points (vertices) connected by lines (edges), provides powerful tools for modeling relationships. A **graph** is a data structure highly useful for representing and solving real-world problems.² It has an intuitive visual representation and includes important algorithms and properties for analyzing complex data.
A graph consists of two main elements: **nodes** (or vertices) and **edges**. The nodes represent individual entities, and the edges represent the relationships or connections between them. In the case of the Seven Bridges of Königsberg problem, the land areas are represented as nodes, while the bridges are represented as edges.

Here's the graph representation of Euler's famous problem:

![Graph Theory, Konigsberg Problem¹](https://prod-files-secure.s3.us-west-2.amazonaws.com/0a3b1b74-615a-438a-8b23-07b9188540d8/7abc11cb-7ee6-4b90-b1e6-f5c134eada7c/image.png)

Graph Theory, Konigsberg Problem¹

You can see in the representation, each part of the city as a circle - node - and each bridge as a line - edge - connecting the respective part of the land.

In this tutorial, we will apply these concepts to the world of football. We’ll scrape data on football players, build a graph representing relationships between players and their teammates, and visualize it. Our primary tool will be Python, and we’ll use libraries like Selenium for web scraping, NetworkX for graph building, and Plotly for visualization.

## Building a Graph of Football Players' Teammates

In the context of football, we can use graphs to model relationships between players based on their teams. Each football player (node) is connected to their teammates (other nodes) by edges. The more teammates a player has, the more connected they are in this graph. This can help identify key social players within a team or league.

### Step 1: Import Necessary Libraries

In this tutorial, we'll employ **Selenium** and **BeautifulSoup** for web scraping.

```python
# 1. Data Manipulation
import pandas as pd  # For handling and analyzing structured data

# 2. Graph Construction
import networkx as nx  # For creating and analyzing network graphs

# 3. Data Visualization
import plotly.graph_objs as go  # For interactive graph visualization

# 4. Web Scraping
from selenium import webdriver  # Automates browser interaction
from selenium.webdriver.chrome.service import Service  # For managing the Chrome service
from bs4 import BeautifulSoup  # For parsing HTML data

# 5. Utility Libraries
import time  # For adding delays in scraping
import re  # For regular expressions, used to clean and extract data
```

### Step 2: Define Constants

Next, we’ll define a few constants. For example, `NUM_PLAYERS` specifies the number of players we’ll scrape data for, and `NUM_TEAMMATES` defines how many teammates to retrieve for each player.

```python
NUM_PLAYERS = 75  # Modify this value to scrape more players
NUM_TEAMMATES = 5  # Number of teammates to scrape for each player
```

Feel free to adjust these values based on your needs. The more players you scrape, the richer your final graph will be, but it will also take more time.

### Step 3: Set Up Selenium WebDriver

Selenium automates interactions with web pages, such as clicking buttons, entering data, and navigating across multiple pages. This makes it ideal for scraping dynamic websites like Transfermarkt, which we’ll use to gather player data.

To use Selenium it is important to set up the WebDriver in your Python script, ensuring the path to your WebDriver is correctly specified.

```python
# Configuring Chrome options (e.g., headless mode can be set here if needed)
chrome_options = webdriver.chrome.options.Options()

# Path to the Chrome WebDriver executable
chrome_driver = "C:\\your\\path\\to\\the\\executable\\chromedriver.exe"

# Initializing the WebDriver service with the executable path
service_to_pass = Service(executable_path=chrome_driver)

# Launching the WebDriver with the service and options
wd = webdriver.Chrome(service=service_to_pass, options=chrome_options)
```

### Step 4: Access the Webpage

Now that Selenium is set up, we’ll use it to access player data from Transfermarkt. Specifically, we’ll extract player names, their respective teams, and URLs to their player profiles.

```python
# Define the URL for the most valuable players on Transfermarkt
url = 'https://www.transfermarkt.com/spieler-statistik/wertvollstespieler/marktwertetop?land_id=0&ausrichtung=alle&spielerposition_id=alle&altersklasse=alle&jahrgang=0&kontinent_id=0&plus=1'
# Open the webpage using the Selenium webdriver
wd.get(url)
```

### Step 5: Scraping Player Data

Now we going to define two functions that will be neccessary in our scrapping code.

The first one is used to extract the player information from a web page of the most valuable players and then add the extracted information to a list of players data.

```python
# 3.1 Scrape Player Information from a Web Page

def scrape_page(html):
    """
    Extracts player information from a web page of the most valuable players. 
    The information of each player is added to the players_data list.

    Args:
        html (str): The page source that will be scraped.

    Returns:
        None: Appends player information to the global players_data list.
    """
    soup = BeautifulSoup(html, 'html.parser')
    
    # Find all player rows in the table (rows with 'odd' and 'even' class)
    player_rows = soup.find_all('tr', class_=['odd', 'even'])
    
    # Extract player information from each row
    for player in player_rows:
        name = player.find_all('td')[1].find('img').get('alt')  # Player's name
        player_link = 'https://www.transfermarkt.com' + player.find('td', class_='hauptlink').find('a')['href']  # Player page link
        
        # Add player data to the global list (uncomment if needed: rank, age, club, market value)
        players_data.append({
            'Name': name,
            'Player Page': player_link,
        })

```

The next one is used to get the address of the next page of players in the Pagination.

```python
# 3.3 Extract the Link to the Next Page in Pagination

def change_page(html):
    """
    Extracts the URL of the next page from a paginated web page.

    Args:
        html (str): The HTML source of the current page.

    Returns:
        link (str): The URL of the next page to be scraped.
    """
    soup = BeautifulSoup(html, 'html.parser')

    # Find the 'Next Page' button and extract the link
    next_button = soup.find('li', class_='tm-pagination__list-item tm-pagination__list-item--icon-next-page')
    link = 'https://www.transfermarkt.com' + next_button.find('a')['href']
    
    return link
```

With the two functions, we can gather the players using the iteration below.

```python
# Initialize an empty list to store player data
players_data = []
# Pagination: Continue scraping until you gather 250 players or run out of pages
players_count = 0

while players_count < NUM_PLAYERS:
    time.sleep(2)  # Wait for the page to fully load
    
    # Scrape data from the current page
    try:
        scrape_page(wd.page_source)  # Scrape player data from the current page
        players_count = len(players_data)  # Update the player count
        
        # Move to the next page using the 'change_page' function
        url = change_page(wd.page_source)
        wd.get(url)
    except:
        print('There was a problem accessing the data or no more players were found')
        break
```

Now we can convert the list with data into a Dataframe.

```python
# Convert the players data list into a DataFrame for future analysis
df_players = pd.DataFrame.from_dict(players_data[:NUM_PLAYERS])
df_players
```

### Step 6: Scraping Teammates' Information and Build Graph

After gathering data on individual players, we’ll scrape information on their teammates. This is critical for building our graph, as teammates will form the connections (edges) between players.

For that, we will need a function to get the teammates page for each player.

```python
# 3.6 Convert Player Profile URL to Teammates Page URL

def teamates_page(df_row):
    """
    Modifies a player's profile URL to navigate to the page listing their most frequently played teammates.

    Args:
        df_row (Series): A row from the DataFrame containing the player's profile URL.

    Returns:
        url (str): The modified URL pointing to the player's teammates page.
    """
    # Replace "profil" with "gemeinsameSpiele" in the player's profile URL to get the teammates page URL
    return df_row['Player Page'].replace("profil", "gemeinsameSpiele")
```

We can apply the function to our Dataframe to populate a new column with the teammates page for each player.

```python
# Add a new column to the DataFrame with URLs to each player's teammates page
df_players['Teammates Page'] = df_players.apply(teamates_page, axis=1)
```

Now we can move on to accessing the teammate page and add that information to construct our graph.

For that we will need another function to scrap the data.

```python
def scrape_teammates(html):
    """
    Extracts players information from a web page of a player's most played teammates. 
    The information of each player is added to the players_data list.

    Args:
        html (str): The page source that will be scraped.

    Returns:
        teamates_vet (DataFrame): A DataFrame of all the teammates on the page.
    """
    soup = BeautifulSoup(html, 'html.parser')
    
    # Find the table containing teammate data
    teammate_table = soup.find('table', {'class': 'items'})
    teamates_vet = []  # List to store teammate data
    count_runs = 0  # Counter to handle iterations over the rows

    for tm in teammate_table.find_all('tr')[1:]:  # Skip the header row
        if count_runs % 3 == 0:  # Process every third row
            try:
                # Extract teammate name and profile link
                teammate_name = tm.find('img').get('alt')
                player_link = 'https://www.transfermarkt.com' + tm.find('td', class_='hauptlink').find('a')['href']
                
                # Add teammate data to global players_data and local teamates_vet
                players_data.append({
                    'Name': teammate_name,
                    'Player Page': player_link,
                })
                teamates_vet.append({
                    'Name': teammate_name,
                    'Player Page': player_link,
                })
            except:
                # Handle any errors in data extraction gracefully
                print("There was a problem gathering data from the row below")
                print(tm)
        
        count_runs += 1  # Increment the counter

    # Return the collected teammate data as a DataFrame
    return pd.DataFrame(teamates_vet)
```

Alongside a function to add the information of each player and their relationship to a graph.

```python
# 3.8 Add a Player and Their Teammates to the Graph

def add_to_graph(player, teammates):
    """
    Adds a player and their teammates to a NetworkX graph (G), creating nodes and edges.

    Args:
        player (dict): A dictionary containing player data to be added as a node.
        teammates (DataFrame): A DataFrame containing teammate data to be added as nodes and connected to the player.

    Returns:
        None: Modifies the global graph G in place.
    """
    # Add the player as a node if not already in the graph
    if player['Player Page'] not in G:
        G.add_node(player['Player Page'], data=player)

    # Iterate through the teammates DataFrame
    for index, row in teammates.iterrows():
        # Add the teammate as a node if not already in the graph
        if row['Player Page'] not in G:
            G.add_node(row['Player Page'], data=row)
        
        # Add an edge between the player and teammate if no existing edge is found
        if not G.has_edge(player['Player Page'], row['Player Page']) and not G.has_edge(row['Player Page'], player['Player Page']):
            G.add_edge(player['Player Page'], row['Player Page'])
```

The function above is responsible to constructing the graph, it uses  the players' and teammates' data to create nodes and edges in a NetworkX graph. Each player and their teammates are represented as nodes, while the relationships between them are represented as edges.

The next step involves iterating over the list of players and applying the scraping function to extract each player's teammates, then adding the data to the graph.

```python
# Initialize an empty graph to store players and their relationships
G = nx.Graph()

# Iterate through each player in the DataFrame
for index, row in df_players.iterrows():
    url = row['Teammates Page']
    wd.get(url)  # Open the player's teammates page
    row = row.drop(labels='Teammates Page')  # Drop the teammates page URL from the current row for cleaner data
    
    # Initialize variables to store teammate data
    teammate_count = 0
    teammates = pd.DataFrame(columns=['Name', 'Player Page'])  # Empty DataFrame to store teammates
    
    # Loop to gather up to 25 teammates from each player's page
    while teammate_count < NUM_TEAMMATES:
        # Scrape teammate data from the current page
        page_teammates = scrape_teammates(wd.page_source)
        
        # Concatenate the current page's teammates to the overall list
        teammates = pd.concat([teammates, page_teammates[:NUM_TEAMMATES]])
        teammate_count = len(teammates)
        
        # Try to navigate to the next page of teammates
        try:
            url = change_page(wd.page_source)
            wd.get(url)
        except:
            print('There was a problem accessing the data or no more players were found')
            break
    
    # Print the scraped teammates and player data for debugging
    print(teammates)
    print(row)
    
    # Add the player and their teammates to the graph
    add_to_graph(row, teammates)
```

This code will iterate through each player, collect their most-played-with teammates, and add both the player and their teammates to the graph, building the network incrementally.

### Step 7: Visualizing the Football Network

Once the graph is constructed, the next step is to visualize the network of relationships between football players. We will use the Plotly library for interactive graph visualization. But one thing that can help with the readability of the graph befor plotting it, is to instead of showing the players pages as the node name, we can show their name. For that we use the function below.

```python
# 3.5 Extract Player Name from a Transfermarkt Profile URL

def extract_player_name(url):
    """
    Extracts the player's name from a Transfermarkt profile URL and formats it.
    
    Args:
        url (str): The URL from which the player's name will be extracted.

    Returns:
        player_name (str): The player's name with hyphens replaced by spaces and proper capitalization.
        If no match is found, the original URL is returned.
    """
    # Use regex to find the player name part of the URL
    match = re.search(r'transfermarkt\.com/([^/]+)/', url)
    
    if match:
        # Extract the name and replace hyphens with spaces
        player_name = match.group(1).replace("-", " ")
        
        # Capitalize the first letter of each word (e.g., 'john-doe' becomes 'John Doe')
        player_name = player_name.title()
        return player_name
    
    # Return the original URL if no match is found
    return url

```

Finally, we’ll use Plotly to visualize the graph. Plotly allows for interactive, web-based visualizations, perfect for exploring complex networks like this one.

```python
# Position nodes in the graph using a spring layout
pos = nx.spring_layout(G)

# Extract edge positions
edge_x = []
edge_y = []
for edge in G.edges():
    x0, y0 = pos[edge[0]]
    x1, y1 = pos[edge[1]]
    edge_x.append(x0)
    edge_x.append(x1)
    edge_x.append(None)  # For separating edges in the plot
    edge_y.append(y0)
    edge_y.append(y1)
    edge_y.append(None)

# Create the edge trace (lines)
edge_trace = go.Scatter(
    x=edge_x, y=edge_y,
    line=dict(width=1, color='black'),
    hoverinfo='none',
    mode='lines'
)

# Extract node positions
node_x = []
node_y = []
for node in G.nodes():
    x, y = pos[node]
    node_x.append(x)
    node_y.append(y)

# Node adjacency and hover text
node_adjacencies = []
node_text = []
for node, adjacencies in G.adjacency():
    num_connections = len(adjacencies)  # Number of connections for the node
    node_adjacencies.append(num_connections)
    node_text.append(f"Node {extract_player_name(node)} has {num_connections} connections")

# Create the node trace (points)
node_trace = go.Scatter(
    x=node_x, y=node_y,
    mode='markers',
    hoverinfo='text',
    marker=dict(
        showscale=True,
        colorscale='viridis',
        size=10,
        color=node_adjacencies,
        colorbar=dict(
            thickness=15,
            title='Node Connections',
            xanchor='left',
            titleside='right'
        ),
    )
)

# Assign node hover text
node_trace.text = node_text

# Create the plot
fig = go.Figure(data=[edge_trace, node_trace],
                layout=go.Layout(
                    title='Interactive Network Graph',
                    titlefont_size=16,
                    showlegend=False,
                    hovermode='closest',
                    margin=dict(b=0, l=0, r=0, t=40),
                    annotations=[dict(
                        text="Network visualization using NetworkX and Plotly",
                        showarrow=False,
                        xref="paper", yref="paper",
                        x=0.005, y=-0.002
                    )],
                    xaxis=dict(showgrid=False, zeroline=False),
                    yaxis=dict(showgrid=False, zeroline=False)
                )
)

# Display the graph
fig.show()
```

In the visualization, each node represents a player, and the edges show teammate relationships. Larger nodes indicate players with more connections. You can hover over each node to view player details, and the graph layout (spring layout) ensures that players with more connections are placed closer together.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/0a3b1b74-615a-438a-8b23-07b9188540d8/b2f58f91-311a-4e88-8c64-4493885dd9ec/image.png)

The resulting graph provides a visual summary of player relationships. Players with many teammates will appear as larger, more central nodes, while isolated players or those with fewer connections will be on the periphery. This type of visualization can highlight key players who serve as the "connectors" within their teams or leagues.

### **Conclusion and Next Steps**

In this tutorial, we’ve built a graph using data scraped from Transfermarkt and visualized the relationships between football players and their teammates. This exercise provides a foundation for more advanced analysis, such as identifying key players using graph algorithms like centrality or even expanding the dataset to include players more players and more teammates for each player.

### **References**

¹ George, Betsy. "Graph Theory, Konigsberg Problem". In *Encyclopedia of GIS*, eds. Shashi Shekar and Hui Xiong, SpringerScience+Business Media, LLC., 2008, pp. 409–412.

² Goldbarg, Marco Cesar; Goldbarg, Elizabeth. “Grafos - Conceitos, Algoritmos e Aplicações”. 1o ed,  Elsevier Editora Ltda, 2012.
