---
layout: academic
permalink: /notes/coding
title: Unity for Toddlers
description: Sequential Logic Devices can do what Combinational Logic Devices can't; to produce output that depends on both past and current input. 
tags: [Coding]
---
* TOC
{:toc}


# Learning Objectives: Visual Effects

-   Adding visual effects with **URP**
- Basics of 2D lights 
-   Creating and using **shader graphs** for 2D rendering: glow and outline,
-   Adding **post processing effects**: Bloom filter
-   **Parallax background** effect: using Texture scrolling and secondary Cameras, setting culling mask
- Basics of **particle systems**: textured sheet particles
- Creating breakable prefab

# Introduction

By now, you should be more or less equipped to implement the basic game logic (at least it will work), for example create platforms, create enemies, create consumable power-ups, count scores, and restarting the game. The next important element in a game is feedback: both visual and auditory feedback. To prove this point, download the asset in the course handout and import it to your Mario project or a new project (your choice). We will be working with this side project for awhile first before returning to our Mario project. 

> The complete free asset can be downloaded from [GothicVania Church Pack](https://assetstore.unity.com/packages/2d/characters/gothicvania-church-pack-147117?aid=1101lPGj&utm_source=aff) from Unity asset score. The content of this section is obtained from the wonderful [Brackeys tutorial](https://www.youtube.com/watch?v=WiDVoj5VQ4c). 


# Universal Render Pipeline

URP package allows us to easily create optimized graphics. We need to enable it first in our project before we get started. 

## Download URP Package
Go to Window >> Package manager, and download Universal RP from Unity Registry as shown. 

![universal RP](https://www.dropbox.com/s/7v0dbmiccay7hu5/1.png?raw=1)


## Create Pipeline Asset 
This will add the URP package to your project. Afterwards, create a folder in the Assets folder called "Rendering". Inside it, create Rendering >> Universal Render Pipeline >> Pipeline Asset (Forward Renderer). 

![urp2](https://www.dropbox.com/s/9rmnuz90dwah5pd/2.png?raw=1)

## Modify Project Graphics Setting

Go to Edit >> Project Settings >> Graphics and select the UniversalRenderPipelineAsset you have created in the previous set as shown:

![urp3](https://www.dropbox.com/s/ws2vqn922gxnqrb/3.png?raw=1)

## Create 2D renderer 
Since we are working with 2D games now, we need to create a 2D renderer as our Universal Render Pipeline Asset's Renderer. Right click inside the Render folder and Create >> Rendering >> Universal Render Pipeline >> 2D Renderer as shown:

![urp4](https://www.dropbox.com/s/p76bbmrb2ke6nzq/4.png?raw=1)

Click on the UniversalRenderPipelineAsset you created before and load the 2D Renderer you just created under its Renderer List in the inspector and we are **done** with setting up URP for your project. 

![urp5](https://www.dropbox.com/s/i54jrije3aylsx3/5.png?raw=1)

Open Fighter.scene and notice that everything in the Game window is dark, although you can clearly see in the Scene that there are some stuffs in the world as shown below. If you can't see anything on the Scene either, toggle the light bulb icon to *disable Scene lighting* (for our view when prototyping, but not the camera's). 

![scenefighter](https://www.dropbox.com/s/m4fwprxh8e2bznl/6.png?raw=1)


*Time to add some Lights, Camera, Action!*

# Adding 2D Lights

The fun thing about enabling URP is that you can now lit up your sprites, thus creating a bit of a *mood*. Right click on your project hierarchy and observe that there are a few 2D Lights GameObjects that you can create, namely FreeformLight2D, SpriteLight2D, GlobalLight2D, and SpotLight2D.

### GlobalLight2D
Create a GlobalLight2D Game Object, and set the properties:
* Intensity of 0.4
* Target Sorting Layers: All (this determines which Sprite layers to lit up)
* Position at `(0, 0, -1)`

Toggle enable lighting at your Scene view and you should observe a somewhat dimly lit environment like this:

![dimlight](https://www.dropbox.com/s/na38excwcuqd9yf/7.png?raw=1)

### SpotLight2D
Another type of light that you can add to the game os SpotLight2D. Create a gameObject of SpotLight2D type and place it as a child gameObject of the player. Place it somewhere near the Player's head to brighten up that area. You should see something like this in your Scene:

![lighthead](https://www.dropbox.com/s/bi163w5okkvbbqn/37.png?raw=1)

Head over to the Light2D inspector and you observe that you can adjust its properties such as light **color**, **intensity**, **radius**, and **target sorting layers** (yes, you can ask light to just lit up certain areas and not the others). 

Expand the **blending** section and observe a few settings: this dictates how two or more light source should blend when they're near one another. Overlap operation of **additive** will cause the overlapping region to be very bright, while **alpha blend** allows the light with higher order to be rendered on after of the ones with lower order. The gif below demonstrates at first the blue light to be rendered after the green light, giving an overall blue-ish appearance. Afterwards, the light order is swapped so you see that the overall overlapping region has a green-ish appearance. Both lights are set to be "Alpha Blend". 

> Wait for a few seconds, the gif is quite long. 

![alpha blend](https://www.dropbox.com/s/1xkizaxktv4y4kq/blending.gif?raw=1)

## URP SpriteLit and SpriteUnlit Material 

The reason that Unity knows which Sprite should be affected by lighting (or not) is because of the material of the Sprite. To prove this point, expand the Environment GameObjects in the Hierarchy, and click on any background object. As an example, let's use `bg-2(1)`. 

Then in the inspector of `bg-2(1)`, change its Material into **Sprite-Unlit-Default**. You can see that part of the background corresponding to `bg-2(1)` isn't affected by the global light, and will just show its regular texture:

![lightdisable](https://www.dropbox.com/s/lb5cr4jgl10ivgx/8.png?raw=1)

The other background objects are affected by light because its material is set to **Sprite-Lit-Default**. Both **Sprite-Unlit-Default** and **Sprite-Lit-Default** are **Materials** that utilizes URP's **Shaders**. 

## Textures, Shaders, and Materials

Materials, Shaders, and Textures are three different things. We need the first two to render the Camera's view, and sometimes Textures as well if we do not want the object to simply contain solid colors. 

When we import images (.png, .jpeg, etc), we are creating **Textures**, also known as *bitmap images*.  Textures alone are not enough to dictate how Sprites or 3D Meshes should be rendered. 

We need **materials** for this. **Materials** are definitions of *how* a surface should be rendered, including references to textures used, tiling information, colour tints, *normal maps*, mask, and more. 

On the other hand, **Shaders** are small scripts that contain the **mathematical calculations** and algorithms for **calculating the colour of each pixel** rendered, based on the lighting input and the Material configuration. Therefore, it should be obvious that when we create materials, we need to specify the **Shader**. 

Let's create a *new Material* and demonstrate this knowledge. 
* Under Materials folder in your Assets, right click Create >> Material and name it GroundMaterial. 

* Set its shader to be Universal Render Pipeline/2D/Sprite-Lit-Default. Then, click on GameObject `tileset_15` and then, 
* Load this GroundMaterial under the Sprite Renderer component of `tileset_15`: the Material property. 

You will see that since GroundMaterial utilises the same shader as Sprite-Lit-Default material that comes with URP package, **there's no change in how the ground tiles look**. 

## Normal Mapping

When creating GroundMaterial, you might've seen a few properties that you can set, namely Normal Map, Mask, and Diffuse. We will briefly touch about Normal Map here. 

![normal](https://www.dropbox.com/s/pmxsenaj2duzfik/9.png?raw=1)

In Computer Graphics, normal mapping is a texture mapping technique used for faking the lighting of **bumps and dents**. A normal map contains normal vector values at each point of the texture, encoded into the RGB color, so you will commonly see them in blue-ish hue as such: 

![nmap](https://www.dropbox.com/s/yii9md00nu98wkp/NormalMap.png?raw=1)

The blue hue comes from the fact that the majority of the surface is flat, and therefore the normal vector is {0,0,1} (full Z-direction, where Z-direction with respect to the local frame of the surface refers to the perpendicular vector (the surface itself is considered to be on X-Y plane *local* to itself). Mapping this into RGB: {0,0,1} results in the color blue. 

### Normal Mapping Example
Take for example this Mario brick + normal map applied (left) vs regular Mario brick (right). Two kinds of normal maps were applied: the one that matches the brick surface (the normal map above), and another is a rough rock normal map (so it's more obvious).

![normal map](https://www.dropbox.com/s/4o9xbkb6ei9vh67/normalmap.gif?raw=1)


### Creating Normal Maps
Now let's apply normal map to our floor. Our floor looks flat currently, and we need to generate normal maps from the floor's tileset (can be found under GothicVania Church folder >> Environment). 

Let's use an online tool to generate the normal map. This online tool: https://cpetry.github.io/NormalMap-Online/ is absolutely awesome (otherwise you can use Photoshop to generate normal map from a given texture too). Load the tileset image and you should be able to view and download a normal map for it as such:

![nmapgen](https://www.dropbox.com/s/kb2gfwoz9d0w5kg/10.png?raw=1)

Store the tileset normal map as an asset inside your Unity Project, and **set its Texture Type to be "Normal map"**. There's two ways to apply this normal map to our floor. We can apply it directly on the tileset Texture as a secondary texture, OR create a new material and specifically load this normal map into the shader. The latter is simpler, so we will use that. 

Open the GroundMaterial you have created earlier, and **load** the tileset normal map under the **Normal Map property**:

![normalground](https://www.dropbox.com/s/ymy5lut1ulu6hpq/11.png?raw=1)

**Load `GroundMaterial`** as the Sprite Renderer's Material property on `tileset_15` GameObject. 

To view the Normal Map, we need a light source. Create a new **SpotLight2D** with **Normal Maps** property enabled, and set its distance into something small as shown:

![normalfloorenabled](https://www.dropbox.com/s/hp0s58a2aab2kja/12.png?raw=1)

Move the light around and you should be able to see the bumpy floor with normal map applied on it. 

# Shaders

Creating your own shader is no easy feat. It is one of the hardest thing to do in fact, but ShaderGraph in URP made it manageable. In this section, we are going to make **custom shaders**  (and post-processing effect) so that we can convert our character from looking like this: 

![noeffect](https://www.dropbox.com/s/r46zjruyqdv75n2/nofilter.gif?raw=1)

 To this:

![filtered](https://www.dropbox.com/s/a1zs8qg6bpjmlql/postprocessing.gif?raw=1)

## Creating a Basic Shader Graph
Under Assets, create a folder called Shader. Inside it, right click Create >> Shader >> Universal Render Pipeline >> Sprite Unlit Shader Graph. 

Name it: `glowGraph`. Double click the graph and you should be presented with the following graph editor UI:

![grapheditor](https://www.dropbox.com/s/dzthchmatvy2szs/13.png?raw=1)

**The way we encode logic to the Shader is via visual scripting.** Right now we only have two **nodes**, called **Vertex** and **Fragment**. The Shader that we are about to create will dictate how we should color a particular *fragment* (think of it as a single pixel) of our gameObject in our Scene (if any).   

### Vertex Shader and Fragment Shader
The figure below illustrates a simplistic summary of graphics pipeline:
![gpupipeline](https://www.dropbox.com/s/m1wjd0w2luykn7y/pipeline.png?raw=1)

In graphics pipeline, there are *two* types of shaders that are commonly customisable (well, actually *three*. The other one being geometry shader but this is not 50.017 so we are going to spare you from that), called v**ertex shader** and **fragment shader**. 

**Vertex Shader:** applied very early in the graphics pipeline, and its job is to **transform** each **vertex's** 3D position in *virtual space* (world space) to the **2D coordinate** at which it appears on the screen (as well as a **depth** value for the Z-buffer; so that the GPU knows what ought to be rendered in front of what). **Vertex shaders** can manipulate properties such as position, color and texture coordinates, but *cannot* create new **vertices**. A vertex is normally composed of *several* **attributes** (positions, normals, texture coordinates etc.).

**Fragment Shader:** applied later in the graphics pipeline, and it defines **RGBA** (red, green, blue, alpha) colors for each **pixel** being processed. It can be programmed to operate on a per pixel basis and takes into account *lighting* and *normal* mapping.

You can learn all about Shaders glory using various 2D and 3D rendering API, for example using OpenGL [here](https://learnopengl.com/Getting-started/Shaders), or using Vulkan [here](https://vulkan-tutorial.com/Drawing_a_triangle/Graphics_pipeline_basics/Shader_modules), or using Metal [here](https://www.raywenderlich.com/7475-metal-tutorial-getting-started). Writing shaders from scratch is one of the hardest thing to do in life, so we gotta be really grateful with the existence of visual scripting tools like URP which allow us to write shaders without having to be bothered with shading language syntaxes. 

> But this doesn't mean that you can forget your linear algebra and programming knowledge, or basic logic. 

```java
public  class ParallaxScroller : MonoBehaviour

{
	public  Renderer[] layers;
	public float[] speedMultiplier;
	private float previousXPositionMario;
	private float previousXPositionCamera;
	public Transform mario;
	public Transform mainCamera;
	private float[] offset;

	void  Start()
	{
		offset = new  float[layers.Length];
		for(int i = 0; i<  layers.Length; i++){
			offset[i] = 0.0f;	
		}
		previousXPositionMario = mario.transform.position.x;
		previousXPositionCamera = mainCamera.transform.position.x;
	}
}
```