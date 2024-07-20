# Software-Defined Networking Lab TTP

## Project Goal
The goal of this project is to set up a Software-Defined Networking (SDN) lab environment. By the end of this project, you will have hands-on experience with SDN concepts, be able to create and manage network topologies programmatically, and understand how to implement network policies using a centralized controller.

## Overview
This TTP provides detailed instructions for setting up an SDN lab environment using Mininet and OpenDaylight controller, allowing you to experiment with SDN concepts and technologies.

![SDN Architecture Overview](https://example.com/path/to/sdn_architecture.png)
*Figure 1: SDN Architecture Overview. Replace with an actual diagram showing the components of an SDN architecture.*

## Prerequisites
- A Linux machine or virtual machine (Ubuntu 20.04 LTS recommended)
- Basic understanding of networking concepts
- Familiarity with Linux command line
- Knowledge of Python programming (for custom network applications)

## Step-by-step Guide

### 1. Install Dependencies

Update the system and install necessary packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git ansible python3-pip openjdk-11-jre-headless -y
```

### 2. Install Mininet

1. Clone the Mininet repository:

```bash
git clone https://github.com/mininet/mininet
```

2. Install Mininet:

```bash
cd mininet
util/install.sh -a
```

3. Verify the installation:

```bash
sudo mn --test pingall
```

![Mininet Test](https://example.com/path/to/mininet_test.png)
*Figure 2: Mininet Test Output. Replace with an actual screenshot of the Mininet test output.*

### 3. Install OpenDaylight Controller

1. Download OpenDaylight:

```bash
wget https://nexus.opendaylight.org/content/repositories/opendaylight.release/org/opendaylight/integration/karaf/0.13.1/karaf-0.13.1.tar.gz
```

2. Extract the archive:

```bash
tar -xzvf karaf-0.13.1.tar.gz
```

3. Start OpenDaylight:

```bash
cd karaf-0.13.1/bin
./karaf
```

4. Install necessary features in the Karaf console:

```
feature:install odl-restconf odl-l2switch-switch odl-mdsal-apidocs odl-dlux-core
```

### 4. Create a Simple SDN Topology

1. Create a Python script `sdn_topology.py`:

```python
from mininet.net import Mininet
from mininet.node import Controller, RemoteController, OVSController
from mininet.node import CPULimitedHost, Host, Node
from mininet.node import OVSKernelSwitch, UserSwitch
from mininet.node import IVSSwitch
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.link import TCLink, Intf
from subprocess import call

def myNetwork():
    net = Mininet(topo=None, build=False, ipBase='10.0.0.0/8')
    
    info('*** Adding controller\n')
    c0 = net.addController(name='c0', controller=RemoteController, ip='127.0.0.1', protocol='tcp', port=6633)

    info('*** Add switches\n')
    s1 = net.addSwitch('s1', cls=OVSKernelSwitch)
    s2 = net.addSwitch('s2', cls=OVSKernelSwitch)

    info('*** Add hosts\n')
    h1 = net.addHost('h1', cls=Host, ip='10.0.0.1', defaultRoute=None)
    h2 = net.addHost('h2', cls=Host, ip='10.0.0.2', defaultRoute=None)

    info('*** Add links\n')
    net.addLink(h1, s1)
    net.addLink(h2, s2)
    net.addLink(s1, s2)

    info('*** Starting network\n')
    net.build()
    
    info('*** Starting controllers\n')
    for controller in net.controllers:
        controller.start()

    info('*** Starting switches\n')
    net.get('s1').start([c0])
    net.get('s2').start([c0])

    info('*** Post configure switches and hosts\n')

    CLI(net)
    net.stop()

if __name__ == '__main__':
    setLogLevel('info')
    myNetwork()
```

2. Run the SDN topology:

```bash
sudo python3 sdn_topology.py
```

![SDN Topology](https://example.com/path/to/sdn_topology.png)
*Figure 3: SDN Topology Visualization. Replace with an actual diagram or screenshot of your SDN topology.*

### 5. Verify SDN Functionality

1. In the Mininet CLI, test connectivity:

```
mininet> pingall
```

2. View switch flow tables:

```
mininet> sh ovs-ofctl dump-flows s1
```

### 6. Implement Traffic Engineering

1. Create a Python script `te_app.py` to implement simple traffic engineering:

```python
from ryu.base import app_manager
from ryu.controller import ofp_event
from ryu.controller.handler import CONFIG_DISPATCHER, MAIN_DISPATCHER
from ryu.controller.handler import set_ev_cls
from ryu.ofproto import ofproto_v1_3
from ryu.lib.packet import packet
from ryu.lib.packet import ethernet
from ryu.lib.packet import ether_types

class TrafficEngineering(app_manager.RyuApp):
    OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

    def __init__(self, *args, **kwargs):
        super(TrafficEngineering, self).__init__(*args, **kwargs)
        self.mac_to_port = {}

    @set_ev_cls(ofp_event.EventOFPSwitchFeatures, CONFIG_DISPATCHER)
    def switch_features_handler(self, ev):
        datapath = ev.msg.datapath
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser

        match = parser.OFPMatch()
        actions = [parser.OFPActionOutput(ofproto.OFPP_CONTROLLER,
                                          ofproto.OFPCML_NO_BUFFER)]
        self.add_flow(datapath, 0, match, actions)

    def add_flow(self, datapath, priority, match, actions, buffer_id=None):
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser

        inst = [parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS,
                                             actions)]
        if buffer_id:
            mod = parser.OFPFlowMod(datapath=datapath, buffer_id=buffer_id,
                                    priority=priority, match=match,
                                    instructions=inst)
        else:
            mod = parser.OFPFlowMod(datapath=datapath, priority=priority,
                                    match=match, instructions=inst)
        datapath.send_msg(mod)

    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    def _packet_in_handler(self, ev):
        # Implement your traffic engineering logic here
        pass
