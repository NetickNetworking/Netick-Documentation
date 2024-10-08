# Port Forward

Port Forward is a technology to allow your machine/resource/instance to be accessed by the internet (WAN). However, several Internet Service Providers (ISP) block these facilitation to prevent security issues by default. Hence, the game server won't be joinable by other players.

Some games also doesn't have a built-in Relay SDK to bypass the port forwarding (NAT Blocking) such as:
- Minecraft
- Terraria
- Valheim

These are the popular solutions to bypass NAT Blocking issues

### 1. Cloud service provider

Host the game server in a cloud service provider with a public IP such as:
- Amazon Web Services (AWS)
- Google cloud
- Azure

### 2. Relay SDK

Relay is a "bridge" to facilitate a connection from a server to a client. However there will be extra latency to be added.

Popular Relay Services:
- Steamworks
- Epic Online Services (EOS)
- Unity Relay

### 3. Reverse Proxy Service

Reverse Proxy is a service to be ran on the server machine independently with the game server to expose It's public IP, similar how Relay SDK works.

- Ngrok (TCP Only)
- Playit.gg
- Hamachi

### 4. Internet Service Provider

If your NAT is blocked by your ISP, try to contact with them. However there might some extra charge in order to accomplish it.

### Solutions Comparison

| Solutions       | Pros                      | Cons                                                          |
|-----------------|---------------------------|---------------------------------------------------------------|
| Cloud           | Lowest Latency            | High cost<br>Less flexible                                    |
| Relay SDK       | Automatic                 | More latency<br>Require a certain transport                   |
| Reverse Proxy   | Cost efficient            | More latency<br>Addition setup required by players            |
| ISP: Public IP  | Applicable to other games | Might have extra cost<br>Additional setup required by players |