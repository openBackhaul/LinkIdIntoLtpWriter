
# Implementation - hygiene queue

The hygiene queue, which is executing the actual updating of LTPs on physical devices, should be maintained
over the elastic search instance. In case the application needs to reboot, the queue information will be retained.

The list of known links will be iterated by the hygiene queue indefinitely at configurable intervals. This list will expand over time.
The hygiene queue must always be running to continuously update information on the devices, because the information on the devices
about link-ids could get lost.

The number of links that can be processed in parallel by the hygiene queue can be configured.
If handling of the queue should be accelerated, the wait-time between cycles can be changed by configuration (profile) at runtime.

# Identifying the correct directional radio link

1) Check if link-utilization is at a maximum of 30%
2) Check if both endpoints use the same frequency
3) Switch off Adaptive Modulation
4) Set end A + B to Min Mod and measure both sides
5) Set end A + B to Max Mod and measure both sides
6) Check if the current transmission-mode changed accordingly - if yes it is a linked LTP pair
7) Change all changed settings to original plan values and turn adaptive modulation on

The link-utilization is determined by a measurement of the bits IN/OUT and compared against plan data.

For read access MWDI (in live mode) is used, for write access MWDG is used.

# About input data
Input data can be derived from APT link data. For example LINK_END from KAP_LINK is B side mount.