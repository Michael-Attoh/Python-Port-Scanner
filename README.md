# Python-Port-Scanner
Custom Python Port Scanner: A multithreaded port scanner built in Python to identify open ports on a target IP. Utilizes socket for connection attempts, threading for concurrent scans, and queue for managing port scans. Configurable to scan standardized, reserved, or custom ports.


# **Introduction**
Port scanners are crucial tools for both penetration testing and information gathering. They are used to detect open ports on a host system. This can help in two primary ways: ensuring that your own servers are secure or identifying potential vulnerabilities in someone else's systems. An open port on a server can represent a security risk if not properly managed.

In this tutorial, we will code our own port scanner in Python. While there are professional open-source tools like Nmap available, creating a custom scanner will allow us to tailor it to our needs and gain valuable programming experience. Python is a great choice for this task due to its simplicity, modern features, and powerful capabilities.

Important Note: Port scanning someone else's network without permission is illegal. Only scan networks you own or have explicit authorization to test. Alternatively, you can use scanme.nmap.org for practice. The responsibility for any misuse of this information rests with the user.


## **Basic Functionality of a Port Scanner**
A port scanner is a tool used to detect open ports on a target IP address. Ports are essential for communication over a network, each serving a specific function. For instance, port 80 is used for HTTP, and port 443 for HTTPS. Other important ports include 21 (FTP), 22 (SSH), and 25 (SMTP). With over 130,000 ports available, including 1,023 well-known and 48,128 reserved ports, focusing on key ports is crucial for effective scanning.

Here’s a simple way to scan ports using Python:

```python
def portscan(port):
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((target, port))
        return True
    except:
        return False
```


In this example, we created a socket that attempts to connect to a specified port on a target IP address. If the connection is successful, the function returns True; if an exception occurs, it returns False. We then print the result for each port.

To enhance the functionality, we can automate the process by using a for-loop to scan all standardized ports, checking each one to determine if it is open.

```python
for port in range(1, 1024):
    result = portscan(port)
    if(result):
        print("Port {} is open!".format(port))
    else:
        print("Port {} is closed!".format(port))
```
<img width="314" alt="1" src="https://github.com/user-attachments/assets/4f9f487e-c269-4a64-b1ad-bc2e08770e69">


Actually, this is a complete port scanner already. However, it has a drawback: it scans one port at a time, which can be quite slow. To improve the speed, we can use multithreading to scan multiple ports simultaneously. Let’s dive into how we can enhance the code with multithreading.

## **Preparing the Ports**
To begin, we'll need to import several libraries that will assist in our port scanning and multithreading tasks:

```python
from queue import Queue
import socket
import threading
```
- Socket will be used for making connection attempts to the host at specific ports. 

- Threading will enable us to run multiple scanning functions simultaneously, significantly speeding up the process. 

- Queue, as a data structure, will help manage access to ports by multiple threads, ensuring that each port is scanned only once, even when threads are running concurrently.

We will also define three global variables to be used throughout our functions:

```python
target = "127.0.0.1"
queue = Queue()
open_ports = []
```

- The target is the IP address or domain of the host we intend to scan. 

- The queue is initially empty but will later be populated with the ports that need to be scanned. 
- Finally, we have an empty list that will store the open port numbers once scanning is complete.

We'll begin by implementing the portscan function, which we have already discussed.

```python
# socket.socket(): This creates a new socket
# socket.AF_INET: This specifies the address family for the socket
# socket.SOCK_STREAM: This specifies the socket type, which in this case is a TCP socket 

def portscan(port):
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((target, port))
        return True
    except:
        return False 
```

In this code, we use a basic try-except block. The function attempts to connect to the target IP address on a specified port using a socket. 

If the connection is successful, the function returns True, indicating that the port is open. 

If an error occurs during the connection attempt, the exception is caught, and the function returns False, meaning the port is assumed to be closed.


**Before diving into threading, we first need to define the ports to scan. To achieve this, we will create a function called get_ports**

```python
def get_ports(mode):
    if mode == 1:
        for port in range(1, 1024):
            queue.put(port)
    elif mode == 2:
        for port in range(1, 49152):
            queue.put(port)
    elif mode == 3:
        ports = [20, 21, 22, 23, 25, 53, 80, 110, 443]
        for port in ports:
            queue.put(port)
    elif mode == 4:
        ports = input("Enter your ports (seperate by blank):")
        ports = ports.split()
        ports = list(map(int, ports))
        for port in ports:
            queue.put(port)

```
In this function, we have defined four different modes for selecting ports:

- Mode 1: Scans the first 1,023 standardized ports.

- Mode 2: Includes the additional 48,128 reserved ports.

- Mode 3: Targets a list of some of the most crucial ports.

- Mode 4: Allows for manual entry of ports.

After determining the ports based on the selected mode, we add them to the queue.

In Mode 4, when ports are entered manually, the input is split into a list of strings. We then use a mapping function to convert these strings into integers to ensure they are usable in the port scanning process.

## **Multithreading**

The next step is to define a **worker function** for our threads. This function will handle the following tasks:

- Retrieve Port Numbers: Extract port numbers from the queue.

- Scan Ports: Use the portscan function to check if each port is open or closed.

- Print Results: Output the scan results for each port.

By using multithreading, we can efficiently manage concurrent scanning of ports, greatly improving the speed and effectiveness of our port scanner.
```python
def worker():
    while not queue.empty():
        port = queue.get()
        if portscan(port):
            print("Port {} is open!".format(port))
            open_ports.append(port)
        else:
            print("Port {} is closed!".format(port))
```

The worker function is designed to process port scanning tasks efficiently. It operates as follows:

- Queue Processing: Continuously retrieves and scans ports from the queue until it is empty.

- Port Scanning: Uses the portscan function to check each port. If a port is open, it prints the port number and adds it to the open_ports list.

- Closed Ports: Prints information about closed ports. However, you might consider removing this part of the code, as it may clutter the output and provide less useful information.

Next, we will implement the main function to create, start, and manage the threads, ensuring that our port scanning is performed efficiently and concurrently.

```python
def run_scanner(threads, mode):
    # Populate the queue with ports based on the specified mode
    get_ports(mode)

    # List to hold thread objects
    thread_list = []

    # Create and start threads
    for t in range(threads):
        thread = threading.Thread(target=worker)
        thread_list.append(thread)

    # Start all threads
    for thread in thread_list:
        thread.start()

    # Wait for all threads to complete
    for thread in thread_list:
        thread.join()

    # Print the list of open ports
    print("Open ports are:", open_ports)

```

In this function, we define two parameters: the number of threads to start and the scanning mode. 

We first call get_ports to load the ports based on the selected mode. 

We then initialize an empty list to hold our threads. For each thread, we create a new thread object assigned to the worker function and add it to the list. 

After starting all the threads, they work concurrently to scan the ports.

Finally, we wait for all threads to complete using join() and then print the list of open ports.

```python
run_scanner(100, 1)
```
<img width="437" alt="2" src="https://github.com/user-attachments/assets/13d9e9a0-2edf-4ada-a005-02475435ba03">

By executing the function with the specified parameters, we scan all standardized ports using a set number of threads. For instance, using a hundred threads allows us to scan approximately one hundred ports per second. You can adjust this number to increase the scanning speed; with my computer, I achieved around 800 port scans per second at the maximum.

The results will display the status of each port, indicating whether it is open or closed.

**I hope this tutorial was helpful to you!** 










