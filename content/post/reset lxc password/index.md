---
title : "Reset Lxc Password"
date : "2025-07-31"
image : ""
tags : ["linux", "password", "lxc", "proxmox"]
categories : [contenedores]
description : ""
---


## How to Reset Your Proxmox LXC Container Root Password

1. Grab the container ID
    
    First you need to grab the **container ID**, which is the number next to the container name in the Proxmox node. For example, if the container name is “monitor”, the ID will be the number next to it.
    
2. Open up a shell
    
    Next, open up a shell (as root) into the container from the Proxmox host. You can do this by running the command **“lxc-attach -n [container_id]”**, with the appropriate container ID in place of the placeholder.
    
    For example, if the container ID is 150, the command would be:
    
    ```sh
    lxc-attach -n 150
    ```
    

- Change your password
    
    Finally, change your password by running the command **“passwd”**. This will prompt you to enter a new password for your container.
    
    ```sh
    passwd
    ```
    

