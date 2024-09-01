---
description: This page describes how to create various objects that can be placed by player
---

# Placeables

## Placeable Object

{% hint style="info" %}
The [CoreLib.Entity submodule](../../modding-libraries/corelib.md) is highly recommended for this guide section.
{% endhint %}



A placeable object is a item with a graphical prefab that can be placed into the world. Player can interact with said object, and it can do various things in the world.

### Making the item

First, follow the steps in the generic item guide to define the other required properties of your entity.

{% content-ref url="../items/" %}
[items](../items/)
{% endcontent-ref %}

Set the `Object Type` to `Placeable Prefab`.&#x20;

Add following components:

* `Placeable Object Authoring`. This component allows to set various properties of the object. For example here you can make the object take up more than one tile. It also allows to set on what the object can be placed.
* `Physics Shape`. This will define the collision of the object
* `Ghost Authoring Component`. This is a component that controls your object ghost settings. Ghosts are a NetCode concept and really mean networked object. For most objects keep setting as defaults.
* `Mineable Authoring`
* `Health Authoring`. Set `Health` and `Max Health` to 2
* `Health Regeneration Authoring`
* `Damage Reduction Authoring`. Set `Max Damage Per Hit` to 1
* `State Authoring`
* `Idle State Authoring`
* `Death State Authoring`
* `Animation Authoring`
* `Ignore Vertex Offsets`

Optional components:

* `Interactable Authoring`. Enables your object to be interacted with. Ensure your graphical prefab has a Interactable Object component
* `Rotation Authoring`. Allows your object to be rotated
* `Electricity Authoring`. Allows your object to be powered and to power things
* `Supports Pooling` <mark style="color:orange;">(CoreLib)</mark>. Fixes issues you will encounter with Graphical Prefabs. HIGHLY recommended.

### Making the graphical prefab

Graphical prefab is a Game Object that represents the entity on screen. It is important to understand that Graphical prefabs live only on clients, and are only responsible for visual appearance of an entity. They should never encode core behavior of an entity. The only exceptions to this are UI and inventory logic.

To make a graphical prefab create a new Prefab. On the root of it you need to place a `EntityMonoBehaviour` deriven class.

Then in the hierarchy create a GameObject called `XScaler`. Then inside of it create two GameObjects: `SpriteObject`, `ShadowSprite` and `particleSpawnLocation`. Optionally you can create GameObject `Interactable`, if your object will need to be interacted with. The resulting prefab should look like this:

<figure><img src="../../../.gitbook/assets/simple-prefab-tree.png" alt=""><figcaption><p>Example graphical prefab</p></figcaption></figure>

On sprite GameObjects add a new component called SpriteObject.

Now create two `Sprite Assets` (Create -> 2D -> Sprite Asset), one for main sprite, one for shadow. In the main one assign your main texture, and in the shadow one assign a shadow texture. Now set these Sprite Assets as the `Asset` field on each of the sprites.

On the root component assign fields `X Scaler`, `Shadow` and set list Sprite Objects

<div data-full-width="false">

<figure><img src="../../../.gitbook/assets/sprite.png" alt="" width="410"><figcaption><p>One of the Sprite Objects set</p></figcaption></figure>

</div>

If you want the object to be rotatable create 3 more static variations: `up` (When facing away from camera), `side` (Facing right) and `side2` (Facing left). Assign their texture to relevant textures. To the root of the prefab add component `Sprite Variation From Entity Direction`. Assign both sprites.

<figure><img src="../../../.gitbook/assets/rotation-sa.png" alt="" width="410"><figcaption></figcaption></figure>

If you want the object to be interactable, on the `Interactable` GameObject a new component `Interactable Object`. Assign field `Entity Mono`, set sprites in `Sprite Objects` field (Except shadow) and wire up Unity Events `On Use Actions` and `On Trigger Exit Actions` to your root component methods. On the root component assign its field `Interactable`.&#x20;

<figure><img src="../../../.gitbook/assets/interactable.png" alt="" width="274"><figcaption><p>Interactable Object</p></figcaption></figure>

At the end the root of the prefab should look something like this:

<figure><img src="../../../.gitbook/assets/vp-root.png" alt="" width="274"><figcaption><p>Graphical Prefab Root</p></figcaption></figure>

Once you are done with the prefab, link it to the `Object Authoring` component.

Also don't forget to add following code to `ModObjectLoaded()` method in order to fix issues caused by lack of pooling support by default: <mark style="color:orange;">(CoreLib)</mark>

```csharp
if (obj is not GameObject gameObject) return;

var entityMono = gameObject.GetComponent<EntityMonoBehaviour>();
if (entityMono != null)
{
    EntityModule.EnablePooling(gameObject);
}
```