---
layout: post
title: Reverse Engineering Arturia MiniLab mkII
description:
image:
category: [projects]
tags: [midi,synthesizer,electronics,reverse-engineering]
---
#### Intro
I've started working on building a modular synthesizer, but have made some "interesting"
choices along the way. One of the first of those was purchasing a used Arturia MiniLab mkII
MIDI controller with the intention of using it for this synthesizer before I had even begun
acquiring the necessary components for the synth, and before I really started to understand
how MIDI interfaces with a modular synth. As a result, I now have a totally acceptable MIDI
controller that relies on USB MIDI, instead of simply sending MIDI signals itself. This 
wouldn't be an issue if I intended to have a computer or something else to serve as a USB
host somewhere in the mix of my setup, but I'd like to have this be a computer-free setup.
Alas. Now I'd like to find a way to piggyback the USB MIDI and built a MIDI USB host directly
into the housing of the controller so that I can have both traditional 5-pin MIDI connectors,
as well as USB MIDI (in the event that I decide to use the controller with a computer down
the road).

#### Scope already creeping
Even with this limited task at hand, I think I'd also like to understand more about how
the a controller actually functions, how it generates the MIDI commands to send via USB. 
So, as a result, I'm going to try and do some firmware _and_ hardware reverse engineering 
here.

