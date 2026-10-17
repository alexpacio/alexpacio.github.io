.. title: Stop hyping crap: AI can build genuinely useful things, like robocam
.. slug: robocam-head-tracking-mouse-control
.. date: 2026-10-17 12:00:00 UTC+02:00
.. tags: robocam, rust, opencv, windows, accessibility, llms, ai, egui
.. category: programming
.. description: The press hypes AI clones that fall apart at the first look at a Cargo.toml, while AI can build genuinely useful tools. robocam lets you control the mouse with a webcam by moving your head.
.. type: text

Most people love talking and hyping sh*t nowadays. We know this.

Artcraft and the Adobe clones are crap, but for some reason the press keeps talking about them. It would have been enough to look at the ``Cargo.toml`` file in the root of the projects to see that an application this complex, using egui as its framework, could not possibly be sophisticated or even remotely credible as a solution. It took 5 seconds to realize that, but no. We prefer to talk about and hype up crap instead, when with AI you can actually build genuinely useful and well-made things.

You just need to know that you can't replicate Photoshop with egui, and why!

robocam
=======

An example of something useful is `robocam <https://github.com/alexpacio/robocam>`__, an application that lets you control the mouse with a webcam, simply by moving your head in front of it.

.. image:: /files/robocam/live_session_video.webp
   :alt: robocam live session: controlling the mouse by moving the head in front of the webcam
   :width: 400px
   :align: center

robocam is an app I developed for a close relative of mine, and I hope it can also be useful in other cases, such as for people suffering from conditions or trauma that prevent limb movement. Applications like this enable people with disabilities to use a computer at a low cost, even extensively: in fact, the application is optimized to efficiently use SketchUp.

How it works
============

robocam supports Windows 11 and uses (unsafe) Rust, OpenCV and Windows primitives to generate a bar at the top of your screen where you can change the mouse behavior: single click, double click, dragging, and much more.

Nothing exotic here: a webcam, a computer vision library to track the head, and the native Windows APIs to move the cursor and synthesize the clicks. This is the kind of job where a native, statically compiled application makes sense, and where AI-assisted development lets one person ship something solid for a very specific need.

Get it
======

The source code is on GitHub: https://github.com/alexpacio/robocam

I remain available to support, time permitting, any GitHub issues that may come up.
