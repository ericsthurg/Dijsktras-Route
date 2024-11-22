import googlemaps
import networkx as nx
from datetime import datetime, timedelta

# Initialize Google Maps client with your API key
gmaps = googlemaps.Client(key="YOUR_GOOGLE_MAPS_API_KEY")

class RouteOptimizer:
    def __init__(self, start_location, addresses, preferences, time_constraints):
        """
        Initializes the RouteOptimizer with the starting location, list of delivery addresses,
        any address preferences, and time constraints.
        """
        self.start_location = start_location
        self.addresses = addresses
        self.preferences = preferences
        self.time_constraints = time_constraints
        self.graph = nx.DiGraph()  # Directed graph to hold the locations and travel times

    def fetch_travel_time(self, origin, destination):
        """
        Fetches travel time in minutes from origin to destination using Google Maps API.
        Traffic data is included to get the most accurate estimate of travel time.
        
        Parameters:
            origin (str): Starting address
            destination (str): Destination address
            
        Returns:
            float: Travel time in minutes
        """
        directions = gmaps.directions(
            origin,
            destination,
            mode="driving",
            departure_time=datetime.now(),  # Uses current time for traffic estimation
            traffic_model="best_guess"  # Estimates based on current and historical traffic
        )
        
        # Check if directions are available, then retrieve travel time in minutes
        if directions:
            duration = directions[0]['legs'][0]['duration_in_traffic']['value']  # time in seconds
            return duration / 60  # convert seconds to minutes
        else:
            return float('inf')  # Return infinity if destination is unreachable

    def build_graph(self):
        """
        Constructs a directed graph with nodes as locations (addresses) and edges weighted by travel time.
        Each address is connected to every other address to allow flexible routing between all points.
        """
        locations = [self.start_location] + self.addresses  # Include start location with all destinations
        
        # Create edges between every pair of locations with travel times as weights
        for i, origin in enumerate(locations):
            for j, destination in enumerate(locations):
                if i != j:  # Avoid self-loops
                    travel_time = self.fetch_travel_time(origin, destination)
                    self.graph.add_edge(origin, destination, weight=travel_time)

    def dijkstra_shortest_time_path(self):
        """
        Finds the shortest path from the starting location to each address based on travel time
        using Dijkstra's algorithm on the weighted graph.
        
        Returns:
            dict: Shortest paths from the start location to all destinations
        """
        # Computes the shortest paths from start location to all other nodes by travel time
        paths = nx.single_source_dijkstra_path(self.graph, self.start_location, weight="weight")
        return paths

    def optimize_route(self):
        """
        Optimizes the route based on shortest travel time, considering any address preferences
        and time constraints for each delivery.
        
        Returns:
            list: Final ordered route based on quickest path and time constraints
        """
        self.build_graph()  # Build the graph with travel times
        paths = self.dijkstra_shortest_time_path()  # Get shortest time paths from the start location

        final_route = []  # List to store the optimal route
        # Check each address in the calculated path to ensure it meets time constraints
        for address in paths[self.addresses[-1]]:
            if self.meets_time_constraints(address):
                final_route.append(address)  # Only add addresses that fit within time constraints

        return final_route

    def meets_time_constraints(self, address):
        """
        Checks if a delivery meets its specified time constraints (if any).
        
        Parameters:
            address (str): Address to check against constraints
            
        Returns:
            bool: True if the address meets the time constraints, False otherwise
        """
        if address in self.time_constraints:
            no_earlier_than, no_later_than = self.time_constraints[address]
            current_time = datetime.now()
            # Estimate the arrival time to the address
            estimated_arrival = current_time + timedelta(minutes=self.fetch_travel_time(self.start_location, address))

            # Check if arrival time falls within the "no earlier than" and "no later than" constraints
            if (no_earlier_than and estimated_arrival < no_earlier_than) or \
               (no_later_than and estimated_arrival > no_later_than):
                return False
        return True  # Return True if no constraints are violated

# Example usage of the RouteOptimizer class
start_location = "123 Main St, City A"  # Starting point
addresses = ["456 Maple Ave, City A", "789 Oak St, City B"]  # List of delivery addresses
preferences = {"first": "456 Maple Ave, City A"}  # Delivery preference
# Time constraints (earliest and latest delivery time)
time_constraints = {"789 Oak St, City B": (datetime.now() + timedelta(hours=1), None)}

# Initialize the optimizer
optimizer = RouteOptimizer(start_location, addresses, preferences, time_constraints)
optimal_route = optimizer.optimize_route()  # Calculate optimized route
print("Optimized Route:", optimal_route)  # Output the final optimized route
