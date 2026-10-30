# troubleshooting-network-issues


Goal is to troubleshoot the network. 
<img width="1265" height="669" alt="{85F02D3C-A1BD-4AE1-A30D-0D07E559F278}" src="https://github.com/user-attachments/assets/b7c9f240-96af-440a-a367-30d50aca2bd2" />

Questions I asked and their answers: 

1. Have all routers established an OSPF peerings?
   Answer: Run show ip ospf neighbors.
   R1: <img width="948" height="116" alt="{7CF5B56A-0E7D-4892-9CE3-E828F233608F}" src="https://github.com/user-attachments/assets/cdb1d95f-04b5-47d8-9296-fb25adae0181" />
   R2: <img width="915" height="138" alt="{C7E76CCC-58A4-4C6B-870A-D27A51EE7045}" src="https://github.com/user-attachments/assets/6ee0145c-4ae2-4ea2-8d8e-b2f1b60075ea" />
   R3: ![Uploading {1EB81AE3-1577-4D15-B9F8-DEA806307DE3}.png…]()

R2 to R1: R2 sees that R1 is stuck int he exstart/bdr state. Let's troubleshoot and fix:
1. Both interface are in the correct subnet and the correct area number.
2. Issue the show ip int command to see if the MTU matches or not.
     Note: the Cisco OSPF troubleshooting guide tells us that if a router is in the Exchange/Exstart state, it most likely means that the issue is either layer 2 or MTU related.
   Show ip int command results:
   R1: <img width="499" height="115" alt="{F8EBF08F-43F0-42A7-BD2B-676D7C3AD094}" src="https://github.com/user-attachments/assets/df673de9-0fe3-4365-8d22-d144fbb8c72f" /> MTU IS 1500. 

   R2: <img width="539" height="140" alt="{FE2ADAD4-E424-4FE8-BB17-3B9569A7FF27}" src="https://github.com/user-attachments/assets/999913fe-8811-4795-92d2-49325a566dfd" /> MTU IS 1476. 

Culprit: MISMATCHED MTUs. '
The fix: 
<img width="774" height="381" alt="{98D57418-00BF-42BC-8017-B15441EC96DA}" src="https://github.com/user-attachments/assets/7bc2468d-0916-42ce-a694-aec0a92c86cb" /> 

Let's check again: 
R1: <img width="957" height="109" alt="{FFD1B29B-5FF8-401F-A52E-318DD77DD0E0}" src="https://github.com/user-attachments/assets/2a3a9647-1f10-4249-9d2a-838a7705f6bf" /> Full state achieved! 


Now that the devices have established a full state, let's see if the routes look correct or not. Using the info we have from the topology, if PC1 wants to communicate to SRV, it should go from R2 to R1, then R1 to R3 since this route has the fastest links. 

Let's check R2 routing table: 

<img width="757" height="351" alt="{B53A1155-0F03-4BEC-932C-46506F8EECF7}" src="https://github.com/user-attachments/assets/35ea8b79-a43e-470e-8ba0-f7b7e1d41051" />

ISSUE 1: R2 can reach R3 via two routes
1.  O        198.51.100.0 [110/2] via 203.0.113.2, 00:05:26, GigabitEthernet0/3
                      [110/2] via 192.0.2.1, 00:05:26, GigabitEthernet0/2
    The issue here is that it is treating both routes as having an equal cost of **2**. However, when looking at interface bandwith on the topology design, you can see that R2 to R3 ( 198.51.100.0 [110/2] via 203.0.113.2) has an interface speed of 100 Mbps. The other path, R2 to R1 then R1 to R3, has interface bandwith of 1 Gbps per router segment. The R2 to R3 via R1 route ([110/2] via 192.0.2.1, 00:05:26, GigabitEthernet0/2) is the faster route but OSPF is treating them the same.
    Why: OSPF calculates it's **cost** by the following formula:
   Reference Bandwith / Interface Bandwith. OSPF has a default reference badnwith of **100 Mbps**. The issue with that is any interface speed of **100 Mbps or Higher** will equal **1** because OSPF round up any number less than 1 to 1.

Solution: change the reference bandwith to something that would account for higher interface bandwiths like **10 Gbps**. 
R2: <img width="878" height="241" alt="{5F1C8E2F-8A1C-4629-91AC-4F73025C7526}" src="https://github.com/user-attachments/assets/023a3ae3-c7ae-40ea-9987-08889e976857" />





ISSUE DETECTED: R2 does NOT see 10.1.2.0/24 Network. 
   First theory: The cause of the issue is most likely auto-summarization.
   LESSON: auto-summarization can't be configured on OSPF. 
   
   
   Second theory: R3 is not advertising 10.1.2.0. 
   Two ways to check: show ip route on R1 <img width="809" height="313" alt="{E1503A42-BD37-4F2D-B10C-A7B144A69BDD}" src="https://github.com/user-attachments/assets/dd9163a6-0c75-4bb8-88e2-067ddc45ddab" /> 

  R3#sh running-config | section ospf 
   <img width="479" height="158" alt="{E6F364F1-1F00-41B6-93B2-883CD2180BAC}" src="https://github.com/user-attachments/assets/33ca33c2-f2df-46a9-b715-321804f61689" />

According to R3, it **is** advertising the route, but how come other routers don't see it? 


   












