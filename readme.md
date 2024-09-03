# DHCP Server Simulation

This project simulates a Dynamic Host Configuration Protocol (DHCP) server and client interaction. It consists of two main Python scripts: `dhcp_server.py` and `dhcp_client.py`.

## Overview

The DHCP simulation implements the basic DHCP message exchange process:

1. DHCPDISCOVER
2. DHCPOFFER
3. DHCPREQUEST
4. DHCPACK

The server manages IP address allocation, lease times, and client information, while the client requests and manages its IP address lease.

## Features

### DHCP Server (`dhcp_server.py`)

- IP address pool management
- Support for subnet and range-based IP allocation
- IP reservation for specific MAC addresses
- Blacklist functionality
- Lease time management
- Multi-threaded client handling
- Client information tracking
- Command to show connected clients

### DHCP Client (`dhcp_client.py`)

- DHCP message creation and parsing
- Automatic IP address acquisition
- Lease time tracking
- Retry mechanism with exponential backoff

## Configuration

The server uses a JSON configuration file (`configs.json`) to set various parameters:

- Pool mode (subnet or range)
- Subnet details (if applicable)
- IP range (if applicable)
- Lease time
- Reservation list
- Blacklist

## Usage

1. Start the DHCP server:
   ```
   python dhcp_server.py
   ```

2. Run one or more DHCP clients:
   ```
   python dhcp_client.py
   ```

3. To view connected clients on the server, type `show clients` in the server console.

## Detailed Functionality

### DHCP Server

1. **IP Pool Management**: 
   - Supports both subnet and range-based IP allocation
   - Handles reserved IPs and blacklisted MAC addresses

2. **Client Handling**:
   - Multi-threaded to handle multiple clients simultaneously
   - Processes DHCPDISCOVER and DHCPREQUEST messages
   - Sends DHCPOFFER and DHCPACK messages

3. **Lease Management**:
   - Tracks lease times for each allocated IP
   - Automatically frees expired leases

4. **Client Information**:
   - Stores client details including MAC address, assigned IP, and lease expiration time
   - Provides a command to display current client information

## Explanation of the DHCP Server Code

## Detailed Code Explanation: dhcp_server.py

### Classes

#### `IP_Handler`
This class manages individual IP addresses.
- `__init__(self, ip)`: Initializes an IP address object.
- `setMac(self, mac)`: Associates a MAC address with the IP.
- `set_IP_unavailable()`: Marks the IP as in use.
- `keep_IP_address()`: Marks the IP as reserved.
- `make_IP_available()`: Marks the IP as available.
- `release_IP_address()`: Releases the IP, making it available and clearing its MAC association.

#### `IP_Pool`
This class manages the pool of IP addresses.
- `__init__(self, configsObject)`: Initializes the IP pool based on configuration.
- `reserve_ips(self, reservation_list)`: Reserves specific IPs for given MAC addresses.
- `generate_ips_subnet(self, in_ip, mask)`: Generates IP addresses based on subnet configuration.
- `generate_ips_range(self, start, end)`: Generates IP addresses based on a range.
- `getFreeAddress(self, _mac)`: Finds and returns a free IP address.
- `assign_IPAddress(self, _ip, _mac)`: Assigns a specific IP to a MAC address.
- `getIPAddress(self, _mac)`: Gets an IP address for a given MAC, considering reservations and existing assignments.
- `findIPObjByIPAddr(self, _ip)`: Finds an IP_Handler object by IP address.
- `free_IP(self, _ip)`: Frees an IP address.
- `findIPByMac(self, _mac)`: Finds IP(s) associated with a MAC address.

### Functions

#### `read_configs(file_name)`
Reads and returns the JSON configuration file.

#### `make_DHCPOFFER_message(order, DISCOVER_elements)`
Creates DHCPOFFER or DHCPACK messages based on the received DISCOVER or REQUEST message.

#### `decode_DHCP_message(message)`
Decodes received DHCP messages into a dictionary of elements.

#### `handle_discover()`
Placeholder function for handling DISCOVER messages.

#### `handle_DHCP_message(p_elements)`
Handles different types of DHCP messages based on the message type.

#### `show_clients()`
Displays information about connected clients.

#### `decrease_lease_t()`
Continuously decreases lease times and frees expired leases.

#### `handle_client()`
Handles individual client requests in a separate thread.

### Main Execution Flow

