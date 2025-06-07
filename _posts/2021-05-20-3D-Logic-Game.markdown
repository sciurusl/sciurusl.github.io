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
  
![Blender Asset]({{site.baseurl}}/images/pages/BP/unityCharacterController.PNG)

---

## Game Mechanics

Players must solve puzzles by:
- Moving crates and stepping on pressure plates
- Interacting with mirrors and teleporters
- Switching between mirrored environments

Each puzzle room introduces a new mechanic or builds on previous ones. Here's a look at some components and scenes:

![Modular Room]({{site.baseurl}}/images/pages/BP/MiddleRoom.PNG)
![Modular Combinations]({{site.baseurl}}/images/pages/BP/modularComponentsCombination.jpg)
![Door Prefab]({{site.baseurl}}/images/pages/BP/door_prefab.PNG)
![Crate Shader]({{site.baseurl}}/images/pages/BP/NonMirroringCrateShaderGraph.PNG)
![Puzzle Room]({{site.baseurl}}/images/pages/BP/puzzle_room.PNG)

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
## Advanced Code Highlight: Mirrored World Rotation System

One of the core mechanics of this game is the **Mirrored World Rotation System** — a complex, interwoven feature that allows the player to transition between mirrored versions of the game world. This system blends gameplay logic, physics, camera control, and environmental response.

It consists of three main scripts:

### `RotatingWorld.cs` — World Rotation Controller

This script enables one of the most visually and mechanically unique features in the game: rotating the world around the player. 
It detects when the player enters a teleport zone and initiates a 180° rotation of the environment around the camera. It also manages transition timing, disables gravity, and ensures precise angle snapping.

#### Highlights

- Dynamically rotates the entire environment around a pivot.
- Ensures correct player and camera alignment post-rotation.
- Supports both direction-based and input-triggered world changes.
- Visually and logically reinforces the mirrored-world theme.
- Uses `RotateAround()` to animate the rotation.
- Disables gravity during transition to prevent glitches.
- Coordinates with other components like the player controller and camera.

```csharp
/// <summary>
/// Manages the main rotation of worlds
/// </summary>
public class RotatingWorld : MonoBehaviour
{
    public Transform player;
    public Transform animUp;
    public Transform animParW;
    public float animTime = 1.5f;

    public bool playerIsOverlapping = false;

    public int rotAnglePerSecond = 60;
    public int rotAngle = 180;
    public bool teleporting = false;
    public bool leftTheTeleport = true;
    private float time;

    private Vector3 gravity;
    public Transform playerCamera;

    private Vector3 rotVector;
    private Vector3 cameraPos;
    private int[] coords = new int[3];

    void Start()
    {
        rotVector = transform.up;
        if (!transform.parent.parent.parent.CompareTag("NormalWorld"))
            gameObject.SetActive(false);
        leftTheTeleport = true;
    }

    /// <summary>
    /// Checks the value of the playerIsOverlapping variable. When true and if not already rotating, it starts the rotation of the worlds.
    /// Otherwise, it checks if the rotation has reached the desired angle and stops the rotation.
    /// It rotates all objects around the Camera so that the player does not move during the rotation.
    /// It uses the RotateAround function of the Transform component. 
    /// It takes the center of the rotation in Vector3 format, the direction of the rotation, and rotation angle. 
    /// The angle is calculated as the multiplication of Time.deltaTime and a rotation angle per
    /// second stored in the rotAnglePerSecond variable.
    /// It also calls the StartRotation method of the CameraRotating class
    /// </summary>
    void Update()
    {
        if (playerIsOverlapping)
        {
            if (leftTheTeleport)
            {
                if (!teleporting)
                {
                    cameraPos = playerCamera.transform.position;
                    Debug.Log("setting camera's position");
                    teleporting = true;
                    time = Time.time;
                    player.gameObject.GetComponentInChildren<PlayerMovement>().teleported = true;
                    player.gameObject.GetComponentInChildren<PlayerMovement>().gravity = false;

                    animUp.RotateAround(new Vector3(cameraPos.x, cameraPos.y - 4, cameraPos.z), rotVector, rotAnglePerSecond * Time.deltaTime);
                    if (player.GetComponentInChildren<CameraRotating>())
                        player.GetComponentInChildren<CameraRotating>().StartRotation(transform.parent);
                    gravity = Physics2D.gravity;
                    Physics2D.gravity = new Vector3(0, -1.0F, 0);
                }
                else
                {
                    if (Time.time - time >= rotAngle / rotAnglePerSecond)
                    {

                        leftTheTeleport = false;
                        teleporting = false;
                        playerIsOverlapping = false;
                        player.gameObject.GetComponentInChildren<PlayerMovement>().gravity = true;
                        Physics2D.gravity = gravity;
                        for (int i = 0; i < 3; i++)
                        {
                            if (animUp.localEulerAngles[i] > 100)
                            {
                                if (animUp.localEulerAngles[i] > 340)
                                    coords[i] = 0;
                                else
                                    coords[i] = 180;
                            }

                            else
                                coords[i] = 0;
                        }

                        animUp.localEulerAngles = new Vector3(coords[0], Mathf.Round(animUp.localEulerAngles.y), coords[2]);

                        if (player.GetComponentInChildren<CameraRotating>())
                            player.GetComponentInChildren<CameraRotating>().ChangeBackToNormal(transform.parent);
                    }
                    else
                    {

                        animUp.RotateAround(new Vector3(cameraPos.x, cameraPos.y - 4, cameraPos.z), rotVector, rotAnglePerSecond * Time.deltaTime);

                    }
                }
            }

        }
    }


    /// <summary>
    /// Checks if the other Collider component is assigned to a player by comparing its
    /// tag to the player tag. It sets the playerIsOverlappig variable to true
    /// </summary>
    /// <param name="other"></param>
    private void OnTriggerEnter(Collider other)
    {
        if (other.tag == "player")
        {
            playerIsOverlapping = true;
            if (leftTheTeleport)
                transform.parent.GetComponentInChildren<AudioSource>().Play();
        }
    }

    /// <summary>
    /// Checks if the other Collider component is assigned to a player by comparing its
    /// tag to the player tag. It sets the playerIsOverlappig variable to false
    /// </summary>
    /// <param name="other"></param>
    private void OnTriggerExit(Collider other)
    {
        if (teleporting)
            return;
        if (other.tag == "player")
        {
            leftTheTeleport = true;
            playerIsOverlapping = false;
            player.gameObject.GetComponentInChildren<PlayerMovement>().teleported = false;

        }
    }
}
```

