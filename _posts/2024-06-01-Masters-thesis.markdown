---
layout: post
title:  Using Tight Bounding Volumes for Bounding Volume Hierarchies
date:   2024-06-01 12:46:27 +0300
img: DP/HairballOBBDiTO.PNG
tags: [Ray Tracing, Data Structures, C++, CUDA, GPU Programming]
---
For my Master's thesis, I conducted research on the use of tight bounding volumes in bounding volume hierarchies.

This thesis reviews methods for building bounding volume hierarchies (BVH) for ray tracing. It focuses on techniques applicable to interactive applications with dynamic scenes. Based on the research of these techniques, two efficient BVH construction algorithms that use tight bounding volumes, such as OBB and ODOP, are implemented. For the implementation, available codes in the CUDA language were used and unified inside a common framework. The resulting BVHs were compared in terms of their construction speed and ray tracing speed on six different scenes. The bottlenecks of this construction were discussed, and possible improvements were suggested.

<div style="display: flex; justify-content: space-around;">
  <img src="/images/pages/DP/HairballAABB.PNG" alt="AABB BVH constructed using the PLOC algorithm" style="height: 200px; width: auto; border: none;">
  <img src="/images/pages/DP/HairballOBBDiTO.PNG" alt="OBB BVH constructed using the DiTO algorithm" style="height: 200px; width: auto; border: none;">
  <img src="/images/pages/DP/HairballOBBODOP.PNG" alt="OBB BVH constructed using the ODOP algorithm" style="height: 200px; width: auto; border: none;">
</div>

BVH constructed using different construction methods built over the Hairball scene. From left: AABB, OBB using the DiTO algorithm, OBB using the ODOP algorithm.

<object data="https://dspace.cvut.cz/bitstream/handle/10467/94656/https://dspace.cvut.cz/bitstream/handle/10467/114563/F3-DP-2024-Veverkova-Lucie-Using%20Tight%20Bounding%20Volumes%20for%20Bounding%20Volume%20Hierarchies.pdf?sequence=-1&isAllowed=y" width="100%" height="1080px" type='application/pdf'></object>

The project can be downloaded here: [Download](https://dspace.cvut.cz/bitstream/handle/10467/114563/F3-DP-2024-Veverkova-Lucie-priloha-final.7z.001?sequence=-1&isAllowed=y) 

or seen here: [Dspace](https://dspace.cvut.cz/handle/10467/114563)

Where the project with the code is zipped and splitted in the PRILOHA files.
