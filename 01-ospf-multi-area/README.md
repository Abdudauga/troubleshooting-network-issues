# Troubleshooting Network Issues

**Goal:** Troubleshoot Multi-Area OSPF Network. 

**Lab Source**: Kevin Wallace via YouTube Video titled "
Surprise Training! Troubleshooting Routing Protocols" 

<img width="1265" height="669" alt="{85F02D3C-A1BD-4AE1-A30D-0D07E559F278}" src="https://github.com/user-attachments/assets/b7c9f240-96af-440a-a367-30d50aca2bd2" />

---

## Step 1: Check OSPF Peerings

**Question:** Have all routers established OSPF peerings?

**Answer:** Run `show ip ospf neighbor` on each router.

**R1:**

<img width="948" height="116" alt="{7CF5B56A-0E7D-4892-9CE3-E828F233608F}" src="https://github.com/user-attachments/assets/cdb1d95f-04b5-47d8-9296-fb25adae0181" />

**R2:**

<img width="915" height="138" alt="{C7E76CCC-58A4-4C6B-870A-D27A51EE7045}" src="https://github.com/user-attachments/assets/6ee0145c-4ae2-4ea2-8d8e-b2f1b60075ea" />

**R3:**

<img width="871" height="130" alt="{DFFE60D8-5774-420B-8147-DFA691361E8B}" src="https://github.com/user-attachments/assets/9f48ad12-15a1-4200-aaeb-04991080314d" />


---

## Issue 1: R2 to R1 Stuck in EXSTART

R2 shows that R1 is stuck in the **EXSTART/BDR** state. Let's troubleshoot and fix it:

1. Both interfaces are in the correct subnet and the correct area number.
2. Run `show ip interface` on both routers to see if the MTUs match.

> **Note:** The Cisco OSPF troubleshooting guide says that if a router is stuck in the EXCHANGE/EXSTART state, the issue is most likely an MTU mismatch.

**`show ip interface` results:**

**R1:** MTU is **1500**

<img width="499" height="115" alt="{F8EBF08F-43F0-42A7-BD2B-676D7C3AD094}" src="https://github.com/user-attachments/assets/df673de9-0fe3-4365-8d22-d144fbb8c72f" />

**R2:** MTU is **1476**

<img width="539" height="140" alt="{FE2ADAD4-E424-4FE8-BB17-3B9569A7FF27}" src="https://github.com/user-attachments/assets/999913fe-8811-4795-92d2-49325a566dfd" />

**Culprit:** Mismatched MTUs.

**The Fix:**

<img width="774" height="381" alt="{98D57418-00BF-42BC-8017-B15441EC96DA}" src="https://github.com/user-attachments/assets/7bc2468d-0916-42ce-a694-aec0a92c86cb" />

**Let's check again:**

**R1:**

<img width="957" height="109" alt="{FFD1B29B-5FF8-401F-A52E-318DD77DD0E0}" src="https://github.com/user-attachments/assets/2a3a9647-1f10-4249-9d2a-838a7705f6bf" />

**FULL state achieved!**

---

## Issue 2: OSPF Treats Unequal Links as Equal Cost

Now that the devices have reached the FULL state, let's see if the routes look correct. Based on the topology, if the PC wants to communicate with the Server, traffic should go from **R2 to R1**, then **R1 to R3**, since this path has the fastest links.

**R2 routing table:**

<img width="757" height="351" alt="{B53A1155-0F03-4BEC-932C-46506F8EECF7}" src="https://github.com/user-attachments/assets/35ea8b79-a43e-470e-8ba0-f7b7e1d41051" />

R2 has two equal-cost routes to the R1–R3 network:

```
O    198.51.100.0 [110/2] via 203.0.113.2, 00:05:26, GigabitEthernet0/3
                  [110/2] via 192.0.2.1, 00:05:26, GigabitEthernet0/2
```

