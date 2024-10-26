# Lab - Configure DHCPv6

## Topology

![topology](image.png)

## Addressing table

|  Device   | Interface | IPv6 Address          |
|-----------|-----------|-----------------------|
|R1         |G0/0/0     |2001:db8:acad:2::1/64  |
|           |           | fe80::1               |
|           |G0/0/1     |2001:db8:acad:1::1/64  |
|           |           | fe80::1               |
|R2         |G0/0/0     |2001:db8:acad:2::2/64  |
|           |           | fe80::2               |
|           |G0/0/1     |2001:db8:acad:3::1/64  |
|           |           | fe80::1               |
|PC-A       | NIC       | DHCP                  |
|PC-B       | NIC       | DHCP                  |

## Objectives

Part 1: Build the Network and Configure Basic Device Settings
Part 2: Verify SLAAC address assignment from R1
Part 3: Configure and verify a Stateless DHCPv6 Server on R1
Part 4: Configure and verify a Stateful DHCPv6 Server on R1
Part 5: Configure and verify a DHCPv6 Relay on R2

### Solution

*File *.cpt [here](hw_03_IPv6.pkt)*

#### Part 1: Build the Network and Configure Basic Device Settings

##### Step 1: Cable the network as shown in the topology

I connected the devices as shown in the topological diagram, and connect the cable.

##### Step 2: Configure basic settings for each switch

    a. Assign a device name to the switch.
    b. Disable DNS lookup to prevent the router from attempting to translate incorrectly entered commands as though they were host names.
    c. Assign **class** as the privileged EXEC encrypted password.
    d. Assign **cisco** as the console password and enable login.
    e. Assign **cisco** as the VTY password and enable login.
    f. Encrypt the plaintext passwords.
    h. Save the running configuration to the startup configuration file.
    i. Set the clock on the switch to today's time and date.
    j. Copy the running configuration to the startup configuration.
    
        SW-1(config)#line console 0 
        SW-1(config-line)#logging synchronous 
        SW-1(config-line)#password cisco
        SW-1(config-line)#login
        SW-1(config-line)#exit
        SW-1(config)#no ip domain-lookup
        SW-1(config)#hostname SW-1
        SW-1(config)#enable secret class
        SW-1(config)#service password-encryption
        SW-1(config)#line vty 0 4 
        SW-1(config-line)#password cisco
        SW-1(config-line)#login
        SW-1(config-line)#end
        SW-1#clock set 18:22:00 23 Oct 2024
        SW-1#copy run start

##### Step 3: Configure basic settings for each router

    a. Assign a device name to the router **hostname R1** and **hostname R2**.
    b. Disable DNS lookup (**no ip domain-lookup** command)to prevent the router from attempting to translate incorrectly entered commands as though they were host names.
    c. Assign **class** as the privileged EXEC encrypted password.
    d. Assign **cisco** as the console password and enable login.
    e. Assign **cisco** as the VTY password and enable login.
    f. Encrypt the plaintext passwords **service password-encryption**.
    g. Enable IPv6 Routing
    h. Save the running configuration to the startup configuration file **copy run start**.
    i. Set the clock on the router to today’s time and date.

    For example configuration **R2**:

        Router(config)#line console 0 
        Router(config-line)#logging synchronous 
        Router(config-line)#password cisco
        Router(config-line)#login
        Router(config-line)#exit
        Router(config)#no ip domain-lookup
        Router(config)#hostname R2
        R2(config)#enable secret class
        R2(config)#service password-encryption
        R2(config)#line vty 0 4 
        R2(config-line)#password cisco
        R2(config-line)#login
        R2(config-line)#ipv6 unicast-routing
        R2(config)#end
        R2#clock set 18:22:00 25 Oct 2024
        R2#copy run start

##### Step 4: Configure interfaces and routing for both routers

    a. Configured the G0/0/0 and G0/0/1 interfaces on R1 and R2 with the IPv6 addresses specified in the table above.
        
        R1(config)#interface g0/0/0
        R1(config-if)#ipv6 address 2001:db8:acad:2::1/64
        R1(config-if)#ipv6 address fe80::1 link-local 
        R1(config)#interface g0/0/1
        R1(config-if)#ipv6 address 2001:db8:acad:1::1/64
        R1(config-if)#ipv6 address fe80::1 link-local

    b. Configured a default route on each router pointed to the IP address of G0/0/0 on the other router.

        R1(config)#ipv6 route ::1/64 2001:db8:acad:2::2
        R2(config)#ipv6 route ::1/64 2001:db8:acad:2::1

    c. Verified routing is working by pinging R2’s G0/0/1 address from R1

        R1#ping ipv6 2001:db8:acad:3::1
        Type escape sequence to abort.
        Sending 5, 100-byte ICMP Echos to 2001:db8:acad:3::1, timeout is 2 seconds:
        !!!!!
        Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

    d. Saved the running configuration to the startup configuration file.

