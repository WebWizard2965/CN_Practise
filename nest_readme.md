from nest.topology import Topology
import time

# Initialize topology
topo = Topology()

# Add end devices (ed), switches (s), and router (r)
ed1 = topo.add_host("ed1")   # End Device 1
ed2 = topo.add_host("ed2")   # End Device 2
ed3 = topo.add_host("ed3")   # End Device 3
ed4 = topo.add_host("ed4")   # End Device 4

s1 = topo.add_switch("s1")   # Switch 1
s2 = topo.add_switch("s2")   # Switch 2

r = topo.add_router("r")     # Router

# Add links between devices
topo.add_link(ed1, s1, bw=100)    # Link between ED1 and Switch 1
topo.add_link(ed2, s2, bw=100)    # Link between ED2 and Switch 2
topo.add_link(ed3, s2, bw=100)    # Link between ED3 and Switch 2
topo.add_link(ed4, s1, bw=100)    # Link between ED4 and Switch 1

topo.add_link(s1, r, bw=100)     # Link between Switch 1 and Router
topo.add_link(s2, r, bw=100)     # Link between Switch 2 and Router

# Deploy topology
topo.deploy()

# Ping between ed1 and ed2 to check connectivity
print("\nPinging from ed1 to ed2")
ed1.cmd("ping -c 4 10.0.0.2")  # 4 packets from ed1 to ed2

# Run iperf server on ed2
print("\nRunning iperf server on ed2")
ed2.cmd("iperf -s &")  # Start iperf server in the background

# Run iperf client on ed1 to test bandwidth
print("\nRunning iperf client on ed1 to ed2")
time.sleep(1)  # Wait a bit for the server to start
ed1.cmd("iperf -c 10.0.0.2 -t 10 -i 1")  # Run iperf client for 10 seconds

# Run iperf server on ed3
print("\nRunning iperf server on ed3")
ed3.cmd("iperf -s &")  # Start iperf server on ed3

# Run iperf client on ed4 to test bandwidth
print("\nRunning iperf client on ed4 to ed3")
time.sleep(1)  # Wait a bit for the server to start
ed4.cmd("iperf -c 10.0.0.3 -t 10 -i 1")  # Run iperf client for 10 seconds

# Cleanup
topo.destroy()


to run sudo -E python3 filename.py