OSPF treats both routes as having an equal cost of **2**. However, the topology shows that the direct R2 to R3 link (`via 203.0.113.2`) is only **100 Mbps**. The other path, R2 to R1 and then R1 to R3 (`via 192.0.2.1`), uses **1 Gbps** links on both segments. The path through R1 is faster, but OSPF treats them the same.

**Why:** OSPF calculates its cost with this formula:

```
Cost = Reference Bandwidth / Interface Bandwidth
```

OSPF's default reference bandwidth is **100 Mbps**. Any interface at **100 Mbps or higher** gets a cost of **1**, because OSPF rounds any value below 1 up to 1.

**Solution:** Change the reference bandwidth to a value that accounts for faster interfaces, such as **100 Gbps**.

**Adjusting the cost on R2:**

<img width="878" height="241" alt="{5F1C8E2F-8A1C-4629-91AC-4F73025C7526}" src="https://github.com/user-attachments/assets/023a3ae3-c7ae-40ea-9987-08889e976857" />

**R2 routing table after adjusting auto-cost:**

<img width="737" height="302" alt="{99816DA8-7749-4102-8E2A-B1DCA6FDA387}" src="https://github.com/user-attachments/assets/5a1a247d-a92d-482d-b6b9-da6b76ef9694" />

---

## Issue 3: R2 Does NOT See the 10.1.2.0/24 Network

### First Theory: Auto-Summarization

**Lesson:** OSPF doesn't do auto-summarization. Summarization in OSPF is manual only.

### Second Theory: R3 Is Not Advertising 10.1.2.0/24

Two ways to check:

**1. `show ip route` on R1:**

<img width="809" height="313" alt="{E1503A42-BD37-4F2D-B10C-A7B144A69BDD}" src="https://github.com/user-attachments/assets/dd9163a6-0c75-4bb8-88e2-067ddc45ddab" />

**2. `show running-config | section ospf` on R3:**

<img width="479" height="158" alt="{E6F364F1-1F00-41B6-93B2-883CD2180BAC}" src="https://github.com/user-attachments/assets/33ca33c2-f2df-46a9-b715-321804f61689" />

According to R3, it **is** advertising the route. So why don't the other routers see it?

### Root Cause

Let's recall how OSPF works: **all inter-area advertising must go through Area 0.** The **10.1.2.0/24** network in Area 2 isn't showing up in R1's or R2's routing tables because R3 has no connection to Area 0, so it can't generate Type 3 LSAs.

### The Fix: Virtual Link

Add a virtual link to connect R3 to Area 0. On each router, point the virtual link to the other router's **router ID**.

**R2:**

<img width="506" height="77" alt="{8E6B9A6A-24EE-4987-B10A-2B46FA602B89}" src="https://github.com/user-attachments/assets/bf3e6307-e8b1-417a-9636-7c50589044b3" />

> **Note:** The area number in the `area 1 virtual-link 3.3.3.3` command has to be the **transit** area.

**R3:**

<img width="516" height="52" alt="{86EF4DB1-FD46-40F4-85A0-6C68630276F6}" src="https://github.com/user-attachments/assets/c517d755-fca3-4a12-87e3-046df0ceced2" />

**Let's see if R2 can finally see the 10.1.2.0/24 network:**

<img width="705" height="300" alt="{8C1398F7-D79D-4443-A697-8FEB2FD69856}" src="https://github.com/user-attachments/assets/0e5c9e6d-3d75-4617-a6d9-76b5ef32eba4" />

**It sure can!**

---

## Final Check: Traceroute from PC

Now that everything looks good, let's run a traceroute from the PC to see if traffic takes the shortest path.

**PC:**

<img width="694" height="181" alt="{EC5CB976-EEEB-480B-8194-5647366FFBA5}" src="https://github.com/user-attachments/assets/c6391b94-8c8b-48ee-a052-09d2b1446ca9" />

**Traffic is taking the shortest path!**
