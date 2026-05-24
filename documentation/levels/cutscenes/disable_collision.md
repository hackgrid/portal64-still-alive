# disable_collision

Removes collision layers from [static collision](../level_objects/collision.md).

## Syntax

```
disable_collision COLLISION_NAME [LAYER]...
```

## Arguments

| Name               | Description                                                                            |
| ------------------ | -------------------------------------------------------------------------------------- |
| `COLLISION_NAME`   | The name of the static collision                                                       |
| `LAYER` (optional) | Space-separated list of collision layer names to remove. Defaults to all layers.       |

## Notes

Static collision is not named by default. A name must be explicitly assigned in
order to control it with this cutscene step.
