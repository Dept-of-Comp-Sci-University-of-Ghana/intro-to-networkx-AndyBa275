[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/AYCdDWDE)

#import networkx as nx
import plotly.graph_objects as go
import numpy as np
from collections import Counter

# Create the network graph
G = nx.Graph()

# Add nodes (research group members)
members = ['Andy', 'Michael', 'Henry', 'James', 'Emma', 'Bright']
G.add_nodes_from(members)

# Add edges (collaborative relationships)
# This represents who collaborates with whom
edges = [
    ('Andy', 'Michael'),
    ('Andy', 'Henry'),
    ('Andy', 'James'),
    ('Andy', 'Emma'),
    ('Michael', 'Henry'),
    ('Michael', 'James'),
    ('James', 'Emma'),
    ('Emma', 'Bright'),
    ('Henry', 'Emma')
]
G.add_edges_from(edges)

# ========================================
# COMPUTE NETWORK METRICS
# ========================================


# 1. Number of nodes and edges
num_nodes = G.number_of_nodes()
num_edges = G.number_of_edges()
print(f"\n1. BASIC METRICS:")
print(f"   Number of Nodes: {num_nodes}")
print(f"   Number of Edges: {num_edges}")

# 2. Degree distribution
degrees = dict(G.degree())
print(f"\n2. DEGREE FOR EACH NODE:")
for node, degree in sorted(degrees.items(), key=lambda x: x[1], reverse=True):
    print(f"   {node}: {degree}")

# Degree distribution statistics
degree_values = list(degrees.values())
degree_freq = Counter(degree_values)
print(f"\n3. DEGREE DISTRIBUTION:")
for degree, count in sorted(degree_freq.items()):
    print(f"   Degree {degree}: {count} node(s)")

# 4. Check for isolated nodes
isolated_nodes = list(nx.isolates(G))
print(f"\n4. ISOLATED NODES:")
if isolated_nodes:
    print(f"   Isolated nodes: {', '.join(isolated_nodes)}")
else:
    print(f"   No isolated nodes found (all members are connected)")

# 5. Additional metrics
print(f"\n5. ADDITIONAL METRICS:")
avg_degree = np.mean(degree_values)
print(f"   Average Degree: {avg_degree:.2f}")

density = nx.density(G)
print(f"   Network Density: {density:.3f}")

# Check if graph is connected
is_connected = nx.is_connected(G)
print(f"   Is Connected: {is_connected}")

if is_connected:
    diameter = nx.diameter(G)
    avg_path_length = nx.average_shortest_path_length(G)
    print(f"   Network Diameter: {diameter}")
    print(f"   Average Path Length: {avg_path_length:.2f}")

# Centrality measures
degree_centrality = nx.degree_centrality(G)
betweenness_centrality = nx.betweenness_centrality(G)
closeness_centrality = nx.closeness_centrality(G)

print(f"\n6. CENTRALITY MEASURES:")
print(f"   Degree Centrality (most connected):")
for node, cent in sorted(degree_centrality.items(), key=lambda x: x[1], reverse=True):
    print(f"      {node}: {cent:.3f}")

print(f"\n   Betweenness Centrality (bridge between groups):")
for node, cent in sorted(betweenness_centrality.items(), key=lambda x: x[1], reverse=True):
    print(f"      {node}: {cent:.3f}")

#VASUALIZATION OF GRAPH ANALYSIS

# Use spring layout for node positions
pos = nx.spring_layout(G, seed=42, k=1, iterations=50)

# Create edge traces
edge_x = []
edge_y = []
for edge in G.edges():
    x0, y0 = pos[edge[0]]
    x1, y1 = pos[edge[1]]
    edge_x.extend([x0, x1, None])
    edge_y.extend([y0, y1, None])

edge_trace = go.Scatter(
    x=edge_x, y=edge_y,
    line=dict(width=2, color='#888'),
    hoverinfo='none',
    mode='lines')

# Create node traces
node_x = []
node_y = []
node_text = []
node_color = []
node_size = []

for node in G.nodes():
    x, y = pos[node]
    node_x.append(x)
    node_y.append(y)
    
    # Node information for hover
    degree = degrees[node]
    node_text.append(f"{node}<br>Degree: {degree}<br>Connections: {list(G.neighbors(node))}")
    
    # Color Andy differently
    if node == 'Andy':
        node_color.append('#FF6B6B')  
    else:
        node_color.append('#4ECDC4')  
    
    # Size based on degree
    node_size.append(20 + degree * 10)

node_trace = go.Scatter(
    x=node_x, y=node_y,
    mode='markers+text',
    text=[node for node in G.nodes()],
    textposition="top center",
    hoverinfo='text',
    hovertext=node_text,
    marker=dict(
        showscale=False,
        color=node_color,
        size=node_size,
        line=dict(width=2, color='white')))

# Create the figure
fig = go.Figure(data=[edge_trace, node_trace],
                layout=go.Layout(
                    title=dict(
                        text='<b>Research Group Network</b><br>Node size represents degree (number of connections)',
                        x=0.5,
                        xanchor='center',
                        font=dict(size=20)
                    ),
                    showlegend=False,
                    hovermode='closest',
                    margin=dict(b=20, l=5, r=5, t=80),
                    xaxis=dict(showgrid=False, zeroline=False, showticklabels=False),
                    yaxis=dict(showgrid=False, zeroline=False, showticklabels=False),
                    plot_bgcolor='white',
                    height=600
                ))

fig.show()



degrees_list = sorted(degree_freq.items())
x_degrees = [f"Degree {d}" for d, _ in degrees_list]
y_counts = [count for _, count in degrees_list]

fig2 = go.Figure(data=[
    go.Bar(x=x_degrees, y=y_counts, 
           marker=dict(color='#4ECDC4'),
           text=y_counts,
           textposition='auto')
])

fig2.update_layout(
    title='<b>Degree Distribution</b>',
    xaxis_title='Degree',
    yaxis_title='Number of Nodes',
    plot_bgcolor='white',
    height=400
)

fig2.show()

#EXPLANATION

print("\n" + "=" * 60)
print("NETWORK INTERPRETATION")
print("=" * 60)

print(f"""
This research group consists of {num_nodes} members with {num_edges} collaborative
relationships. Here are the key insights:

1. CONNECTIVITY:
   - Network density: {density:.3f} ({'highly' if density > 0.5 else 'moderately'} connected)
   - Average degree: {avg_degree:.2f} (each member collaborates with ~{int(avg_degree)} others)
   - {'No isolated members - good group cohesion!' if not isolated_nodes else f'Warning: {len(isolated_nodes)} isolated member(s)'}

2. KEY PLAYERS:
   - Most connected: {max(degrees, key=degrees.get)} (degree: {max(degrees.values())})
   - Bridge connector: {max(betweenness_centrality, key=betweenness_centrality.get)}
   - {max(degrees, key=degrees.get)} serves as a central hub for collaboration

3. COLLABORATION PATTERNS:
   - The group shows {'a star-like structure' if max(degrees.values()) >= num_nodes - 1 else 'distributed collaboration'}
   - Members with low degree (1-2 connections) might benefit from broader engagement

4. RECOMMENDATIONS:
   - Encourage more connections for members with degree < 2
   - Foster collaboration between disconnected subgroups
   - Maintain {max(degrees, key=degrees.get)}'s central role while developing backup connectors
""")

print("=" * 60)