```

2. Run the traffic engineering application:

```bash
ryu-manager te_app.py
```

### 7. Visualize Network Topology

1. Install NetworkX and Matplotlib:

```bash
pip3 install networkx matplotlib
```

2. Create a Python script `visualize_topology.py`:

```python
import networkx as nx
import matplotlib.pyplot as plt

def draw_topology():
    G = nx.Graph()
    G.add_edge('s1', 'h1')
    G.add_edge('s1', 's2')
    G.add_edge('s2', 'h2')

    pos = nx.spring_layout(G)
    nx.draw(G, pos, with_labels=True, node_color='lightblue', node_size=500, font_size=16, font_weight='bold')
    nx.draw_networkx_nodes(G, pos, nodelist=['h1', 'h2'], node_color='lightgreen', node_size=700)
    nx.draw_networkx_nodes(G, pos, nodelist=['s1', 's2'], node_color='lightblue', node_size=700, node_shape='s')
    
    plt.title("SDN Topology")
    plt.axis('off')
    plt.show()

if __name__ == '__main__':
    draw_topology()
```

3. Run the visualization script:

```bash
python3 visualize_topology.py
```

![Visualized Topology](https://example.com/path/to/visualized_topology.png)
*Figure 4: Visualized SDN Topology. Replace with an actual output of your visualization script.*

## Advanced Topics

- Implementing Quality of Service (QoS) policies
- Exploring OpenFlow protocol details
- Integrating with external applications via northbound APIs
- Implementing network slicing in SDN
- Developing custom SDN applications for specific use cases

## Troubleshooting

- If Mininet fails to create the topology, check for conflicting network namespaces
- For controller connection issues, verify the controller's IP and port in the Mininet script
- If flows are not being installed, check the OpenFlow version compatibility between switches and controller
- Use Wireshark to capture and analyze OpenFlow messages for debugging

## Best Practices

1. Always use version control (e.g., Git) for your SDN configurations and applications
2. Implement proper error handling and logging in your SDN applications
3. Test your SDN applications thoroughly before deploying in a production environment
4. Keep your SDN controller and switch software up to date
5. Implement security measures such as TLS for controller-switch communications
6. Document your network topology and SDN applications for easier maintenance

## Additional Resources

- [Mininet Documentation](http://mininet.org/documentation/)
- [OpenDaylight Documentation](https://docs.opendaylight.org/)
- [Ryu SDN Framework](https://ryu.readthedocs.io/)
- [SDN: Software Defined Networks (Book)](https://www.oreilly.com/library/view/sdn-software-defined/9781449342425/)
- [OpenFlow Specification](https://opennetworking.org/software-defined-standards/specifications/)

## Glossary

- **SDN**: Software-Defined Networking, an approach to network management that enables dynamic, programmatically efficient network configuration
- **OpenFlow**: A communications protocol that gives access to the forwarding plane of a network switch or router over the network
- **Mininet**: A network emulator which creates a network of virtual hosts, switches, controllers, and links
- **OpenDaylight**: An open-source SDN controller platform
- **Ryu**: A component-based software-defined networking framework
- **Flow Table**: A table in an OpenFlow switch that contains a set of flow entries
- **Northbound API**: APIs that abstract the lower-level details of the network and provide high-level network services to applications

