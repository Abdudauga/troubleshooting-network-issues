# troubleshooting-network-issues


Goal is to troubleshoot the network. 
<img width="1265" height="669" alt="{85F02D3C-A1BD-4AE1-A30D-0D07E559F278}" src="https://github.com/user-attachments/assets/b7c9f240-96af-440a-a367-30d50aca2bd2" />

Questions I asked and their answers: 

1. Have all routers established an OSPF peerings?
   Answer: Run show ip ospf neighbors.
   R1: <img width="948" height="116" alt="{7CF5B56A-0E7D-4892-9CE3-E828F233608F}" src="https://github.com/user-attachments/assets/cdb1d95f-04b5-47d8-9296-fb25adae0181" />
   R2: <img width="915" height="138" alt="{C7E76CCC-58A4-4C6B-870A-D27A51EE7045}" src="https://github.com/user-attachments/assets/6ee0145c-4ae2-4ea2-8d8e-b2f1b60075ea" />
   R3: ![Uploading {1EB81AE3-1577-4D15-B9F8-DEA806307DE3}.png…]()

   What this tells me: R3 is the designated router because it has two neighbors who are in the FULL/BDR stage, where the others don't. This also tells me the that the BDR hasn't established a FULL/BDR state with the non-BDR router. 



