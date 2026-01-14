.. title: Linux Desktop Influencers and the Standard Problem
.. slug: linux-desktop-influencers-and-the-standard-problem
.. date: 2026-01-13 12:00:00 UTC
.. tags: linux, desktop, gnulinux, opensource, kde, influencers
.. category: technology
.. author: Alessandro Bolletta
.. summary: A personal reflection on Linux desktop fragmentation, the role of influencers in guiding new users, and why concentrating efforts on a single reference solution is essential for mainstream adoption.

I hate those who always criticize. I appreciate instead those who *do*, and I consider myself to belong to the latter category.
However, recently my posts have been mostly criticisms rather than constructive articles, and for this, I apologize; I promise to release something constructive as well and to keep my promises.

Having made this brief *mea culpa*, I must clarify nonetheless that it is not easy to remain indifferent to certain events. I feel I can speak about them, having enough experience and a wide-ranging vision to do so. I do it for the good of everyone and, above all, to protect the precious work of many developers who allowed me to witness the evolution of a world that, although exclusively for hobbyists until 25 years ago, has managed to churn out solutions that today constitute the foundations of modern computing and are therefore a fundamental and indispensable asset for humanity.

.. TEASER_END

My Journey with Open Source
---------------------------

Let me try to give you some context on my thoughts and experiences. My first experiences with the open-source world date back to 2002. I was 11 years old. I remember very well a year when I went on vacation to a resort with my parents, and there I met one of the greatest disseminators of information on the open-source and Linux world of that time. He worked as an entertainer at the tourist village. I remember we hit it off immediately (he was practically a legend to me), and for the whole vacation, we would hole up in the tool shed tinkering with Slackware.

In the same period, my father, who owned an IT company and was a very open-minded person attentive to new developments, began to be captured by the Linux hype, which was starting to materialize thanks to the adoption and birth of the first commercial distros.
I carefully preserve the original boxes of RedHat 8.0, Suse Linux, and even Slackware.

At the time, GNU/Linux was like it is today: fragmented and colorful, although it had a soul, a common thread shared among its various branches. It was precisely that fragmentation that truly made the difference, the trigger that helped turn on the light of innovation and genius which shortly thereafter gave rise to numerous open-source software projects that made history and that we still use every day, directly or indirectly.

The Server World Found Its Path
-------------------------------

However, this happened mainly in the server realm. There, a precise path was found, thanks also to the intervention of Corporations which, moved by the practical goals of the enterprise world, managed with pragmatism to select the software in the basket. In some ways, this was negative, in the sense that corporations did not always adopt the best software (see systemd), but they were able to determine a standard, which is fundamental for those who use a computer.

A standard helps the computer user above all. Who among you has the desire and time to learn, try, and play with billions of different implementations of the same thing? Nobody.

In the desktop realm, this search for a standard, this cherry-picking, did not happen. Surely the interference of Microsoft and Apple played their part; for them, the desktop world was too rich a world to be left to the fate of open source.

I also remember the very first versions of Fedora and Ubuntu. I even keep the case and original CDs of the very first release that Ubuntu sent from South Africa to spread and evangelize Linux for free; you just had to register on a form and they shipped the copy for free! Madness.

I saw GNU/Linux emerge from the occult world of hobbyism to the professional world; through my father's business, I was already doing systems consulting on Linux servers at a very young age, and I followed its developments in the domestic/desktop scope as well.

The Present State
-----------------

Today GNU/Linux is the *de facto* standard for the server world. It no longer seems so absurd to hypothesize that one day Windows Server will become a mere Linux distro. Once, that was an idea that made one's skin crawl.

Due to the increasing closure/lock-in of proprietary OSs (Windows and Mac), today people would like to think, would like to lay the foundations to effect the same change in the desktop realm as well.

In the era of influencers and YouTube, where everything is easily newsworthy (see the videos found around that are released in real-time as soon as Linus Torvalds farts), we are witnessing the "birth" of certain characters, each with the aspiration of becoming the reference point, the media guru of their own niche.

A Critique of Linux Influencers
-------------------------------

