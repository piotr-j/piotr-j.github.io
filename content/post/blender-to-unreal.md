+++
description = ""
author = "PiotrJ"
tags = ["unreal", "blender"]
date = "2022-10-31T08:00:00+02:00"
title = "Blender to Unreal animation export"
+++

I'm using Blender 3.2.1 and Unreal Engine 4.27.2. I'm no expert in either, so it is quite possible that there are easier or better ways to achieve the same result. 


Let's start by making some very simple model in blender that we will animate and export. 
Before we actually model anything we will set scene unit scale to 0.01, so it matches Unreals 1cm/unit.
![Blender unit setup](/img/blender-to-unreal/blender-units-setup.png)

Additionally Clip Start and End should be adjusted or stuff will be cut off.
![Blender view setup](/img/blender-to-unreal/blender-view-settings.png)

We will use a simple 1x2x1m boxy chest with a lid that we will animate later
![Blender box model](/img/blender-to-unreal/blender-box.png)

On Unreal side we will use fresh Third Person Template project with a scaled Cube that will serve as something to compare our exported model to.
![Unreal project setup](/img/blender-to-unreal/unreal-setup.png)

Lets export our model to see if the dimensions match. From Blender export menu we will select FBX.
We will use following settings:
![Blender export include](/img/blender-to-unreal/blender-export-settings.png)

- Include
  - `Selected Objects` mainly useful when you have multiple models in one .blend file
  - `Object types`, we only need mesh and armature to start. Empty will be used for LoD import
  - `Custom properties` will be needed for LoD import
- Transform
  - Defaults are fine here, however it might be needed to change Forward/Up values if your model ends up with unexpected orientation in Unreal. Quite likely when LoDs are involved
- Geometry
  - We change `Smoothing` to `Face` to fix import warnning in Unreal
- Armature
  - we do want `Only Deform Bones` 
  - and we don't need extra `leaf bones`
- Bake Animation
  - we don't need `NLA strips`
  
Export the model somewhere and import in Unreal with default options, it will be setup as regular static mesh at this point. Easiest way to get there is to save it to somewhere in Content directory so it is picked up automatically.
![Unreal import settings](/img/blender-to-unreal/unreal-import-mesh.png)

Once it is imported we can put it next to our fake chest to see how it looks like. Size matches perfectly! We will deal with materials next time...
![Unreal chest in level](/img/blender-to-unreal/unreal-imported-mesh.png)


Now we can add some animations to see how to deal with them. First lets add armature to the scene and parent our chest to it. We use `empty groups` as automatic weights work best for organic shapes.
![Blender parent chest to armature](/img/blender-to-unreal/attach-armature-to-chest.png)

Two bones should be sufficient, one for the box and another for the lid. 
Make sure to assign vertex groups to parts of the mesh so bones influence it. Vertex Group name must match the bone name.
![Blender chest vertex groups](/img/blender-to-unreal/blender-vertex-groups.png)

To demonstrate animation blending we will make the chest bounce abit by default. We will do that in Action Editor as we will have multiple animations. Make sure to check the Fake User toggle (shield) 
or your animation might be removed after you close blender!
![Blender chest animation idle](/img/blender-to-unreal/blender-box-anim-idle.png)

For open animation we will key the lid bone.
![Blender chest animation open](/img/blender-to-unreal/blender-box-anim-open.png)


Now lets export it again, this time with animations! Replacing static mesh file with skeletal one will probably go on unnoticed by unreal, best to export it with new name or cleanup preview export. 
Make sure to check 'import animations'. To update existing animations export again to same filename.
![Unreal import skeletal mesh](/img/blender-to-unreal/unreal-import-skeletal-mesh.png)

Adding new animations is quite annoying, as they do not seem to be picked up when reimporting. You have to export with different filename and during import make sure to use previous skeletal mesh so they can be used wiht it. Remove duplicate content.
![Unreal reimport skeletal mesh](/img/blender-to-unreal/unreal-reimport-skeletal-mesh.png)

After all the imports we end up with a bunch of assets, skeletal mesh, animations, skeleton. 
![Unreal reimport skeletal mesh](/img/blender-to-unreal/unreal-imported-assets.png)

With simple Animation Blueprint we can make the chest open and close when pawn gets near by playing relevant animation.
![Unreal open chest](/img/blender-to-unreal/unreal-anim.png)

## Bonus! LoDs

As a bonus, lets setup Level of Detail meshes for our chest! For clarity we will use a shpere as LoD 1 for our model. 
We need to do few things, first is a common parent for our meshes, LoDs in the screenshot. 
Make sure that Lod1 mesh also has same armature modifier as main mesh. The LoD mesh order is dependant on order they ware parented in. So parent them to the LoDs objects one by one, first LoD0, then LoD1 etc. 
![Blender LoD setup](/img/blender-to-unreal/blender-lod-group.png)

Next we need to add a Custom Property to that parent so Unreal picks it up as a LoD group.
Press 'New' and add a String property named 'fbx_type' with value 'LodGroup'. 
![Blender LoD setup](/img/blender-to-unreal/blender-custom-prop.png)

Once all of that is done, we can export. Make sure to check 'Custom properties'!
It is likely that mesh ends up rotated after import. In that case you can try `Apply transform` (but that may break animations) or play with axis during export to fix that. For whatever reason the `Empty` object breaks stuff, feels like it is attached to first bone or something. Perhaps it would be just fine if first bone had default direction?
As with other things, simple remimport will probably not work. But this time it is fairly easy, in skeletal mesh lod settings select `import lod1` and pick the newly exported file.
![Unreal import lod1](/img/blender-to-unreal/unreal-import-lod1.png)

Adding LoD meshes directly to Armature does not work, they end up merged. LoDs do not have to be parrented to Armature for this to work, Empty still breaks stuff.

Here is the final result!
{{< youtube APPZ4i1Ksus >}}
