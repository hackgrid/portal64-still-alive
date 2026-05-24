# enable_collision

Adds collision layers to [static collision](../level_objects/collision.md).

## Syntax

```
enable_collision COLLISION_NAME [LAYER]...
```

## Arguments

| Name               | Description                                                                            |
| ------------------ | -------------------------------------------------------------------------------------- |
| `COLLISION_NAME`   | The name of the static collision                                                       |
| `LAYER` (optional) | Space-separated list of collision layer names to add. Defaults to all layers.          |

## Notes

Static collision is not named by default. A name must be explicitly assigned in
order to control it with this cutscene step.