One of these is the author of a channel called "Alternativa Linux". I leave the link to his most recent video here: https://www.youtube.com/watch?v=jCPc1eE3TVQ

This person absolutely deserves respect and esteem because, without sponsors and with a true and genuine spirit of initiative, he tries to passionately spread GNU/Linux precisely to the desktop world audience.

However, I must raise some technical criticisms, albeit constructive and respectful, because he risks nullifying the precious work of those who have worked behind the scenes for years to lay the foundations of this new "standard" for the desktop. I fear that, in the attempt to make a noble gesture, this gentleman unfortunately lacks context and also technical vision, and that this could contribute to influencing and guiding people towards absolutely losing choices.

Alternativa Linux strongly sponsors and supports the Linux Mint and Zorin OS projects for desktop environments. Aside from the fact that in his long videos, according to him, the most important aspects—the problems that these distros solve compared to Windows or MacOS—are essentially two: hardware obsolescence and the freedom to use whatever you want.

As if switching from one distro to another, from one Desktop Environment to another, were a little game. As if restoring a PC otherwise destined for the landfill were the ONLY points that matter, or at least the prevailing ones.

In my experience, however, the points that always determine the success and large-scale use of one solution over another often lie in **usability** and **long-term stability**.

The Problem with Popular Recommendations
----------------------------------------

Linux Mint with the Cinnamon DE, the latter being a project actively worked on by maybe 4 people (see the `commit history <https://github.com/linuxmint/cinnamon/commits/master/?after=0275da135572f727d3753491416d37fea7ac1bff+69>`_).

Zorin OS, a distro that uses a highly customized version of GNOME 3 and Ubuntu, the latter being a distro that has shown it has taken an inconsistent path.

Even in the videos where he talks about Linux Mint, he says: "choose which DE you like best between MATE and Cinnamon". As if choosing between these two DEs were the same thing, an interchangeable thing, choose the one you prefer – underneath it's always Linux anyway, right?

But have you ever asked yourselves why MacOS or Windows don't have a thousand DEs? Do you think there is an obstructionist motive on the part of the manufacturers? I really don't think so. I think simply that efforts should be concentrated on one project and that this should be made as customizable as possible.

The Only Serious Contender
--------------------------

The only DE that fully reflects this and has the credentials to become a reference point, as it effectively always was until its version 3, is **KDE**. Thanks also to the QT libraries, which are the only ones that have characteristics similar to their counterparts for Windows or Mac.

Not to mention tiling window managers, Hyprland, Sway, i3, etc. Those would make sense in an embedded context, sure, but not as a desktop.

Other influencers talk about Omarchy. And that is supposed to be a distro to entrust your work to? It is a horrible kludge and jumble of rubbish that will be deprecated in 2 years.

System76 has released yet another DE, COSMIC Desktop, written in Rust. Yet another clone of GNOME and company, but much barer.

The Fragmentation Nightmare
---------------------------

KDE Plasma, GNOME, XFCE, i3, Wayfire, LXQt, Openbox, Cinnamon, COSMIC, UKUI, LXDE, Mate, Budgie, Qtile, Hyprland, Sway, and Niri.

These are just some of the DEs actively used here and there by distros.

Imagine developing an application that runs perfectly and with optimal performance on all these DEs, with homogeneous and predictable behavior. A delirium.

Conclusion
----------

Ultimately, I can state that I have seen it all in the GNU/Linux field, and I can state with certainty that the only serious, true, and concrete solution for the desktop world is a distro that adopts a release style like Fedora and a DE like KDE Plasma.

Instead of dispersing energy and time across 20 different DEs (and I only cited some above, there are many other minor ones), in a world where proprietary OSs are making mistakes and thus giving us juicy assists to knock them out—but at the same time do not want to offer us, for one reason or another, the same help, the same push and enterprise-type adoption that was given in the server realm—why don't we rationalize our energies and concentrate them on a winning reference solution?

I am sorry to admit it, because I truly care, but until we take this step, GNU/Linux for desktop will remain a solution for hobbyists.

I hope that those who disseminate this information to the general public will also come to their senses on these points and understand the importance of having a vision.
