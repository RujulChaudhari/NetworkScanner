# Network and Port Scanner Documentation

## Step 1: Set Up Your Development Environment

**Install Python:**

1. Download and install Python from the [official website](https://www.python.org/downloads/).
2. During installation, make sure to check the box that says "Add Python to PATH."

**Install Required Packages:**

1. Open a terminal or command prompt.
2. Run the following commands to install the necessary Python packages:

```bash
pip install scapy
pip install pyinstaller
```

## Step 2: Write the Network and Port Scanner Code
Create a New Python Script:
Open your preferred code editor (e.g., VSCode, Atom, Sublime Text).
Create a new Python script, for example, network_port_scanner.py.

## Write the Code:
Copy and paste the following Python code into your script:
``` bash

# import scapy.all as scapy
import socket
import csv
import sys

# Check if running in a console environment
try:
    from tkinter import filedialog
    from tkinter import Tk
    console = False
except ImportError:
    console = True

def scan(ip):
    # Create an ARP request packet
    arp_request = scapy.ARP(pdst=ip)
    
    # Create an Ethernet frame to contain the ARP request packet
    broadcast = scapy.Ether(dst="ff:ff:ff:ff:ff:ff")
    
    # Combine the Ethernet frame and ARP request packet
    arp_request_broadcast = broadcast/arp_request
    
    # Send the packet and receive the response
    answered_list = scapy.srp(arp_request_broadcast, timeout=1, verbose=False)[0]
    
    # Create a list to store results
    results = []
    
    # Iterate through the list of answered packets
    for element in answered_list:
        client_dict = {"ip": element[1].psrc, "mac": element[1].hwsrc}
        results.append(client_dict)
    
    return results

def scan_ports(ip, ports):
    port_status = {}

    try:
        # Iterate through specified ports
        for port in ports:
            # Create a socket object
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            
            # Set a timeout for the connection attempt
            sock.settimeout(1)
            
            # Attempt to connect to the IP and port
            result = sock.connect_ex((ip, port))
            
            # Check if the connection was successful
            if result == 0:
                port_status[port] = "Open"
            else:
                port_status[port] = "Closed"
            
            # Close the socket
            sock.close()
    
    except socket.error:
        print(f"[-] Unable to connect to {ip}")

    return port_status

def save_to_csv(results_list, port_status):
    if not console:
        root = Tk()
        root.withdraw()  # Hide the main window
        directory = filedialog.askdirectory(title="Select a directory to save the CSV file")
    else:
        directory = input("Enter the directory to save the CSV file: ")

    if directory:
        if not console:
            filename = filedialog.asksaveasfilename(
                initialdir=directory,
                title="Save CSV file",
                defaultextension=".csv",
                filetypes=[("CSV files", "*.csv"), ("All files", "*.*")]
            )
        else:
            filename = input("Enter the filename to save the CSV file (e.g., scan_results.csv): ")

        if filename:
            with open(filename, mode='w', newline='') as file:
                writer = csv.DictWriter(file, fieldnames=["ip", "mac", "port", "status"])
                writer.writeheader()

                for client in results_list:
                    for port, status in port_status[client["ip"]].items():
                        writer.writerow({"ip": client["ip"], "mac": client["mac"], "port": port, "status": status})
                
            print(f"\nResults saved to {filename}")

def print_result(results_list, port_status):
    if not console:
        print("IP Address\t\tMAC Address\tPort\tStatus")
        print("-----------------------------------------")
        for client in results_list:
            for port, status in port_status[client["ip"]].items():
                print(f"{client['ip']}\t\t{client['mac']}\t\t{port}\t{status}")
    else:
        print("\nResults:")
        for client in results_list:
            for port, status in port_status[client["ip"]].items():
                print(f"IP: {client['ip']}, MAC: {client['mac']}, Port: {port}, Status: {status}")

if __name__ == "__main__":
    # Get the target IP range from the user
    target_ip = input("Enter the target IP address or range (e.g., 192.168.1.1/24): ")

    # Get the target ports from the user
    target_ports = list(map(int, input("Enter the target ports (comma-separated): ").split(',')))

    # Perform the network scan
    scan_result = scan(target_ip)

    # Perform the port scan on discovered devices
    port_status = {}
    for client in scan_result:
        print(f"\nScanning ports for {client['ip']}...")
        port_status[client["ip"]] = scan_ports(client["ip"], target_ports)

    # Print the network scan results and port status
    print_result(scan_result, port_status)

    # Ask the user if they want to export the results as a CSV file
    export_csv = input("\nDo you want to export the results as a CSV file? (yes/no): ").lower()

    # Save results to a CSV file if the user chooses to export
    if export_csv == "yes":
        save_to_csv(scan_result, port_status)

    # Add this line to keep the console window open
    input("\nPress Enter to exit...")

```
## Step 3: Test the Python Script
Test the Script:
Open a terminal or command prompt.

Navigate to the directory where your script is located.

Run the script using the command:
``` bash
python network_port_scanner.py
```
Follow the prompts to enter the target IP range, target ports, and choose whether to export results as a CSV file.

## Step 4: Create a Standalone Executable
Install PyInstaller:
Open a terminal or command prompt.

Run the following command to install PyInstaller:
``` bash
pip install pyinstaller
```
Create the Executable:
In the same terminal, navigate to the directory where your script is located.

Run the following command to create the executable:
``` bash
pyinstaller --onefile network_port_scanner.py
```
This command creates a standalone executable file in the dist directory.

Run the Executable:
- Navigate to the dist directory.
- Run the generated executable (e.g., network_port_scanner.exe).
- Test the executable to ensure it behaves as expected.

## Conclusion:
By following these steps, you should have a working network and port scanner script with the ability to export results as a CSV file. Additionally, you've created a standalone executable for easy distribution. Feel free to customize the script further based on your needs or experiment with additional features.

