---
title: 'Network troubleshooting for Azure Functions'
date: 2023-07-03T12:59:15+01:00
draft: false
description: 'This post explains the steps required to troubleshoot networking issues with Azure Functions on the Microsoft Azure Cloud Platform'
thumbnail: '/images/azure-functions.png'
---

#### **How to troubleshoot Networking in Azure Functions**

Geez, what an experience this has been!
It all started with an .NET azure function not being able to login into an on-premises SQL Server due to TLS negotiation. Now what?

In order to understand if the function is able to reach the on-premises server, I learned that we can get into the running container by ssh/bash whilst using the Kudu service.
Kudu is the engine behind a number of features in Azure App Service . In order to access Kudu, we need to inject the word "scm" between the app-name and the azurewebsites.net
For eg: https://app-name.**scm**.azurewebsites.net.
There´s also an easier way to get to Kudu by visiting the Azure Function blade and then Development Tools -> Advanced Tools.

![Azure Kudu Service](/images/20230703kudu-azurefunctions.png)

Once I was in the SSH shell, I realized the hard that PING was not available. Apparently this a measure to prevent DDos attacks. Luckily I found an article explaining that theres another tooling available for pinging over TCP, say [hello to tcpping](https://www.elifulkerson.com/projects/tcping.php).

In order for us to be able to use tcpping, we need to install first with the following steps:
{{% notice info "Getting tcpping available" %}}

1. apt-get update
2. apt install tcptraceroute
3. cd /usr/bin
4. wget http://www.vdberg.org/~richard/tcpping
5. chmod 755 tcpping
6. apt instal bc
   {{% /notice %}}

Now we are good to run a ping over the port 1433 like such:
{{% notice info "Run the following commands in your ssh shell?" %}}
tcpping **site.domain.com** 1433
{{% /notice %}}
And then I was able to get something like this:
![tcpping example](/images/20230307-tcpping.png)

#### **Okay, we can reach the target but still no luck, what now?**

Network capture for the rescue!

In order to capture a network capture, we need to get the tcpdump tooling installed on the ssh shell.
For that, follow the following steps
{{% notice info "Getting tcpdump available" %}}

1. apt-get update
2. apt install tcpdump
   {{% /notice %}}
   {{% notice warning "Attention!" %}}
   All of these packages being installed are not persistent. If the function is stopped/restarted, they will be deleted.
   {{% /notice %}}

   Now that we have tcpdump installed, we can use it to create a **pcap** file that we can then analyze it with Wireshark.

{{% notice info "Capturing a pcap file with tcpdump" %}}

1. List all the network interfaces available
   tcpdump -D
2. Usually we will be after the network interface starting with vnet\*
   tcpdump **-i** vnet**xxxx** **-w** output_file_name.pcap
   {{% /notice %}}

Once I was done with my test scenario, I stopped the trace with Control + C and the file got generated.

Now, the interesting part, how do you extract the file into your machine?
This might sound a bit lame, but the way I did was by moving the file into the home directory and then trying to reach the file through the API as that would start the download of the file, as such:
{{% notice info "Downloading the file to my machine" %}}

1. mv output_file_name /home
2. Visit https://app-name.scm.azurewebsites.net/api/vfs/output-file-name.pcap
   {{% /notice %}}

   Voila, we can now open the \*.pcap file with Wireshark

For future reference, Microsoft has an amazing helper on how to troubleshoot vnet integration [here](https://learn.microsoft.com/en-us/troubleshoot/azure/app-service/troubleshoot-vnet-integration-apps)
