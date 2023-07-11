import os
import random
from time import sleep
from dotenv import load_dotenv
from linkedin_v2 import linkedin

# Load environment variables from .env file
load_dotenv('.env')

# Set up LinkedIn API credentials
linkedin_api = linkedin.LinkedInApplication(token=os.getenv('LINKEDIN_ACCESS_TOKEN'))

# Competitor profiles to monitor
competitor_profiles = [
    'competitor1',
    'competitor2',
    'competitor3'
]

# Personalized connection request messages
connection_messages = [
    "Hello {name}, I noticed your recent post about {topic}. I found it insightful and thought we could connect to share ideas.",
    "Hi {name}, I read your 'About Us' section and I'm impressed with your company's mission. Let's connect and explore potential synergies.",
    "Hey {name}, I came across your profile and noticed your expertise in {skill}. I'd love to connect and learn from your experiences."
]

def monitor_competitors():
    new_connections = []

    for profile in competitor_profiles:
        # Fetch new connections from competitor profiles
        response = linkedin_api.get_connections(params={'count': 10, 'start': 0, 'q': 'recent'})
        connections = response.get('elements', [])

        for connection in connections:
            new_connections.append(connection)

    return new_connections

def generate_connection_request(connection):
    # Extract relevant information from the connection
    name = connection.get('firstName', '')
    about = connection.get('summary', '')
    job_title = connection.get('headline', '')

    # Choose a random personalized connection request message
    message = random.choice(connection_messages)

    # Replace placeholders with connection information
    message = message.replace('{name}', name).replace('{topic}', about).replace('{skill}', job_title)

    return message

if __name__ == '__main__':
    # Call the function to monitor competitors' LinkedIn activity
    new_connections = monitor_competitors()

    # Generate personalized connection requests for each new connection
    for connection in new_connections:
        connection_request = generate_connection_request(connection)

        # Send the connection request using the LinkedIn API
        linkedin_api.send_invitation(
            invitation_message=connection_request,
            recipients=[{'person': {'_path': f'/people/{connection["entityUrn"].split(":")[-1]}'}}]
        )

        # Add a delay between sending requests to avoid rate limiting
        sleep(3)
