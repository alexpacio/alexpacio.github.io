.. title: Embracing the IPv6 Revolution: A Homelab Story
.. slug: embracing-the-ipv6-revolution-a-homelab-story
.. date: 2025-05-30 10:00:00 UTC+00:00
.. tags: ipv6, homelab, nas, networking, cgnat, Docker, openmediavault, internet
.. category: Networking
.. description: A personal experience setting up a home NAS and discovering the readiness of IPv6, even behind CGNAT, and its implications for a more democratic internet.

The other day, I was contacted by a friend of mine asking for help setting up a home NAS (aka Network Attached Storage) based on OpenMediaVault. While he's an enthusiast, he wasn't able to properly configure all the technical aspects of the setup.

The NAS world is increasingly resembling a homelab rather than just file storage. Using Docker, you can get enterprise-grade software and setups running with quite simple steps.

After setting up the Docker containers, we faced the usual issue that most homelabs encounter these days: his internet connection was behind a CGNAT for IPv4 networking. However, I was pleasantly surprised to find an IPv6 public subnet already configured!

.. TEASER_END

Since his intent was to use the NAS via LAN or his mobile phone (and those of his family/relatives), we investigated and figured out that various Italian 4G/5G carriers were providing IPv6 connectivity. So, we decided to give it a try.

I am surprised that IPv6 is still such an unexplored world, even though it's already here and many of us haven't even noticed. Carriers are starting to support it seriously, and the IPv4 scarcity situation is getting worse as time goes by.

One of the most interesting things about using IPv6 is that it truly enables *peer-to-peer* communication, as you no longer need inbound or outbound IP address translations (NAT) to communicate with others.

IPv6 is poised to be one of the biggest *silent* revolutions in the Internet/networking field. While it might not offer apparently groundbreaking innovations to the general public, it will enable us to be more interconnected.

The Internet is meant to be a democratic platform. However, if users are masked or filtered by CGNAT, they can never truly use the Internet actively. They become passive actors on the scene, mere consumers, or producers reliant on someone else's services or infrastructure. IPv6 enables a more democratic approach, where we can connect all of our devices, even personal ones, directly to the web and interact with others' devices without middlemen.

If you have a homelab, give IPv6 a try! Be prepared for the revolution and the major move to IPv6, which might happen sooner rather than later—even if you'd given up waiting for it!