<!--
  The face of github.com/LimoniteMC. It only says what limonite-client's README says, so the milestone
  badge, the numbers and the roadmap follow that page. Images are absolute links, because they live in
  limonite-client and this file lives in LimoniteMC/.github.
-->

<div align="center">

<img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/title.png" alt="Limonite Client" width="640">

**A Minecraft: Bedrock Edition client, written in Rust from scratch,<br>with a forked [Pumpkin](https://github.com/Pumpkin-MC/Pumpkin) server built in for singleplayer.**

[![Bedrock 1.26.51](https://img.shields.io/badge/Bedrock-1.26.51-818600?style=for-the-badge&labelColor=1A1A1A)](https://github.com/LimoniteMC/limonite-client#the-pumpkin-fork)
[![Rust 2024 edition](https://img.shields.io/badge/Rust-2024_edition-818600?style=for-the-badge&logo=rust&logoColor=white&labelColor=1A1A1A)](https://www.rust-lang.org/)
[![Licence: GPL-3.0](https://img.shields.io/badge/licence-GPL--3.0-818600?style=for-the-badge&labelColor=1A1A1A)](https://github.com/LimoniteMC/limonite-client/blob/main/LICENSE)
<br>
[![Current milestone: 5, playable singleplayer](https://img.shields.io/badge/current_milestone-5_%C2%B7_playable_singleplayer-818600?style=for-the-badge&labelColor=1A1A1A)](https://github.com/LimoniteMC/limonite-client/blob/main/docs/milestones/m5-plan.md)
[![Developed AI-first with Claude](https://img.shields.io/badge/AI--first-Claude-818600?style=for-the-badge&logo=claude&logoColor=white&labelColor=1A1A1A)](https://github.com/LimoniteMC/limonite-client#how-its-made)

[**The client**](https://github.com/LimoniteMC/limonite-client) · [**The fork**](https://github.com/LimoniteMC/Pumpkin/tree/limonite) · [**Build it**](https://github.com/LimoniteMC/limonite-client#try-it) · [**Wiki**](https://github.com/LimoniteMC/limonite-client/wiki) · [**Report a bug**](https://github.com/LimoniteMC/limonite-client/issues/new/choose)

<br>

<img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/screenshots/hero.jpg" alt="Limonite drawing spruce-covered mountains and stone cliffs with vanilla textures" width="100%">

<sub>Flying over spruce-covered mountains in Limonite: 3,787 chunks loaded, 103 fps in a release build.</sub>

</div>

> **limonite** /ˈlaɪmənaɪt/ *noun*
>
> **1.** A yellow-brown iron ore, made of much the same stuff as rust.<br>
> **2.** A Minecraft client written in Rust, with a lemon for a mascot.

> [!WARNING]
> **Limonite is under heavy construction, so expect little to no functionality.** Today you can walk
> and fly around a singleplayer world drawn with vanilla textures, with a HUD, and that's it: no
> lighting, sky or mobs yet, and nothing to build with. There are no releases yet either, so you
> [build it from source](https://github.com/LimoniteMC/limonite-client#try-it).

## Two repos

<table>
<tr>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/icons/cube.svg" width="28" height="28" alt="">

**[limonite-client](https://github.com/LimoniteMC/limonite-client)**

The client, and the place to start: the window, the renderer, the world state, the input and the
pack filesystem, in one Cargo workspace.

</td>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/icons/network.svg" width="28" height="28" alt="">

**[Pumpkin](https://github.com/LimoniteMC/Pumpkin/tree/limonite)**

Our fork of [Pumpkin](https://github.com/Pumpkin-MC/Pumpkin), the Rust Minecraft server, on branch
`limonite`. It stays small on purpose: changes to Pumpkin's files go through named seams, and
upstream's `master` is merged in every week.

</td>
</tr>
</table>

## One process, two threads, one seam

The client renders, predicts and presents. The embedded server owns world generation, physics, mob
AI and saving, so nothing Pumpkin already does gets rebuilt. They meet only at the `Transport` trait,
so the server doesn't know or care whether a player is the host in the same process, a LAN guest or
a future peer.

```text
┌──────────────────────── one binary ────────────────────────┐
│ main thread (winit)                                        │
│   Renderer (wgpu)  <──  World state  ──>  UI               │
│                             ▲                              │
│                             │ in-memory channel (bytes)    │
│                             ▼                              │
│ background thread (tokio): forked Pumpkin                  │
│   LocalChannel  <──>  Game core (worldgen, physics, AI)    │
│                        │                  │                │
│                   NetherNet (LAN)     P2P (later)          │
└────────────────────────┼──────────────────┼────────────────┘
                         ▼                  ▼
                     LAN guests        remote peers
```

<sub>Singleplayer works today, over memory, with no socket in the process. LAN is milestone 7, and P2P is milestone 11.</sub>

## Measured, not guessed

Today Limonite is a window onto a real world. Chunks load as you go, walking and flying match the
official client, and most blocks wear their real models if you have Java Edition's jar.

<table>
<tr>
<td align="center" valign="top" width="25%">
<h3>0.000015</h3>
<sub>blocks: how closely the official client's 12 recorded runs replay, tick by tick</sub>
</td>
<td align="center" valign="top" width="25%">
<h3>0</h3>
<sub>sockets in the process while the host plays singleplayer</sub>
</td>
<td align="center" valign="top" width="25%">
<h3>4.8×</h3>
<sub>faster Bedrock chunk encoding in the fork: 145 µs instead of 696, byte for byte the same</sub>
</td>
<td align="center" valign="top" width="25%">
<h3>1,016</h3>
<sub>fps at radius 16 on the dev machine: the baseline the GPU-driven renderer has to beat</sub>
</td>
</tr>
</table>

The official client is the ground truth. Here's the same spot in the same world, at the same pose,
in both clients:

<table>
<tr>
<th width="50%">Limonite today</th>
<th width="50%">The official Bedrock client</th>
</tr>
<tr>
<td><img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/screenshots/same-pose-limonite.jpg" alt="A birch forest by a lake, drawn by Limonite: vanilla textures, even light and a plain blue sky"></td>
<td><img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/screenshots/same-pose-bedrock.jpg" alt="The same birch forest in the official Bedrock client, with lighting, shadows, clouds and reflective water"></td>
</tr>
</table>

<sub>Most of the difference is light. Milestone 5 adds smooth lighting, ambient occlusion, the sky's
colour and fog; clouds, weather and reflective water aren't part of it.</sub>

## Roadmap

Each milestone ends in something runnable, and is done only when its acceptance check passes. Green
is done, orange is under way, and grey is planned.
[GitHub's milestones](https://github.com/LimoniteMC/limonite-client/milestones) follow the same list,
with the current one's steps ticked as they land.

```diff
+  0   Recon                     How Pumpkin talks to Bedrock, mapped                  
+  1   Workspace and fork seam   The Transport trait, the Java listener compiled out   
+  2   Headless embedded client  The host joins over LocalChannel, with no socket      
+  3   Chunk map                 Chunks decoded into Bedrock-native world state        
+  4   First pixels              A window, a camera, terrain in vanilla textures       
!  5   Playable singleplayer     Physics, break and place, light, mobs, a HUD, worlds  
#  5b  Containers                Crafting, chests and furnaces                         
#  6   GPU-driven renderer       Compute meshing, GPU culling, indirect draws          
#  7   Open to LAN               NetherNet started and stopped from a running world    
#  8   Resource packs            Full pack stacking, Molang, entity models, sounds     
#  9   Iris packs                Java Edition shader packs, translated to SPIR-V       
#  10  Extras                    Ray tracing, a JSON UI subset, server forms, particles
#  11  P2P                       Joining across networks with no port forwarding       
```

## How it's made

Limonite is built AI-first. Its code, plans and docs are written by **Claude Opus 5.5**, and nearly
every commit is co-authored by Claude. [@LaDoodleInTheHat](https://github.com/LaDoodleInTheHat)
steers: each milestone's plan is approved before any code is written, and each milestone is accepted
against the official Bedrock client. The
[plans](https://github.com/LimoniteMC/limonite-client/tree/main/docs/milestones) are public, with
every deviation from them on the record.

<table>
<tr>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/icons/layers.svg" width="28" height="28" alt="">

**Seams over spaghetti**

Every change to Pumpkin goes through a small, named seam, so upstream syncs stay cheap.

</td>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/icons/chip.svg" width="28" height="28" alt="">

**Detect capabilities, never brands**

A GPU feature flag decides a code path; a vendor name never does.

</td>
</tr>
<tr>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/icons/bolt.svg" width="28" height="28" alt="">

**Measure, don't guess**

Performance tiers come from timed frames, not GPU names.

</td>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/LimoniteMC/limonite-client/main/docs/readme/icons/globe.svg" width="28" height="28" alt="">

**The official client is the ground truth**

Packets are golden-tested against its bytes, and what Limonite draws is compared side by side with it.

</td>
</tr>
</table>

## Ground rules

Limonite is for your own worlds and your own LAN.

- **Your game, your files.** Vanilla textures are read at runtime from your own Bedrock install, and
  block models from your own Java jar: never copied, cached or shipped.
- **Not for public servers or Realms.** Custom clients can trip anti-cheat or break terms of
  service, so Limonite is only tested on its embedded server and its owner's LAN.
- **No Marketplace decryption.** Encrypted packs aren't supported, and never will be.

<div align="center">
<br>

<img src="https://avatars.githubusercontent.com/u/350470842?s=96" alt="" width="48" height="48">

**Enjoy a rustfully amazing life.**

<sub>Limonite is not an official Minecraft product, and is not approved by or associated with Mojang or Microsoft.<br>The screenshots show Minecraft's textures, and one shows the official client. They're © Mojang, and not covered by the GPL.</sub>

</div>
