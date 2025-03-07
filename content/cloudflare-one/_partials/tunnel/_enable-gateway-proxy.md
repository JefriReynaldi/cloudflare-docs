---
_build:
  publishResources: false
  render: never
  list: never
---

1. Go to **Settings** > **Network**.
2. Enable **Proxy** for TCP.
3. (Recommended) To proxy traffic to internal DNS resolvers, select **UDP**.
4. (Recommended) To proxy traffic for diagnostic tools such as `ping` and `traceroute`, select **ICMP**. You may also need to update your system to allow ICMP traffic through `cloudflared`:

{{<details header="Linux" open="false">}}
1. Ensure that `ping_group_range` includes the Group ID (GID) of the user running `cloudflared`.
    1. To get the Group ID of the user, run `id -g`.
    2. To verify the Group IDs that are allowed to use ICMP:

    ```sh
    $ sysctl net.ipv4.ping_group_range
    net.ipv4p.ping_group_range= 0 100000
    ```
    3.  Either add the user to a group within that range, or update the range to encompass a group the user is already in by modifying `/proc/sys/net/ipv4/ping_group_range`.

2. If you are running multiple network interfaces (for example, `eth0` and `eth1`), configure `cloudflared` to use the external Internet-facing interface:

    ```sh
    $ cloudflared tunnel run --icmpv4-src <IP of primary interface>
    ```
{{</details>}}

{{<details header="Docker" open="false">}}

In your environment, modify the `ping_group_range` parameter to include the Group ID (GID) of the user running `cloudflared`.

By default the [`cloudflared` Docker container](https://github.com/cloudflare/cloudflared/blob/master/Dockerfile#L29C6-L29C13) executes as a user called `nonroot` inside of the container. `nonroot` is a specific user that exists in the [base image](https://github.com/GoogleContainerTools/distroless/blob/859eeea1f9b3b7d59bdcd7e24a977f721e4a406c/base/base.bzl#L8) we use, and its Group ID is hardcoded to 65532.

{{</details>}}

Cloudflare will now proxy traffic from enrolled devices, except for the traffic excluded in your [split tunnel settings](/cloudflare-one/connections/connect-networks/private-net/cloudflared/#3-route-private-network-ips-through-warp). For more information on how Gateway forwards traffic, refer to [Gateway proxy](/cloudflare-one/policies/gateway/proxy/).