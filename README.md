This script is a helper for rotary axis alignment.

### The problem
it's really hard to properly align a rotary axis on a CNC bed. It's not that important in case you using angular axis as a replacement of linear axis and "ufolding" linear trajectory onto a round object, but if you trying to use it in a real full 4 axis mode you'll have quite a bad results if you try to roll over your object and process it in two directions e.g. for 0.1 degree misalignment you will get a 0.43mm bead within ~150mm.


* Step 1 - do a cut on one side:
![step1](https://user-images.githubusercontent.com/3116070/132454090-87c5d1e2-6e88-4fb7-a312-e1b9de4506a6.png)

* Step 2 - roll over and do a through cut on the other side with the same XZ coordinates and mirrored Y
![step2](https://user-images.githubusercontent.com/3116070/132454857-f41542a8-f29b-463b-893b-9d94a937b413.png)

* Step3 - due to bad alignment of rotary axis you will get a notable bead
![step3](https://user-images.githubusercontent.com/3116070/132455104-175ff562-6917-4ec2-807c-8280f51c5ec1.png)




### Proposed solution
In order to get rid of this problem the rotary axis has to be well aligned(what a surpise). That is somewhat painfull and time consuming process. In order to automate it i'm proposing conductive probe and this script.
It's intended to find the accurate rotary axis position on a bed by doing two series of probes in the beginning of a testbar and at the end of the testbar. This two series of probes will find two right-most points and will calculate offset angle in XY plane and YZ planes and will produce proper `G68 coordinate rotation commands` to softwarely compensate for the offset. Of course the test bar has to be calibrated, properly secured in your chuck and tailstock and electrically insulated from your probe.

In order to probe you will need a conductive mechanical probe that can tolerate overrun(`XProbeIncrements`). Typically a ball in a tube held by a spring. Before running this script probe has to be located right(X++) from the calibrated stock in a distance not more than `MaxXProbe` in a position that collision between ball and stock happen in a lower part.

![Positioning1](https://user-images.githubusercontent.com/3116070/132311769-a984b3d3-2e47-4d64-a4aa-393cd9cc5235.png)
![Positioning](https://user-images.githubusercontent.com/3116070/132604303-a7b7323d-ac83-4636-89d7-47adba14c33b.png)

The second probe will be located in `BarL` distance along Y axis. Keep in mind that move to the second probe will be straight G1 move, watch for collisions.

![image](https://user-images.githubusercontent.com/3116070/132318167-3f63a02a-959f-4621-866b-9d2f1857fb45.png)


### Assumptions
The probe start position is X++ and Y-- from the bar as shown on picture. Probably you can make if work by playing with negative coefficients, but never tested this.

This scripts is designed for XHC MKX-IV https://www.nvcnc.net/mkx-iv-xhc-cnc-controller.html controller or similar (practically any chineese mach3 controller).
That is also the explanation for weird programmatic implementation of G31. In a normal word people doing probes with G31 move, the software part controls step-dir generation and stop the movement at the best of his abilities according to deceleration settings as it's impossible to stop and physical object instantly. This lead a certain amount of probe overrun, which caused by:
- time to discover that probe engaged by software(mach3 running in userspace of multitasking Windows - far away from realtime)
- time to stop each axis engaged onto G31 move with a respect to slowest axis as during decelleration it has to keep the same vector


By design the hardware part supposed to keep track of the exact position where collision happened and this position supposed to be available via special `GetVar(2001)` call.
Apparently this designed seems too complex and "redundant" by standardz of designers of this kind of controllers and they proposing to use current position instead of `GetVar(2001)`. Some of them have `GetVar(2001)` implemented but it's a shortcut for current DRO and even may return gargabe after first probe. Without proper implementation i've got extremely poor repeatability even with feedrate of 5, so i had to implement this programmtic solution for G31.

Considering this flaw in design and poor repeatibility along Z axis using G31
![G31](https://user-images.githubusercontent.com/3116070/132485554-34a43510-ae9f-433f-880a-23d5592b724f.png)

vs relatively good with G1

![G1](https://user-images.githubusercontent.com/3116070/132485470-7dd914ea-5d0f-4ca4-9f5e-3f5cd5d1db3e.png)
(raw data https://github.com/tsipa/mach3-rotary-axis-probe/blob/main/G1vsG31 measured with 5.5mm ball probe on 30mm diameter bar)

I decided to use G1 and incremental probes.

