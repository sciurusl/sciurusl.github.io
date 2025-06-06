---
layout: post
title:  3D Logic Game in Unity
date:   2021-05-20 12:46:27 +0300
img: BP/MiddleRoomBig.jpg
tags: [Unity, C#, bachelor-thesis]
---
As part of my Bachelor's thesis at the Czech Technical University in Prague, I designed and developed a modular 3D logic puzzle game in Unity. The core concept involves mirrored worlds, where players interact with environmental elements to unlock paths and progress through interconnected rooms.

The project includes:
- 3 uniquely themed levels
- 29 modular puzzle rooms
- 100% custom models created in Blender
- Game logic implemented in C# using Unity HDRP

---

## Game Mechanics

Players must solve puzzles by:
- Moving crates and stepping on pressure plates
- Interacting with mirrors and teleporters
- Switching between mirrored environments

Each puzzle room introduces a new mechanic or builds on previous ones. Here's a look at some components and scenes:

![Modular Room]({{site.baseurl}}/images/pages/BP/MiddleRoom.PNG)
![Modular Combinations]({{site.baseurl}}/images/pages/BP/modularComponentsCombination.jpg)
![Door Prefab]({{site.baseurl}}/images/pages/BP/door_prefab.jpg)
![Crate Shader]({{site.baseurl}}/images/pages/BP/NonMirroringCrateShaderGraph.PNG)
![Puzzle Room]({{site.baseurl}}/images/pages/BP/puzzle_room.PNG)
![Blender Asset]({{site.baseurl}}/images/pages/BP/unityCharacterController.jpg)

## Modular Wall Components

These are custom-modeled wall prefabs used to construct levels in a modular and repeatable fashion:

| Wall Pillars Wide | Wall Open | Wall No |
|-------------------|-----------|---------|
| ![WallPillarsWide]({{site.baseurl}}/images/pages/BP/WallPillarsWide.PNG) | ![WallOpen]({{site.baseurl}}/images/pages/BP/WallOpen.PNG) | ![WallNo]({{site.baseurl}}/images/pages/BP/WallNo.PNG) |

| Wall Hole Small | Wall Hole Side | Wall Fire |
|------------------|----------------|------------|
| ![WallHoleSmall]({{site.baseurl}}/images/pages/BP/WallHoleSmall.PNG) | ![WallHoleSide]({{site.baseurl}}/images/pages/BP/WallHoleSide.PNG) | ![WallFire]({{site.baseurl}}/images/pages/BP/WallFire.PNG) |

| Wall Cube | Wall Ceiling | Wall Portal |
|-----------|---------------|--------------|
| ![WallCube]({{site.baseurl}}/images/pages/BP/WallCube.PNG) | ![WallCeiling]({{site.baseurl}}/images/pages/BP/wallCeiling.PNG) | ![WallPortal]({{site.baseurl}}/images/pages/BP/WallPortal.PNG) |

## Non-Mirroring Crate Components

These images represent the visual and technical design of the crate used in puzzles that don't interact with world-mirroring mechanics.

| In-Game Appearance | Shader Graph | Normal Map | Raw Model |
|--------------------|--------------|------------|-----------|
| ![Crate Unity]({{site.baseurl}}/images/pages/BP/nonMirroringCrateUnity.PNG) | ![Shader Graph]({{site.baseurl}}/images/pages/BP/NonMirroringCrateShaderGraph.PNG) | ![Normal Map]({{site.baseurl}}/images/pages/BP/NonMirroringCrateNormalMap.png) | ![Crate Model]({{site.baseurl}}/images/pages/BP/nonMirroringCrate.PNG) |

---

## Code Highlight: `RotatingWorld.cs`

This script enables one of the most visually and mechanically unique features in the game: rotating the world around the player.  
It plays a key role in creating mirrored puzzle mechanics and spatial transitions that challenge the player's perception.

### Highlights

- Dynamically rotates the entire environment around a pivot.
- Ensures correct player and camera alignment post-rotation.
- Supports both direction-based and input-triggered world changes.
- Visually and logically reinforces the mirrored-world theme.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// Script for rotating the world — enables shifting between mirrored versions of the environment.
/// </summary>
public class RotatingWorld : MonoBehaviour
{
    public Transform player;
    public Transform center;
    public GameObject pivot;
    private bool rotating = false;
    private float rotationSpeed = 90f;
    private float rotated = 0f;
    private int rotationDir = 0;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Q) && !rotating)
        {
            rotationDir = -1;
            rotating = true;
        }
        else if (Input.GetKeyDown(KeyCode.E) && !rotating)
        {
            rotationDir = 1;
            rotating = true;
        }

        if (rotating)
        {
            float step = rotationSpeed * Time.deltaTime;
            pivot.transform.RotateAround(center.position, Vector3.up, step * rotationDir);
            rotated += step;

            if (rotated >= 90f)
            {
                rotated = 0f;
                rotating = false;
                pivot.transform.rotation = Quaternion.Euler(0f, Mathf.Round(pivot.transform.eulerAngles.y / 90f) * 90f, 0f);
            }
        }
    }
}
```

---

## Thesis Documentation

You can read the full thesis here:  
<object data="https://dspace.cvut.cz/bitstream/handle/10467/94656/F3-BP-2021-Veverkova-Lucie-Bachelor_thesis_veverlu4.pdf?sequence=-1&isAllowed=y" width="100%" height="1080px" type='application/pdf'></object>

📘 [Download Thesis (PDF)](https://dspace.cvut.cz/bitstream/handle/10467/94656/F3-BP-2021-Veverkova-Lucie-Bachelor_thesis_veverlu4.pdf?sequence=-1&isAllowed=y)

It includes chapters on:
- Game and level design
- Custom asset pipeline
- Code architecture
- User testing methodology

---

## Download & Source Code

You can download the complete Unity project:
- [GitHub Repository](https://github.com/sciurusl/3D-logic-game-in-Unity)
- [CTU DSpace Archive](https://dspace.cvut.cz/handle/10467/94656) *(includes PRILOHA split archives)*

C# files are in Assets/Scripts folder. Models can be found in Assets/Models folder.

**To run the game:**
1. Extract all PRILOHA files.
2. Open the project in **Unity 2020.3.18f1** or newer.
3. Ensure **Blender** is installed if you want to view/edit models.

---

## ▶️ Gameplay Video

Watch a short demo of the game mechanics and level design:  
[![Watch on YouTube](https://img.youtube.com/vi/VPyuXbMLG64/0.jpg)](https://www.youtube.com/watch?v=VPyuXbMLG64)

---

This project is a comprehensive demonstration of my skills in Unity, C#, modular level design, asset integration, and gameplay programming.  
**Feedback is welcome!**