#### Part 2: Verify SLAAC Address Assignment from R1

Powered **PC-A** up and switched the NIC for IPv6 automatic configuration.
I saw the inscription *ipv6 requestsuccessful*
Using the **ipconfig** command, I saw that the PCA had assigned itself an address from the 2001:db8:1::/64 network.

    C:\>ipconfig 

    FastEthernet0 Connection:(default port)

        Connection-specific DNS Suffix..: 
        Link-local IPv6 Address.........: FE80::209:7CFF:FED7:572B
        IPv6 Address....................: 2001:DB8:ACAD:1:209:7CFF:FED7:572B
        IPv4 Address....................: 0.0.0.0
        Subnet Mask.....................: 0.0.0.0
        Default Gateway.................: FE80::1
                                            0.0.0.0

**Where did the host-id portion of the address come from?** I suppose that through the response from R1 using SLAAC

#### Part 3: Configure and Verify a DHCPv6 server on R1

##### Step 1: Examine the configuration of PC-A in more detail

    a. I executed the **ipconfig /all** command on the PC and studied the result.

        C:\>ipconfig /all

        FastEthernet0 Connection:(default port)

            Connection-specific DNS Suffix..: 
            Physical Address................: 0009.7CD7.572B
            Link-local IPv6 Address.........: FE80::209:7CFF:FED7:572B
            IPv6 Address....................: 2001:DB8:ACAD:1:209:7CFF:FED7:572B
            Autoconfiguration IP Address....: 169.254.87.43
            Subnet Mask.....................: 255.255.0.0
            Default Gateway.................: FE80::1
                                                0.0.0.0
            DHCP Servers....................: 0.0.0.0
            DHCPv6 IAID.....................: 
            DHCPv6 Client DUID..............: 00-01-00-01-91-17-47-67-00-09-7C-D7-57-2B
            DNS Servers.....................: ::
                                                0.0.0.0

    b.  Noticed that there is no Primary DNS suffix. I also noticed that no DNS server addresses are provided for the PC.

##### Step 2: Configure R1 to provide stateless DHCPv6 for PC-A

    a. Created an IPv6 DHCP pool on **R1** named **R1-STATELESS**. As a part of that pool, assigned the DNS server address as 2001:db8:acad::1 and the domain name as stateless.com.

        R1(config)#ipv6 dhcp pool R1-STATELESS
        R1(config-dhcpv6)#dns serve
        R1(config-dhcpv6)#dns-
        R1(config-dhcpv6)#dns-server 2001:db8:acad::254
        R1(config-dhcpv6)#domain-na
        R1(config-dhcpv6)#domain-name STATLESS.com

    b. Configured the G0/0/1 interface on R1 to provide the **OTHER** config flag to the R1 LAN, and specified the DHCP pool you just created as the DHCP resource for this interface.
    
        R1(config)#interface g0/0/1
        R1(config-if)#ipv6 nd other-config-flag 
        R1(config-if)#ipv6 dhcp server R1-STATELESS
    
    c. Saved the running configuration to the startup configuration file.
    d. Restarted PC-A.
    e. Examined the output of **ipconfig /all** and notice the changes.

        C:\>ipconfig /all

        FastEthernet0 Connection:(default port)

            Connection-specific DNS Suffix..: 
            Physical Address................: 0009.7CD7.572B
            Link-local IPv6 Address.........: FE80::209:7CFF:FED7:572B
            IPv6 Address....................: ::
            IPv4 Address....................: 0.0.0.0
            Subnet Mask.....................: 0.0.0.0
            Default Gateway.................: FE80::1
                                                0.0.0.0
            DHCP Servers....................: 0.0.0.0
            DHCPv6 IAID.....................: 
            DHCPv6 Client DUID..............: 00-01-00-01-91-17-47-67-00-09-7C-D7-57-2B
            DNS Servers.....................: 2001:DB8:ACAD::254
                                                0.0.0.0

    f. Tested connectivity by pinging R2’s G0/0/1 interface IP address.

        C:\>ping 2001:db8:acad:3::1

        Pinging 2001:db8:acad:3::1 with 32 bytes of data:

        Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
        Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
        Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
        Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254

        Ping statistics for 2001:DB8:ACAD:3::1:
            Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
        Approximate round trip times in milli-seconds:
            Minimum = 0ms, Maximum = 0ms, Average = 0ms