1. Reads configuration and initializes the IP pool.
2. Sets up the UDP server socket.
3. Starts threads for showing clients and decreasing lease times.
4. Enters a loop to receive and handle client messages:
   - Receives a message
   - Starts a new thread to handle the client request
   - Continues listening for new messages

This server implementation demonstrates key DHCP server functionalities including IP allocation, lease management, and multi-threaded client handling. It uses object-oriented programming for IP management and leverages Python's threading capabilities for concurrent operations.

### DHCP Client

1. **DHCP Message Exchange**:
   - Sends DHCPDISCOVER to request an IP address
   - Processes DHCPOFFER from the server
   - Sends DHCPREQUEST to confirm IP address
   - Handles DHCPACK from the server

2. **Lease Management**:
   - Tracks the lease time of the assigned IP address
   - Initiates renewal process when the lease is close to expiration

3. **Retry Mechanism**:
   - Implements exponential backoff for retries if no response is received from the server

## Explanation of the DHCP Client Code
## Detailed Code Explanation: dhcp_client.py

### Functions

#### `read_configs(file_name)`
- Reads the JSON configuration file.
- Returns the configuration as a Python object.

#### `make_DHCP_message(configsObject, order, OFFER_elements)`
- Creates DHCP messages (DHCPDISCOVER or DHCPREQUEST).
- Parameters:
  - `configsObject`: Configuration settings
  - `order`: Type of message ('DHCPDISCOVER' or 'DHCPREQUEST')
  - `OFFER_elements`: Elements from the DHCPOFFER message (for DHCPREQUEST)
- Constructs the message with appropriate fields (op, htype, hlen, xid, etc.).
- Returns the constructed message as bytes.

#### `decode_DHCP_message(message)`
- Decodes received DHCP messages (DHCPOFFER or DHCPACK).
- Parses the message into its constituent elements.
- Returns a dictionary with decoded message fields.

#### `communicate()`
- Handles the DHCP communication process:
  1. Sends DHCPDISCOVER
  2. Receives DHCPOFFER
  3. Sends DHCPREQUEST
  4. Receives DHCPACK
- Updates global variables `main_IP`, `start_time`, and `send_req_time`.

#### `decrease_lease_t()`
- Continuously checks and decreases the lease time.
- Resets `main_IP` and `start_time` when the lease expires.

### Main Execution Flow

1. Reads configuration from 'configs.json'.
2. Sets up a UDP socket for communication.
3. Starts a thread to run the `communicate()` function for initial DHCP exchange.
4. Enters a main loop that:
   - Checks if it needs to request a new IP (based on initial_interval or timeout).
   - Starts a new `communicate()` thread if needed.
   - Updates the initial_interval using exponential backoff.
   - Checks lease time and frees the IP if the lease has expired.

### Key Variables

- `main_IP`: Stores the currently assigned IP address.
- `start_time`: Tracks when the current lease started.
- `send_req_time`: Tracks when the last DHCPREQUEST was sent.
- `initial_interval`: Time interval for retrying DHCP requests.
- `backoff_cutoff`: Maximum backoff time for retries.
- `Timeout`: Time to wait for a response before retrying.

### DHCP Client Behavior

1. **Initial Request**: 
   - Sends DHCPDISCOVER to request an IP.
   - Processes DHCPOFFER and sends DHCPREQUEST to confirm.
   - Handles DHCPACK to receive the assigned IP.

2. **Lease Management**:
   - Tracks the lease time of the assigned IP.
   - Frees the IP when the lease expires.

3. **Retry Mechanism**:
   - Uses exponential backoff for retries if no response is received.
   - Increases the retry interval up to `backoff_cutoff`.

4. **Timeout Handling**:
   - Retries the DHCP process if no response is received within `Timeout` seconds.

This client implementation demonstrates the core functionalities of a DHCP client, including message exchange, lease management, and retry mechanisms. It uses threading for non-blocking communication and implements a basic exponential backoff algorithm for retries.

## Network Details

- The server listens on IP 127.0.0.1 (localhost) and port 31
- The client uses IP 127.0.0.1 (localhost) and port 33
- Communication uses UDP broadcast (255.255.255.255) for DHCP messages

## Future Improvements

- Implement DHCPRELEASE message handling
- Add support for additional DHCP options
- Enhance error handling and logging
- Implement a graphical user interface for easier management

## Dependencies

- Python 3.x
- Standard Python libraries (socket, json, threading, etc.)

## Note

This is a simulation for educational purposes and does not implement all features of a full DHCP server. It should not be used in a production environment.
