ed = end device, r = router, s = switch

ed - s - r - s - ed
    '        '
    ed      ed 

#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/bridge-module.h"
#include "ns3/applications-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("CustomTopology");

int main(int argc, char *argv[])
{
    CommandLine cmd;
    cmd.Parse(argc, argv);

    // Create nodes
    NodeContainer endDevices;
    endDevices.Create(4); // ED1, ED2, ED3, ED4

    Ptr<Node> switch1 = CreateObject<Node>(); // S1
    Ptr<Node> switch2 = CreateObject<Node>(); // S2
    Ptr<Node> router = CreateObject<Node>();  // R

    InternetStackHelper internet;
    internet.Install(endDevices);
    internet.Install(router);

    // Switch nodes don't need full IP stack
    BridgeHelper bridge;

    PointToPointHelper p2p;
    p2p.SetDeviceAttribute("DataRate", StringValue("100Mbps"));
    p2p.SetChannelAttribute("Delay", StringValue("2ms"));

    // === Switch 1: Connect ED1, ED2, Router ===
    NetDeviceContainer s1Devices;

    for (int i = 0; i < 2; ++i) {
        NetDeviceContainer link = p2p.Install(NodeContainer(endDevices.Get(i), switch1));
        s1Devices.Add(link.Get(1));
        internet.Assign(Ipv4AddressHelper("10.1." + std::to_string(i+1) + ".0", "255.255.255.0").Assign(link));
    }

    NetDeviceContainer rToS1 = p2p.Install(NodeContainer(router, switch1));
    s1Devices.Add(rToS1.Get(1));
    Ipv4AddressHelper addrR1("10.1.3.0", "255.255.255.0");
    Ipv4InterfaceContainer ifaceR1 = addrR1.Assign(rToS1);

    bridge.Install(switch1, s1Devices);

    // === Switch 2: Connect ED3, ED4, Router ===
    NetDeviceContainer s2Devices;

    for (int i = 2; i < 4; ++i) {
        NetDeviceContainer link = p2p.Install(NodeContainer(endDevices.Get(i), switch2));
        s2Devices.Add(link.Get(1));
        internet.Assign(Ipv4AddressHelper("10.1." + std::to_string(i+1) + ".0", "255.255.255.0").Assign(link));
    }

    NetDeviceContainer rToS2 = p2p.Install(NodeContainer(router, switch2));
    s2Devices.Add(rToS2.Get(1));
    Ipv4AddressHelper addrR2("10.1.6.0", "255.255.255.0");
    Ipv4InterfaceContainer ifaceR2 = addrR2.Assign(rToS2);

    bridge.Install(switch2, s2Devices);

    // Enable IP forwarding on the router
    Ptr<Ipv4> ipv4 = router->GetObject<Ipv4>();
    ipv4->SetAttribute("IpForward", BooleanValue(true));

    // Example Application: ping from ED1 to ED4
    V4PingHelper ping("10.1.4.1"); // ED4's IP
    ping.SetAttribute("Verbose", BooleanValue(true));
    ApplicationContainer pingApps = ping.Install(endDevices.Get(0));
    pingApps.Start(Seconds(2.0));
    pingApps.Stop(Seconds(10.0));

    Ipv4GlobalRoutingHelper::PopulateRoutingTables();

    Simulator::Run();
    Simulator::Destroy();

    return 0;
}

https://chatgpt.com/share/6811c0fa-22e4-800a-91a7-4d366d2bbbc1
