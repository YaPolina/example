
# daAnimations

- [Animations and nodeMasks](#animations-and-nodemasks)
- [About controllers](#about-controllers)
  - [List of controllers](#list-of-controllers)
    - [null](#null)
    - [fifo3](#fifo3)
    - [randomSwitch](#randomswitch)
    - [paramSwitch](#paramswitch)
    - [hub](#hub)
    - [align](#align)
    - [moveNode](#movenode)
    - [rotateNode](#rotatenode)
    - [linearpoly](#linearpoly)
    - [condHide](#condhide)
    - [attachNode](#attachnode)
    - [paramsCtrl](#paramsctrl)
    - [effFromAttachment](#efffromattachment)
    - [setParam](#setparam)
- [States, channels, etc.](#states-channels-etc)
  - [States include](#states-include)
     - [chan](#chan)
     - [state](#state)
  



# Animations and nodeMasks
Animations are made up of a2d (animation files) and settings (see [animation export](https://info.gaijin.lan/display/DAG/animation+export)).
The animation's use within the engine is outlined in the AnimTree of the relevant unit. The AnimTree includes descriptions of the AnimBlendNodeLeaf nodes. These nodes must bear the name of the animation file (a2d:t).
Nodes can accommodate animations of four different types:
- single - plays the animation once from start to finish
- continuous - plays the animation in a loop
- still - one frame of animation
- parametric - plays the animation depending on the state of a specified variable
Nodes use parameters:

| Name            | Description         | Default Value | Used by Animations      |
|-----------------|----------------------|---------------|-------------------------|
| name:t          | animation file name   | | all                     |
| key:t           | prefix of animation start and end keys in the a2d file; | "&lt;name&gt;"      | all                     |
| key_start:t     | name of the start key of the animation in the a2d file (presence of this parameter cancels out key:t) | "<key>_start" | single, continuous, parametric |
| key_end:t       | name of the end key of the animation in the a2d file (presence of this parameter cancels out key:t)| "<key>_end"   | single, continuous, parametric |
| start:i         | start frame number for the animation; if greater than 0, the value of "key_start:t" is ignored; <br> for **still** animations, similarly but for "key:t" | -1            | single, continuous, still |
| end:i           | last frame number for the animation; if greater than 0, the value of "key_end:t" is ignored  | -1            | single, continuous, still |
| own_timer:b     | animation uses its own (true) or global (false) timer   | false         | single, continuous     |
| timer:t         | name of the global timer, only for own_timer == false    | "::GlobalTime"| single, continuous     |
| time:r          | animation duration  | 1.0           | single, continuous     |
| labels          | blocks for creating named animation intervals  | -             | single, continuous     |
| eoa_irq:b       | creates an irq when the animation ends  | false         | continuous              |
| rand_start:b    | the animation starts playing from a random position  | false         | continuous              |
| start:i         | frame number for the animation; if greater than 0, the value of "key:t" is ignored      | -1            | still                   |
| varname:t       | name of the parameter variable for the animation   |     | parametric                      |
| p_start:r       | minimum value of the parameter variable for the animation  | 0.0           | parametric              |
| p_end:r         | maximum value of the parameter variable for the animation  | 1.0           | parametric              |
| addk:r          | additional coefficient and multiplier to the varname value (p*mulk + addk) | 0.0           | parametric              |
| mulk:r          | additional coefficient and multiplier to the varname value (p*mulk + addk)  | 1.0           | parametric              |


**Example**
```
AnimBlendNodeLeaf {
    a2d:t="animation__file.a2d"
    continuous {
        name:t="animation_name"
        key_start:t="start"
        key_end:t="end"
        time:r=1
        own_timer:b=yes
        eoa_irq:b=yes
        addMoveDirH:r=-90
        addMoveDist:r=1.33
    }
}
```

**IRQ** - interrupt events, are required to notify or request certain events (such as sounds or damage) from the game code. For **single, parametric, continuous** animations, interrupts can be defined using IRQ blocks. These blocks typically indicate the name and the moment of interruption. The moment of interruption can be specified using one of the following parameters: key:t (key name or an integer as a string), relPos:r, or keyFloat:r (in seconds).
```
irq{
 key:t="end" /*key name, can also be a number (as a string)*/
 name:t="arize_end"; /* name of the irq event that will be sent to the engine */
}
```


# About controllers

There are various controllers - **paramswitch, randomswitch, linearPoly, align, rotate, fifo3, hub, null**. 

Controllers are described in a block
```
AnimBlendCtrl{
}
```

All controllers have a name **name:t**.

In the character (at the root of the block), the parameter **root:t="&lt;controller name&gt;** must be specified to define the root of our animation tree.

Animations are combined in the **fifo3** controller as they are set. Controllers (including **fifo3**) are combined in the **hub** controller of type a.

**An example of a block for a character**

```
root:t="root"
AnimBlendCtrl{
    fifo3{
        name:t="fifo3"
        varname:t="fifo"
    }
    null{
        name:t="null"
    }
    hub{
        name:t="root"
        const:b=yes
        child{
            name:t="fifo3"
            weight:r=1
            enabled:b=yes
        }
    }
}

```

## List of controllers:

### null

Empty, no animations.

```
null { name:t="~~car_aim"; }
```

### fifo3

Combines up to three animations at a time.

```
fifo3{
 name:t="fifo3_body"
 varname:t="fifo_body"
}
```

### randomSwitch

Selects one of the random animations specified in it.

```
randomSwitch{
 name:t="~~knife_melee"
 weight{
 knife_melee:r=1 // random weight
 knife_melee_2:r=1
 }
}
```

### paramSwitch

Selects one of the animations by parameter.

```
paramSwitch {
 name:t="head_look_switch"
 varname:t="have_head_look_at"
 nodes{
 node { name:t="null"; rangeFrom:r=-1; rangeTo:r=0.5; } // if have_head_look_at is from -1 to 0.5 this animation will be selected
 node { name:t="head_look"; rangeFrom:r=0.5; rangeTo:r=1.5; } // if have_head_look_at is from 0.5 to 1.5 this animation will be selected
 }
}
```

### hub

Combines all controllers and animations, weights for each can be specified.

```
hub{
 name:t="root"
 const:b=yes
 child{
 name:t="~~rudder_move"
 weight:r=1
 enabled:b=yes
 }
 child{ name:t="fifo3_arms"; }
}
```

### align

Assigns animation to another node (the service name **::root** indicates the scene root), with the option to specify the axis, its direction and rotation.

### moveNode

Moves a node along the specified axis

```
moveNode{
 name:t="~~rudder_move"
 targetNode:t="rudder"
 param:t="rudder_move"
 dirAxis:p3=0, 1, 0
}

```

### rotateNode

This function rotates the node to a specified angle, controlled by a float parameter. If multiple nodes are indicated, each one rotates proportionally. The total rotation is equal when all rotations are combined. This is useful for elements such as bones in a skeleton. The function includes a parameter.
```
rotateNode{
 name:t="~~neck_rotate_yaw"
 targetNode:t="Bip01 Neck"
 kMul:r=1
 kAdd:r=0
 param:t="torso_rotate"
 axis_course:t="pers_course" //if specified, then relative to the character
 kCourseAdd:r=0
 rotAxis:p3=0, 0, 1
 dirAxis:p3=0, 0, 1
}
```

### linearpoly

Linear blending of animations by variable value (varname:t).

```
linearPoly{
 name:t="walk"
 varname:t="local_walk_dir"
 morphTime:r=0.3
 child{ name:t="walk_backward"; val:r=-180; }
 child{ name:t="walk_left"; val:r=-90; } // with local_walk_dir == 135, 50/50 walk_backward and walk_left will be mixed
 child{ name:t="walk_forward"; val:r=0; }
 child{ name:t="walk_right"; val:r=90; }
 child{ name:t="walk_backward"; val:r=180; }
}
```

### condHide

Hides the node according to the conditions.

```
condHide{
 name:t="rudder_hide~~"
 targetNode { node:t="rudder_r"; param:t="rudder"; op:t=">"; p0:r=10; }
}
// options for comparison operations - =, !- , >, <

```

### attachNode

Simply copies the world matrix from one node to another

### paramsCtrl

Automatic (time-based) control of other controllers (rotation, for example)

```
paramsCtrl {
 name:t="#rot_prop"
 changeParam { param:t="propeller_rotate"; changeRate:r=6; mod:r=360; variableScale:b=yes; }
}

```

### effFromAttachment

This function copies a position from an attached animated skeleton to a specified effector.

```
effFromAttachment{
 name:t="arms_aim_set_effectors_l"
 slot:t="weapon" // The slot to which the original skeleton is attached
 node {
 node:t="ik_hand_left"; // src node in the attached skeleton
 effector:t="l.hand.aim"; // The name of the effector where the position is set
 destNode:t="Bip01 L Hand"; // dest node in the original skeleton
 wtModulate:t="l.hand_ik_mul"; // The name of the variable where the value for modulation is taken from
 }
}
// writeMatrix:b (for the "node" block) - take not only the position and basis but also write to the variable with the effector name
// with the .m postfix (for example, l.hand.att.m). If the parameter is absent, the matrix is written!
// localNode - the node will be taken from the original skeleton, ignoring the slot
// ignoreZeroWt -
```


### setParam

Passes the parameter value to another AnimChar.

```
setParam{
  name:t="#set_weapon_state"
  destSlot:t="weapon" // slot in which another animchar is located
  destVar:t="~~weapon_state" // variable in the animtree of another animchar
  srcVar:t="weapon_state" // variable that will be passed
}
```

If "destSlot" doesn't exist, a variable named "destVar" will be created in the current animchar, and the data will be assigned to it.

A specific value can be provided by replacing "srcVar" with "value:r=0".

Instead of "srcVar," "namedValue" can also be specified. In this case, the numerical value from the specified enum in "namedValue" (namedValue:t="enumName") will be passed.

```
enum {
 weapon_states {
 weapon_null {}
 aim {}
 sprint {}
 }
}
setParam{
 name:t="#set_weapon_aim"
 destVar:t="weapon_state"
 namedValue:t="aim"
}
//same as below, but without magic constants
setParam{
 name:t="#set_weapon_aim"
 destVar:t="weapon_state"
 value:r=1
}
```

You can view and manipulate the parameters of controllers in the debug window of the page, directly from the client code.

For each controller, there is a parameter named **accept_name_mask_re:t**. While this parameter doesn't affect the visual representation, it optimizes the generation of bnl based on nodeMasks that match the regular expression.



# States, channels, etc.

States are configurations that incorporate multiple animations and controllers simultaneously. 
They are primarily used to streamline work for animators and programmers and to simplify debugging of complex animations on a page.

States are described in a block

```
stateDesc{
}
```

## States include

defMorphTime:r=0.15 - default morph time
<br/> 
defNodeName:t="" - below <br/> 
"*" - means not to change <br/> 
"" - means to remove the animation from there <br/> 
the rest - the name of the controller or animation

If a channel in the state doesn't have a specified action, the default action will be used.

### chan

The "chan {} channels" is a FIFO3 controller used for combining one channel into another, particularly for transitions. It requires a NodeMask name and a channel name.

```
chan{
 name:t="body"
 fifo3:t="fifo3"
 nodeMask:t=""
}
chan{
 name:t="additive_body_movements"
 fifo3:t="fifo3_additive_body_movements"
 nodeMask:t=""
}
```

### state

A state consists of a name and guidelines for arranging content within particular channels.

```
state{
 name:t="blinking"
 body{ name:t="blinking"; }
 additive_body_movements { name:t="emotions_additive"; }
} 
```

Morph time can be set per state (morphTime:r).

