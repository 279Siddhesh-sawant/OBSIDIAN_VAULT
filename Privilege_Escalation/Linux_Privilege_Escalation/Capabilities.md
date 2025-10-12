```
# Login to the application using provided ssh credentials.
# Run followinng command to check how many binaries have set capabilities.
getcap -r / 2>/dev/null

# We found that 6 binaries have set capabilities.
getcap -r / 2>/dev/null
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/ping = cap_net_raw+ep
/home/karen/vim = cap_setuid+ep
/home/ubuntu/view = cap_setuid+ep

# Check GTFobins for help
# Run find command for flag as it doesnt requires root privilege.
find / -name flag*
```