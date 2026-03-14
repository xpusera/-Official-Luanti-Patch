# How to Port Minecraft JSON Models to glTF

This guide explains how to convert Minecraft Bedrock JSON models into glTF format for use in Luanti Patch.

We use glTF instead of GLB because glTF is a readable text file. You can open it in any text editor, see all bone names and animation names, and fix problems easily. GLB is a binary format which is harder to work with even though it loads slightly faster.

## What You Need

You need [Blockbench](https://www.blockbench.net/) which is available on PC or directly in the browser. You also need the JSON model file and animation file from the Minecraft mod you want to port.

## Step 1. Open the Model

Open Blockbench and go to File select json then Open Model. Select the model.json file from the Minecraft mod. The model should load with all its bones visible in the outliner panel.

## Step 2. Import the Animations

This is the most important step. If you skip it your model will be completely static with no movement.

Go to the Animate tab at the top of Blockbench( if you cant see it, press the icon botton on the top bar "right" corner if you press it it will show a dropdown choose the "Animate"). Then go to File and click Import Animations. Select the animation.json file from the mod. You should see all animation clips appear in the timeline such as walk, idle and attack. Press play on each one to confirm they work correctly before continuing.

## Step 3. Export as glTF

Go to File then Export and choose Export glTF. Save the file and you will get a .gltf.txt file ready to use.

## Step 4. Use It In Your Mod

Place the gltf file inside your mod folder. To play a specific animation clip by name use `set_animation_clip`. The clip name comes from the animation file you imported in Blockbench.

```lua
obj:set_properties({ mesh = "my_model.gltf" })

-- play by clip name
obj:set_animation_clip("walk", { x=0, y=60 }, 30, 0.2, true)

-- play by clip index (0 is the first animation)
obj:set_animation_clip(0, { x=0, y=60 }, 30, 0.2, true)
```

To handle multiple animation states automatically use `core.animator`. It switches between clips based on conditions you define.

```lua
local anim = core.animator.create(obj, {
    initial = "idle",
    states = {
        idle   = { clip="idle",   range={x=0,y=60}, speed=30, loop=true,  blend=0.2  },
        walk   = { clip="walk",   range={x=0,y=40}, speed=30, loop=true,  blend=0.15 },
        attack = { clip="attack", range={x=0,y=20}, speed=30, loop=false, blend=0.1  },
    },
    transitions = {
        { from="idle", to="walk",   condition=function(ctx) return ctx.moving     end },
        { from="walk", to="idle",   condition=function(ctx) return not ctx.moving end },
        { from="*",    to="attack", condition=function(ctx) return ctx.attacking  end },
    },
    get_context = function(self, obj, dtime)
        return {
            moving    = obj:get_velocity() ~= vector.zero(),
            attacking = false,
        }
    end,
})

core.animator.register(anim)