### CameraRotating.cs — Camera Animation & World Sync
Controls the player's camera during the transition phase. It temporarily disables player control, moves the camera in a smooth animation, and updates objects in both worlds.

#### Highlights
- Smooth Lerp animation coroutine.
- Coordinates environmental updates across mirrored states.
- Reactivates movement once the transition ends.

```csharp
/// <summary>
///  Manages the updating of the objects during the rotation
/// </summary>
public class CameraRotating : MonoBehaviour
{
    public Transform otherWorldParent;
    public float cameraOffset = 0.75f;
    public float cameraRotationViewOffset = 3.5f;
    public float animTime;// = 2f;
    public Transform tempParent;
    public Transform upWorldParent;
    private Transform portalParent;
    private Vector3 origLocalPos;
    private Transform portalTmp;

    private List<Transform> rooms = new List<Transform>();

    public void Start()
    {
        origLocalPos = transform.localPosition;
    }

    /// <summary>
    /// Calls UpWorld and ParallelWorld objects to update their objects (rigidbodies, doors, mirroring objects, pressure plates, and lights
    /// It also starts the animation of the camera
    /// </summary>
    /// <param name="portal"></param>
    public void StartRotation(Transform portal)
    {

        transform.parent.GetComponent<CharacterController>().enabled = false;
        transform.parent.GetComponentInChildren<PlayerMovement>().enabled = false;
        GameManager.instance.teleportationEnded = false;

        rooms.Clear();
        rooms.Add(GameManager.instance.playerRoomLocation);
        rooms.Add(GameManager.instance.playerParallelRoomLocation);
        foreach (Transform room in rooms)
        {
            //if (GameManager.instance.inParallelWorld)
            room.GetComponent<GravityObjects>().UpdateDoor(true);
            room.GetComponent<GravityObjects>().UpdateRigidbody(true);
            room.GetComponent<GravityObjects>().UpdateMirroringObejcts();
            room.GetComponent<GravityObjects>().UpdatePressurePlates();
            room.GetComponent<GravityObjects>().UpdateLights();
        }
        portalTmp = portal;
        transform.parent.GetComponentInChildren<PlayerMovement>().anim.SetFloat("Walking", 0);
        StartCoroutine(LerpFromTo());
    }

    /// <summary>
    /// MSmoothly moves the camera back and slightly up
    /// </summary>
    /// <returns></returns>
    private IEnumerator LerpFromTo()
    {
        float elapsedTime = 0f;
        float waitTime = 1f;
        Vector3 curPos = transform.position;
        Vector3 playersCurPos = transform.parent.position;

        Vector3 dir = Vector3.Normalize(portalTmp.position - playersCurPos);
        dir.y = 0;
        transform.parent.position = playersCurPos + dir;

        while (elapsedTime < waitTime)
        {
            transform.position = Vector3.Lerp(curPos, curPos - 4*Vector3.Normalize((transform.forward-transform.up/4.5f)), (elapsedTime / waitTime));            
            elapsedTime += Time.deltaTime;

            yield return null;
        }

    }

    /// <summary>
    /// Returns camera to its default position
    /// </summary>
    /// <returns></returns>
    private IEnumerator LerpFromToBack()
    {
        float elapsedTime = 0f;
        float waitTime = 1f;
       
        Vector3 curPos = transform.localPosition;

        while (elapsedTime < waitTime)
        {
            transform.localPosition = Vector3.Lerp(curPos, origLocalPos, (elapsedTime / waitTime));
            elapsedTime += Time.deltaTime;

            yield return null;
        }
    }

    /// <summary>
    /// Calls UpWorld and ParallelWorld objects to update their objects 
    /// (rigidbodies, doors, mirroring objects, pressure plates, and lights) at the end of the rotation
    /// It also starts the returning animation of the camera
    /// </summary>
    /// <param name="portal"></param>
    public void ChangeBackToNormal(Transform portal)
    {
        StartCoroutine(LerpFromToBack());
        
        GetComponent<MouseView>().playerBody = transform.parent;
        
        foreach (Transform room in rooms)
        {

            room.GetComponent<GravityObjects>().UpdateRigidbody(false);
            room.GetComponent<GravityObjects>().UpdateMirroringObejcts();
            room.GetComponent<GravityObjects>().UpdatePressurePlates();
            room.GetComponent<GravityObjects>().UpdateDoor(false);
            room.GetComponent<GravityObjects>().UpdateLights();
        }

        transform.parent.GetComponent<CharacterController>().enabled = true;
        transform.parent.GetComponentInChildren<PlayerMovement>().enabled = true;

        StartCoroutine(StopTeleporting());
        GameManager.instance.teleportationEnded = true;

        GameManager.instance.inParallelWorld = !GameManager.instance.inParallelWorld;
        GameManager.instance.rotated = !GameManager.instance.rotated;
       
    }

    /// <summary>
    /// Manages updating of objects after changing of worlds when the game is loaded
    /// </summary>
    public void ChangeBackToNormal()
    {
        GameManager.instance.playerRoomLocation.GetComponent<GravityObjects>().UpdateRigidbody(false);
        GameManager.instance.playerRoomLocation.GetComponent<GravityObjects>().UpdateLights();
        GameManager.instance.playerParallelRoomLocation.GetComponent<GravityObjects>().UpdateRigidbody(false);
        GameManager.instance.playerParallelRoomLocation.GetComponent<GravityObjects>().UpdateLights();

        transform.parent.GetComponent<CharacterController>().enabled = true;
        transform.parent.GetComponentInChildren<PlayerMovement>().enabled = true;
      
        GameManager.instance.teleportationEnded = true;

        GameManager.instance.inParallelWorld = !GameManager.instance.inParallelWorld;
        GameManager.instance.rotated = !GameManager.instance.rotated;

        if (GameManager.instance.currentLevelsActive[0])
        {
            foreach (MainDoorOpening door in FindObjectsOfType<MainDoorOpening>())
            {
                if (!door.tutorialDoor)
                    continue;
                door.Open();
            }
        }

    }

    IEnumerator StopTeleporting()
    {
        yield return new WaitForSeconds(1);
        transform.parent.GetComponentInChildren<PlayerMovement>().teleported = false;
    }
}
```

