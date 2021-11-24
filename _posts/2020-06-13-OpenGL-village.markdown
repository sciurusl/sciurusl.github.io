---
layout: post
title:  OpenGL village
date:   2020-06-13 10:11:50 +0300
img: PGR/fire.jpg
tags: [OpenGL, C++]
---
For a student project, I created a small village scene in OpenGL and C++, where the player could
somehow interact with the environment. 

The goal of this project was to create a scene with a village, where the player can move using WASD, 
cut trees by clicking on them, collect wood and light a fire by clicking on one of the rocks.
The player can turn on a flashlight, a fog, change camera and open a menu. The time changes, which is visualized by 
darkening and dawning of the sky.

![village1]({{site.baseurl}}/images/pages/PGR/screenshot1.jpg)

![fog]({{site.baseurl}}/images/pages/PGR/screenshot2.jpg)

An example of some of the shaders: 
The part of the project can be found here: [Github](https://github.com/sciurusl/OpenGL-village).

This is part of the vertex shader which computes color using texture and light in the current vertex

    struct Material {      // structure that describes currently used material
    vec3  ambient;       // ambient component
    vec3  diffuse;       // diffuse component
    vec3  specular;      // specular component
    float shininess;     // sharpness of specular reflection
    
    bool  useTexture;    // defines whether the texture is used or not
    };
    
    // warning: sampler inside the Material struct can cause problems -> so its outside
    uniform sampler2D texSampler;  // sampler for the texture access
    
    struct Light {         // structure describing light parameters
    vec3  ambient;       // intensity & color of the ambient component
    vec3  diffuse;       // intensity & color of the diffuse component
    vec3  specular;      // intensity & color of the specular component
    vec3  position;      // light position
    vec3  spotDirection; // spotlight direction
    float spotCosCutOff; // cosine of the spotlight's half angle
    float spotExponent;  // distribution of the light energy within the reflector's cone (center->cone's edge)
    };
    
    in vec3 position;           // vertex position in world space
    in vec3 normal;             // vertex normal
    in vec2 texCoord;           // incoming texture coordinates
    
    uniform bool fog;
    uniform bool cat;
    uniform bool cube;
    
    uniform float time;         // time used for simulation of moving lights (such as sun)
    uniform Material material;  // current material
    
    uniform mat4 PVMmatrix;     // Projection * View * Model  --> model to clip coordinates
    uniform mat4 Pmatrix;     // Projection * View * Model  --> model to clip coordinates
    uniform mat4 Vmatrix;       // View                       --> world to eye coordinates
    uniform mat4 Mmatrix;       // Model                      --> model to world coordinates
    uniform mat4 normalMatrix;  // inverse transposed Mmatrix
    
    
    uniform vec3 reflectorPosition;   // reflector position (world coordinates)
    uniform vec3 reflectorDirection;  // reflector direction (world coordinates)
    
    smooth out vec2 texCoord_v;  // outgoing texture coordinates
    smooth out vec4 color_v;       // outgoing fragment color
    
    smooth out vec3 thePosition;
    smooth out vec4 theColor;
    smooth out vec3 theNormal;
    smooth out vec3 pointLightPos;
    smooth out vec3 pointLightPosRef;
    smooth out vec3 refDirection;
    
    
    out float visibility;
    float density = 0.5;
    float gradient = 1.5;
    
    void main() {
    
    visibility = 0.01;
    
    vec3 vertexPosition = (Vmatrix * Mmatrix * vec4(position, 1.0)).xyz;         // vertex in eye coordinates
    vec3 vertexNormal   = normalize( (Vmatrix * normalMatrix * vec4(normal, 0.0) ).xyz);   // normal in eye coordinates by NormalMatrix
    
    vec4 outputColor = vec4(0.0, 0.0, 0.0, 1.0f);
    
    if(cat){
    float pct = abs(sin(time/3));
    gl_Position = PVMmatrix * vec4(position*mix(1, 1.05, pct), 1);
    }
    else
    gl_Position = PVMmatrix * vec4(position, 1);   // out:v vertex in clip coordinates
    
    thePosition = vertexPosition;
    theColor = outputColor;
    theNormal = vertexNormal;
    
    vec4 distanceCam = Vmatrix* Mmatrix * vec4(position, 1.0);
    float distance  = length(distanceCam);
    
    
    visibility = exp(-pow((distance*density), gradient));
    visibility = clamp(visibility,0.0, 1.0);
    
    color_v = outputColor;
    texCoord_v = texCoord;
    }


![vertex shader]({{site.baseurl}}/images/pages/PGR/openglvert.jpg)

This is part of the fragment shader which computes color in the current fragment



    #version 140
    
    struct Material {           // structure that describes currently used material
    vec3  ambient;            // ambient component
    vec3  diffuse;            // diffuse component
    vec3  specular;           // specular component
    float shininess;          // sharpness of specular reflection
    
    bool  useTexture;         // defines whether the texture is used or not
    };
    
    uniform sampler2D texSampler;  // sampler for the texture access
    
    uniform Material material;     // current material
    
    struct PointLight{
    vec3  ambient;       
    vec3  diffuse;      
    vec3  specular;     
    vec3  position;
    float cutOff;
    float exponent;
    float effect;
    vec3 attenuation;
    vec3  direction;
    };
    
    struct DirectionalLight{
    vec3  ambient;       
    vec3  diffuse;      
    vec3  specular;     
    vec3  direction;
    bool shines;
    };
    
    struct SpotLight{
    vec3  ambient;       
    vec3  diffuse;      
    vec3  specular;     
    vec3  position;
    bool turnedOn;
    vec3 attenuation;
    };
    
    uniform PointLight pointLight;
    uniform DirectionalLight directionalLight;
    uniform SpotLight spotLight;
    
    uniform mat4 PVMmatrix;     // Projection * View * Model  --> model to clip coordinates
    uniform mat4 Pmatrix;     // Projection * View * Model  --> model to clip coordinates
    uniform mat4 Vmatrix;       // View                       --> world to eye coordinates
    uniform mat4 Mmatrix;       // Model                      --> model to world coordinates
    uniform mat4 normalMatrix;
    
    smooth in vec3 thePosition;
    smooth in vec3 theNormal;
    smooth in vec4 theColor;
    smooth in vec3 pointLightPos;
    smooth in vec3 pointLightPosRef;
    smooth in vec3 refDirection;
    
    smooth in vec4 color_v;        // incoming fragment color (includes lighting)
    smooth in vec2 texCoord_v;     // fragment texture coordinates
    out vec4       color_f;        // outgoing fragment color
    
    uniform bool fog;
    uniform bool cube;
    in float visibility;
    
    uniform float time;
    uniform bool fastenTime;
    float sunSpeed = 0.05f;
    
    vec4 getColor(vec3 pos, vec3 rayDir, vec3 ambient, vec3 diffuse, vec3 specular, Material material, vec3 fragPosition, vec3 fragNormal){
    
    vec3 R = reflect(-rayDir, fragNormal);
    vec3 V = normalize(-fragPosition);
    float NdotL = max(0.0, dot(fragNormal, rayDir));
    float RdotV = max(0.0, dot(R, V));

	return vec4(material.ambient * ambient + material.diffuse * diffuse * NdotL + material.specular * specular * pow(RdotV, material.shininess),1.0f);

    }
    
    vec4 getDirLight(DirectionalLight light, Material material, vec3 fragPosition, vec3 fragNormal) {
    vec3 pos = (Vmatrix * vec4(cos(time * sunSpeed), 0.0, sin(time * sunSpeed), 0.0)).xyz;
    vec3 rayDir = normalize(pos);
    vec4 color = getColor(pos,rayDir,light.ambient, light.diffuse,light.specular,material,fragPosition, fragNormal);
    
    return color;
    }
    
    vec4 getPointLight(PointLight light, Material material, vec3 fragPosition, vec3 fragNormal) {
    
        vec3 pos = (Vmatrix * vec4(light.position, 1.0)).xyz;
    
    vec3 direction = normalize((Vmatrix * vec4(light.direction, 0.0)).xyz);
    
    vec3 rayDir = pos - fragPosition;
    float distance = length(pos - fragPosition);
    rayDir = normalize(rayDir);
    
    vec4 color =  getColor(pos,rayDir,light.ambient, light.diffuse,light.specular,material,fragPosition, fragNormal);
    float angle = dot(-rayDir, direction);
    
    
    float attenuationFactor = 1.0f/(light.attenuation.x + light.attenuation.y*distance + light.attenuation.z*distance*distance);
    
    if(max(angle, 0) < light.cutOff)
    color = vec4(vec3(0.0),1);
    
    return color*attenuationFactor*pow(max(angle,0), light.exponent);
    }
    
    vec4 getSpotLight(SpotLight light, Material material, vec3 fragPosition, vec3 fragNormal) {
    
    vec3 pos = (Vmatrix * vec4(light.position, 1.0)).xyz;
    
    vec3 rayDir = pos - fragPosition;
    float distance = length(pos - fragPosition);
    rayDir = normalize(rayDir);
    vec4 color =  getColor(pos,rayDir,light.ambient, light.diffuse,light.specular,material,fragPosition, fragNormal);
    float attenuationFactor = 1.5f;
    if(distance>1)
    attenuationFactor = 1.0f/(light.attenuation.x + light.attenuation.y*distance + light.attenuation.z*distance*distance);
    
    return color*attenuationFactor;
    }
    
    void main() {

	vec3 L = normalize(pointLightPos-thePosition);
	if(directionalLight.shines)
	color_f = color_v;
	if(!fastenTime)
		color_f = color_f + getDirLight(directionalLight, material, thePosition,theNormal);
	color_f = color_f + getPointLight(pointLight, material, thePosition, theNormal);
	if(spotLight.turnedOn)
		color_f = color_f + getSpotLight(spotLight, material, thePosition, theNormal);
	if(cube)
		color_f = vec4(0.7, 0.7, 0.7, 1);
	if(material.useTexture)
		color_f =  color_f * texture(texSampler, texCoord_v);

	if(fog)
		color_f = mix(vec4(vec3(0.5, 0.5, 0.5), 1.0), color_f, visibility);
    }



![fragment shader]({{site.baseurl}}/images/pages/PGR/openglfrag.jpg)