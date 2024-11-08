+++
title = "BebasIT"
date = 2024-03-11
draft = true
+++

<!-- [BebasIT](https://github.com/bebasid/bebasit) implementation on -->
<!-- [Asuswrt-Merlin](https://www.asuswrt-merlin.net/). Based on [BebasIT OpenWRT -->
<!-- tutorial](https://github.com/bebasid/bebasit/blob/master/docs/openwrt-tutorial.md). -->

## Motivation

<!-- TODO: motivation -->

After studying in Malaysia for my undergraduate, I cannot live without Reddit
anymore. It's like the oasis of knowledge where you could find literally
everything and interact with communities you like.

For the technical things you may learn more [here](https://github.com/bebasid/bebasdns/blob/main/dev/readme/learnmore.en.md).

## Prerequisite

1. Install Entware. Refer to [RMerl/asuswrt-merlin.ng
   wiki](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Entware). A USB drive
   is required for Entware installation.

2. Make sure `iptables git git-http` and a text editor of your choice are
   installed.

   ```sh
   opkg install iptables git git-http vim
   ```

## Step

<!-- TODO: -->

### Setup BebasID hosts

1. Navigate to Administration -> System -> Enable
   JFFS custom scripts and configs. Reboot the router.

   <!-- TODO: include image here -->

2. SSH into the router. Create a new bash script to fetch BebasID hosts file.

   Filename: `/jffs/scripts/hosts.add.sh`

   ```sh
   #!/bin/sh

   echo "" > /jffs/configs/hosts.add
   echo "# ------------------------------------------------------- #" >> /jffs/configs/hosts.add
   echo "# -------------------- BEBASID HOSTS -------------------- #" >> /jffs/configs/hosts.add 
   echo "# ------------------------------------------------------- #" >> /jffs/configs/hosts.add
   echo "" >> /jffs/configs/hosts.add

   curl -fsL https://raw.githubusercontent.com/bebasid/bebasid/master/releases/hosts |\
           sed 's/\r$//' >> /jffs/configs/hosts.add

   echo "" >> /jffs/configs/hosts.add
   echo "# Last generated on $(date)" >> /jffs/configs/hosts.add
   ```

3. Add execute permission for the script.

   ```sh
   chmod +x /jffs/scripts/hosts.add.sh
   ```

4. Try to run the script.

   ```sh
   /jffs/scripts/hosts.add.sh
   ```

   If success, `/jffs/configs/hosts.add` should be populated with BebasID hosts.

   ```sh
   cat /jffs/configs/hosts.add
   ```

5. Create a cron job that will execute the script periodically. Put the command
   in `init-start` script which will be executed after the JFFS has been
   mounted.

   Filename: `/jffs/scripts/init-start`

   ```sh
   #!/bin/sh
   cru a hosts.add "0 */12 * * * /jffs/scripts/hosts.add.sh"
   ```

6. Reboot the router to apply the custom script. Upon turning on, BebasID
   hosts should be applied. Check the contents of `/etc/hosts`.

   ```sh
   cat /etc/hosts
   ```

   Then check by querying to `lamanlabuh.aduankonten.id`.

   ```sh
   nslookup lamanlabuh.aduankonten.id
   ```

   It should shows `Server: 127.0.0.1` if it's working correctly.

   ```txt
   ax56u@RT-AX56U-F1C0:/tmp/home/root# nslookup lamanlabuh.aduankonten.id
   Server:    127.0.0.1
   Address 1: 127.0.0.1 localhost.localdomain

   Name:      lamanlabuh.aduankonten.id
   Address 1: 147.139.211.126 lamanlabuh.aduankonten.id
   ```

### Setup zapret

1. Clone zapret repo into `/opt` and navigate into it.

   ```sh
   git clone https://github.com/bol-van/zapret.git /opt/zapret
   cd /opt/zapret
   ```

2. Run zapret easy installation script.

   ```sh
   ./install-easy.sh
   ```

   For firewall type, choose `iptables`.

   For ipv6 support, choose according to your use case.

   For mode, choose `nfqws`.

   For HTTP and HTTPS support, choose `Y`.

   Press enter until finish.

3. Fine-tune zapret configuration by testing against blocked site.

   ```sh
   ./blockpage.sh
   ```

   For domain, choose one of blocked site (e.g. `reddit.com`, `vimeo.com`,
   `omegle.com`, etc).

   For ip protocol version, choose according to your use case.

   Press enter until `how many times to repeat each test (default: 1)`. Choose
   `2`.

   For `do all test despite of result?`, choose `Y`.

   Save the result summary.

4. Update zapret config. Look for the following lines.

   Filename: `/opt/zapret/config`

   ```sh
   ...
   #NFQWS_OPT_DESYNC_HTTP=
   #NFQWS_OPT_DESYNC_HTTPS=
   #NFQWS_OPT_DESYNC_HTTP6=
   #NFQWS_OPT_DESYNC_HTTPS6=
   ...
   ```

   Uncomment and put the argument for `nfqws` based on the result of previous step.

5. As Asuswrt-Merlin has unusual (but simple) init, we need to start zapret
   after firewall has been started.

   Filename: `/jffs/scripts/firewall-start`

   ```sh
   #!/bin/sh
   /opt/zapret/init.d/sysv/zapret start
   ```

   Note that `firewall-start` will still be executed even if the firewall is
   disabled in the router dashboard.

6. Restart zapret.

   ```sh
   /opt/zapret/init.d/sysv/zapret restart
   ```

7. Test by `dig`-ing one of blocked site.

   ```sh
   dig reddit.com
   ```

   The response should be similar to the following. Note the answer section and
   server.

   ```txt
   ax56u@RT-AX56U-F1C0:/tmp/home/root# dig reddit.com

   ; <<>> DiG 9.18.16 <<>> reddit.com
   ;; global options: +cmd
   ;; Got answer:
   ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17776
   ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

   ;; OPT PSEUDOSECTION:
   ; EDNS: version: 0, flags:; udp: 1232
   ;; QUESTION SECTION:
   ;reddit.com.                    IN      A

   ;; ANSWER SECTION:
   reddit.com.             0       IN      A       151.101.193.140

   ;; Query time: 1 msec
   ;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
   ;; WHEN: Tue Feb 06 19:40:33 ICT 2024
   ;; MSG SIZE  rcvd: 55
   ```

### (Bonus) Setup ntpMerlin

It's highly recommended to setup ntpMerlin to make sure the router is always on
sync. NTP is very critical as it could cause unexpected error for other tools.

1. Entware

## Reference

- https://github.com/bebasid/bebasit/blob/master/docs/openwrt-tutorial.md
- https://github.com/RMerl/asuswrt-merlin.ng/wiki/Entware

```txt
ax56u@RT-AX56U-F1C0:/tmp/home/root# ls -la /jffs/scripts/
drwxr-xr-x    2 ax56u    root             0 Oct 11 21:36 .
drwxr-xr-x   14 ax56u    root             0 Feb  3 12:14 ..
-rwxr-xr-x    1 ax56u    root            73 Sep 18 17:05 dnsmasq.postconf
-rwxr-xr-x    1 ax56u    root            47 Sep 11 12:35 firewall-start
-rwxrwxrwx    1 ax56u    root           117 Oct 11 21:36 hosts.add.sh
-rwxrwxrwx    1 ax56u    root            10 Sep 18 17:05 init-start
-rwxr-xr-x    1 ax56u    root         66852 Sep 11 07:01 ntpmerlin
-rwxr-xr-x    1 ax56u    root           179 Sep 11 07:01 post-mount
-rwxr-xr-x    1 ax56u    root           220 Sep 18 17:05 service-event
-rwxrwxrwx    1 ax56u    root            65 Oct  1 22:04 services-start
-rwxr-xr-x    1 ax56u    root            63 Sep 10 19:37 services-stop
-rwxr-xr-x    1 ax56u    root           209 Sep 11 08:34 unmount
ax56u@RT-AX56U-F1C0:/tmp/home/root# cat /jffs/scripts/services-start
#!/bin/sh
cru a hosts.add "0 * * * * /jffs/scripts/hosts.add.sh"
ax56u@RT-AX56U-F1C0:/tmp/home/root# cat /jffs/scripts/firewall-start
#!/bin/sh
/opt/zapret/init.d/sysv/zapret start
```