### GravityObjects.cs — Environment Updater
This component manages what changes in the environment when the player switches worlds — such as physics, lighting, doors, pressure plates, and mirroring objects.

#### Highlights
- Dynamically toggles rigidbodies and gravity.
- Enables/disables pressure plates and mirror logic.
- Activates/deactivates parallel world lights and doors.

```csharp
/// <summary>
/// Updates isKinematic property and gravity property of rigidbodies based on the rotationStarted parameter 
/// and whether the objects are in the up world or in the parallel world
/// </summary>
/// <param name="rotationStarted"></param>
public void UpdateRigidbody(bool rotationStarted)
{
    if (rotationStarted)
    {
        foreach (var rb in rigidbodyChildren)
            rb.isKinematic = true;
    }
    else
    {
        if (!upWorld && !GameManager.instance.inParallelWorld)
        {
            foreach (var rb in rigidbodyChildren)
                rb.isKinematic = false;

            foreach (Transform crate in nonMirroringObejcts)
                crate.GetComponent<Rigidbody>().useGravity = true;
        }
      ...
    }
}
```

Toggling lights:

```csharp
public void UpdateLights()
{
    foreach (GameObject light in parallelLights)
    {
        light.SetActive(!GameManager.instance.inParallelWorld);
    }
}
```

Pressure plate toggle:

```csharp
public void UpdatePressurePlates()
{
    for (int i = 0; i < pressurePlates.Length; i++)
    {
        pressurePlates[i].GetComponent<BoxCollider>().enabled = 
            !pressurePlates[i].GetComponent<BoxCollider>().enabled;

        pressurePlates[i].GetComponent<PressurePlate>().enabled = 
            !pressurePlates[i].GetComponent<PressurePlate>().enabled;
    }
}
```

### Summary
These three scripts form a tightly integrated system that handles:

- Smooth transitions between mirrored dimensions
- Logical updates to environment and gameplay states
- Player control synchronization and immersion

This rotation system demonstrates:
- Deep understanding of Unity’s transform system
- Modular object-oriented programming
- Cohesive gameplay design and code architecture

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