#### Part 4: Configure a stateful DHCPv6 server on R1

    a. Created a DHCPv6 pool on R1 for the 2001:db8:acad:3:aaaa::/80 network. This will provide addresses to the LAN connected to interface G0/0/1 on R2. As a part of the pool, set the DNS server to 2001:db8:acad::254, and set the domain name to STATEFUL.com.

        R1(config)# ipv6 dhcp pool R2-STATEFUL
        R1(config-dhcp)# address prefix 2001:db8:acad:3:aaa::/80
        R1(config-dhcp)# dns-server 2001:db8:acad::254
        R1(config-dhcp)# domain-name STATEFUL.com

    b. Assigned the DHCPv6 pool you just created to interface g0/0/0 on R1.
    
        R1(config)# interface g0/0/0
        R1(config-if)# ipv6 dhcp server R2-STATEFUL

#### Part 5: Configure and verify DHCPv6 relay on R2

##### Step 1: Power on PC-B and examine the SLAAC address that it generates

        C:\>ipconfig /all

        FastEthernet0 Connection:(default port)

            Connection-specific DNS Suffix..: 
            Physical Address................: 000A.4104.A936
            Link-local IPv6 Address.........: FE80::20A:41FF:FE04:A936
            IPv6 Address....................: 2001:DB8:ACAD:3:20A:41FF:FE04:A936
            IPv4 Address....................: 0.0.0.0
            Subnet Mask.....................: 0.0.0.0
            Default Gateway.................: FE80::1
                                                0.0.0.0
            DHCP Servers....................: 0.0.0.0
            DHCPv6 IAID.....................: 
            DHCPv6 Client DUID..............: 00-01-00-01-B4-07-55-AE-00-0A-41-04-A9-36
            DNS Servers.....................: ::
                                                0.0.0.0

    Noticed in the output that the prefix used is **2001:db8:acad:3::**

##### Step 2: Configure R2 as a DHCP relay agent for the LAN on G0/0/1

    a. Configured the **managed-config-flag** command on R2 interface G0/0/1. It was not possible to configure the command **ipv6 dhcp relay** by specifying the destination address of the interface G0/0/0 on R1.

        R2(config)#int g0/0/1
        R2(config-if)#ipv6 nd managed-config-flag 
        R2(config-if)#ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0
                                ^
        % Invalid input detected at '^' marker.
       
##### Step 3: Attempt to acquire an IPv6 address from DHCPv6 on PC-B

    a. Restarted PC-B.
    b. Opened a command prompt on PC-B and issued the command ipconfig /all and examine the output to see the results of the DHCPv6 relay operation.

    C:\>ipconfig /all

    FastEthernet0 Connection:(default port)

        Connection-specific DNS Suffix..: 
        Physical Address................: 000A.4104.A936
        Link-local IPv6 Address.........: FE80::20A:41FF:FE04:A936
        IPv6 Address....................: ::
        IPv4 Address....................: 0.0.0.0
        Subnet Mask.....................: 0.0.0.0
        Default Gateway.................: FE80::1
                                            0.0.0.0
        DHCP Servers....................: 0.0.0.0
        DHCPv6 IAID.....................: 1438320122
        DHCPv6 Client DUID..............: 00-01-00-01-B4-07-55-AE-00-0A-41-04-A9-36
        DNS Servers.....................: ::
                                            0.0.0.0
    
    I didn't see the addresses and DNS servers, as expected. I believe this is due to the limitations of Cisco Packet Tracer.

    c. Test connectivity by pinging R1’s G0/0/1 interface IP address.

    I rolled back the settings of the R2 router and checked the connection using the ping of the IP address of the interface R1 G0/0/1.

        C:\>ping 2001:db8:acad:1::1

        Pinging 2001:db8:acad:1::1 with 32 bytes of data:

        Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
        Reply from 2001:DB8:ACAD:1::1: bytes=32 time=12ms TTL=254
        Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
        Reply from 2001:DB8:ACAD:1::1: bytes=32 time=1ms TTL=254

        Ping statistics for 2001:DB8:ACAD:1::1:
            Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
        Approximate round trip times in milli-seconds:
            Minimum = 0ms, Maximum = 12ms, Average = 3ms
