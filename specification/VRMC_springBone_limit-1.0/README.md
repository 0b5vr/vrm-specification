# VRMC_springBone_limit-1.0

*Version 1.0-draft*

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Contributors](#contributors)
- [Status](#status)
- [Dependencies](#dependencies)
- [Overview](#overview)
  - [Limits](#limits)
    - [Cone Limit](#cone-limit)
    - [Hinge Limit](#hinge-limit)
    - [Spherical Limit](#spherical-limit)
    - [Singular Directions](#singular-directions)
  - [Rotation](#rotation)
  - [Limit Application Order](#limit-application-order)
- [glTF Schema Updates](#gltf-schema-updates)
  - [Extending Springs](#extending-springs)
  - [VRMC_springBone_limit](#vrmc_springbone_limit)
    - [Properties](#properties)
    - [JSON Schema](#json-schema)
    - [VRMC_springBone_limit.specVersion ✅](#vrmc_springbone_limitspecversion-)
    - [VRMC_springBone_limit.limit ✅](#vrmc_springbone_limitlimit-)
  - [Limit](#limit)
    - [Properties](#properties-1)
    - [JSON Schema](#json-schema-1)
    - [Limit.cone](#limitcone)
    - [Limit.hinge](#limithinge)
    - [Limit.spherical](#limitspherical)
  - [ConeLimit](#conelimit)
    - [Properties](#properties-2)
    - [JSON Schema](#json-schema-2)
    - [ConeLimit.angle ✅](#conelimitangle-)
    - [ConeLimit.rotation](#conelimitrotation)
  - [HingeLimit](#hingelimit)
    - [Properties](#properties-3)
    - [JSON Schema](#json-schema-3)
    - [HingeLimit.angle ✅](#hingelimitangle-)
    - [HingeLimit.rotation](#hingelimitrotation)
  - [SphericalLimit](#sphericallimit)
    - [Properties](#properties-4)
    - [JSON Schema](#json-schema-4)
    - [SphericalLimit.pitch ✅](#sphericallimitpitch-)
    - [SphericalLimit.yaw ✅](#sphericallimityaw-)
    - [SphericalLimit.rotation](#sphericallimitrotation)
- [Appendix: Reference Implementations](#appendix-reference-implementations)
  - [Overview](#overview-1)
  - [Rotation](#rotation-1)
  - [ConeLimit](#conelimit-1)
  - [HingeLimit](#hingelimit-1)
  - [SphericalLimit](#sphericallimit-1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Contributors

- 0b5vr

## Status

Draft

## Dependencies

Written against the glTF 2.0 specification.

This specification depends on [`VRMC_springBone`](https://github.com/vrm-c/vrm-specification/blob/master/specification/VRMC_springBone-1.0/README.md).

## Overview

`VRMC_springBone_limit` is a glTF extension that limits the range of motion of springs controlled by `VRMC_springBone`.
This extension defines cone, hinge, and spherical limits.

### Limits

This extension defines three types of limits: cone, hinge, and spherical limits.

Reference implementations for each limit are provided in [Appendix: Reference Implementations](#appendix-reference-implementations).

#### Cone Limit

A cone limit constrains the range of motion of a spring to a cone.

A cone limit is defined by an angle representing the spread of the cone and a rotation representing the orientation of the cone.

The rotation of a spring with a cone limit is constrained so that it does not tilt from the Head-to-Tail direction by more than the angle specified for the cone.

If no limit rotation is specified, the cone is defined to spread in the Head-to-Tail direction.

![Diagram of a cone limit](./figures/cone-limit.png)

#### Hinge Limit

A hinge limit constrains the range of motion of a spring to a hinge.

A hinge limit is defined by an angle representing the spread of the hinge and a rotation representing the orientation of the hinge.

The rotation of a spring with a hinge limit is constrained so that it does not tilt from the Head-to-Tail direction by more than the angle specified for the hinge and does not rotate about any axis other than the rotation axis defined by the hinge.

If no limit rotation is specified, the hinge is defined by first creating a shape that spreads in the positive y-axis direction and permits rotation about the x-axis, and then rotating it along the shortest path so that the hinge spreads in the Head-to-Tail direction.

![Diagram of a hinge limit](./figures/hinge-limit.png)

#### Spherical Limit

A spherical limit constrains the range of motion of a spring to a spherical region.

A spherical limit is defined by two angles, Pitch and Yaw, and a rotation representing the orientation of the sphere.

The rotation of a spring with a spherical limit is constrained so that it does not tilt from the Head-to-Tail direction by more than the Pitch and Yaw values specified by the parameters.

If no limit rotation is specified, the spherical limit is defined by first creating a spherical limit relative to the positive y-axis direction, where rotation about the x-axis is Pitch and rotation about the z-axis is Yaw, and then rotating it along the shortest path so that the Head-to-Tail direction becomes the reference direction.

![Diagram of a spherical limit](./figures/spherical-limit.png)

The following diagram illustrates Pitch and Yaw, the two angles specified for a spherical limit.

![Diagram of Pitch and Yaw for a spherical limit](./figures/spherical-limit-pitch-yaw.png)

#### Singular Directions

If the constrained direction cannot be uniquely determined from the Tail direction, implementations **MUST** select the direction shown in the following table, expressed in the limit's local coordinate system.

| Limit | Tail direction | Constrained direction |
|:-|:-|:-|
| Cone | Negative y-axis direction | Boundary on the positive z-axis side |
| Hinge | Positive or negative x-axis direction | Positive y-axis direction |
| Hinge | Negative y-axis direction | Boundary on the positive z-axis side |
| Spherical | Positive or negative x-axis direction | Boundary on the positive y-axis side |
| Spherical | Negative y-axis direction | Boundary on the positive z-axis side |

### Rotation

The orientation of each limit can be changed by modifying its `rotation` property.
The initial rotation of each limit is obtained by rotating its shape, which is initially oriented relative to the positive y-axis direction, along the shortest path so that the Head-to-Tail direction becomes the reference direction. The applied orientation is the result of applying `rotation` in the local coordinate system of that initial rotation.

![Animation showing the rotation order](./figures/rotation.gif)

If the Head-to-Tail direction is exactly the negative y-axis direction, the shortest-path rotation cannot be uniquely determined. In this case, implementations **MUST** use a 180-degree rotation about the x-axis as the initial rotation.

> Results may be unstable across implementations if the Head-to-Tail direction is exactly or nearly the negative y-axis direction. Artists are advised to avoid directions close to the negative y-axis direction whenever possible when applying a Limit to a SpringBone Joint.

A detailed reference implementation of rotation is provided in [Appendix: Reference Implementations](#appendix-reference-implementations).

### Limit Application Order

When angular constraints by a limit are enabled, they **SHOULD** be applied at all of the following times:

- Immediately after the inertia calculation
- Immediately after each collision with a collider

> Implementations may reduce the frequency at which limits are applied or disable angular constraints by limits depending on the target platform, runtime settings, LOD, or other considerations. In such cases, the final Tail direction produced at runtime is not guaranteed to remain within the limit's range.

## glTF Schema Updates

### Extending Springs

A limit is specified by adding the `VRMC_springBone_limit` extension to a joint defined by `VRMC_springBone`.

Because the last joint in the `VRMC_springBone.springs[*].joints` array is used only as a Tail by `VRMC_springBone`, implementations **MUST** ignore this extension if it is specified on the last joint.
Exporters **MUST NOT** write this extension to the last joint.

```json
{
  "extensionsUsed": [
    "VRMC_springBone",
    "VRMC_springBone_limit"
  ],
  "extensions": {
    "VRMC_springBone": {
      "specVersion": "1.0",
      "springs": [
        {
          "joints": [
            {
              "node": 0,
              "hitRadius": 0.25,
              "stiffness": 1.0,
              "dragForce": 0.4,
              "extensions": {
                "VRMC_springBone_limit": {
                  "specVersion": "1.0-draft",
                  "limit": {
                    "cone": {
                      "angle": 0.785398,
                      "rotation": [ 0.0, 0.0, 0.0, 1.0 ]
                    }
                  }
                }
              }
            },
            // ...
          ]
        },
        // ...
      ],
      // ...
    }
  },
  // Standard glTF 2.0 data
  "nodes": [
    // ...
  ]
}
```

### VRMC_springBone_limit

The root object of this extension.

{
  "$schema": "http://json-schema.org/draft-04/schema",
  "title": "VRMC_springBone_limit",
  "type": "object",
  "description": "An angle limit for VRMC_springBone.",
  "allOf": [ { "$ref": "glTFProperty.schema.json" } ],
  "properties": {
    "specVersion": {
      "type": "string",
      "description": "Specification version of VRMC_springBone_limit."
    },
    "limit": {
      "$ref": "VRMC_springBone_limit.limit.schema.json"
    },
    "extensions": { },
    "extras": { }
  },
  "required": [ "specVersion", "limit" ]
}


#### Properties

|| Type | Description | Required |
|:-|:-|:-|:-|
|`specVersion`|`string`|The version of this extension|✅ Yes|
|`limit`|[Limit](#limit)|The limit definition|✅ Yes|

#### JSON Schema

[VRMC_springBone_limit.schema.json](schema/VRMC_springBone_limit.schema.json)

#### VRMC_springBone_limit.specVersion ✅

The version of the `VRMC_springBone_limit` extension. The value **MUST** be `"1.0-draft"`.

- Type: `string`
- Required: Yes

#### VRMC_springBone_limit.limit ✅

Defines the limit applied to the spring.

- Type: [Limit](#limit)
- Required: Yes

### Limit

Defines the limit applied to the spring.

It **MUST** contain exactly one of `cone`, `hinge`, or `spherical`.

#### Properties

|| Type | Description | Required |
|:-|:-|:-|:-|
|`cone`|[ConeLimit](#conelimit)|Cone limit|No|
|`hinge`|[HingeLimit](#hingelimit)|Hinge limit|No|
|`spherical`|[SphericalLimit](#sphericallimit)|Spherical limit|No|

#### JSON Schema

[VRMC_springBone_limit.limit.schema.json](schema/VRMC_springBone_limit.limit.schema.json)

#### Limit.cone

Defines a cone limit.

- Type: [ConeLimit](#conelimit)
- Required: No

#### Limit.hinge

Defines a hinge limit.

- Type: [HingeLimit](#hingelimit)
- Required: No

#### Limit.spherical

Defines a spherical limit.

- Type: [SphericalLimit](#sphericallimit)
- Required: No

### ConeLimit

Defines a cone limit.

#### Properties

|| Type | Description | Required |
|:-|:-|:-|:-|
|`angle`|`number`|The angle of the cone limit, in radians|✅ Yes|
|`rotation`|`number[4]`|The relative rotation of the cone limit|No|

#### JSON Schema

[VRMC_springBone_limit.coneLimit.schema.json](schema/VRMC_springBone_limit.coneLimit.schema.json)

#### ConeLimit.angle ✅

The angle of the cone limit. The angle is expressed in radians and **MUST** be greater than or equal to zero.
If the angle is set to π or greater, implementations **MUST** interpret the angle as π.
When the angle is set to π, the cone shape is equivalent to a sphere.

- Type: `number`
- Required: Yes

#### ConeLimit.rotation

The relative rotation from the default orientation of the cone limit.
The rotation is expressed as a quaternion (x, y, z, w), where w is the scalar component.
The quaternion **MUST** be a unit quaternion.

- Type: `number[4]`
- Required: No
- Default: `[ 0.0, 0.0, 0.0, 1.0 ]`

### HingeLimit

Defines a hinge limit.

#### Properties

|| Type | Description | Required |
|:-|:-|:-|:-|
|`angle`|`number`|The angle of the hinge limit, in radians|✅ Yes|
|`rotation`|`number[4]`|The relative rotation of the hinge limit|No|

#### JSON Schema

[VRMC_springBone_limit.hingeLimit.schema.json](schema/VRMC_springBone_limit.hingeLimit.schema.json)

#### HingeLimit.angle ✅

The angle of the hinge limit. The angle is expressed in radians and **MUST** be greater than or equal to zero.
If the angle is set to π or greater, implementations **MUST** interpret the angle as π.
When the angle is set to π, the hinge shape is equivalent to a disk.

- Type: `number`
- Required: Yes

#### HingeLimit.rotation

The relative rotation from the default orientation of the hinge limit.
The rotation is expressed as a quaternion (x, y, z, w), where w is the scalar component.
The quaternion **MUST** be a unit quaternion.

- Type: `number[4]`
- Required: No
- Default: `[ 0.0, 0.0, 0.0, 1.0 ]`

### SphericalLimit

Defines a spherical limit.

#### Properties

|| Type | Description | Required |
|:-|:-|:-|:-|
|`pitch`|`number`|The Pitch of the spherical limit, in radians|✅ Yes|
|`yaw`|`number`|The Yaw of the spherical limit, in radians|✅ Yes|
|`rotation`|`number[4]`|The relative rotation of the spherical limit|No|

#### JSON Schema

[VRMC_springBone_limit.sphericalLimit.schema.json](schema/VRMC_springBone_limit.sphericalLimit.schema.json)

#### SphericalLimit.pitch ✅

The Pitch angle of the spherical limit. The angle is expressed in radians and **MUST** be greater than or equal to zero.
If the angle is set to π or greater, implementations **MUST** interpret the angle as π.

- Type: `number`
- Required: Yes

#### SphericalLimit.yaw ✅

The Yaw angle of the spherical limit. The angle is expressed in radians and **MUST** be greater than or equal to zero.
If the angle is set to π/2 or greater, implementations **MUST** interpret the angle as π/2.

- Type: `number`
- Required: Yes

#### SphericalLimit.rotation

The relative rotation from the default orientation of the spherical limit.
The rotation is expressed as a quaternion (x, y, z, w), where w is the scalar component.
The quaternion **MUST** be a unit quaternion.

- Type: `number[4]`
- Required: No
- Default: `[ 0.0, 0.0, 0.0, 1.0 ]`

## Appendix: Reference Implementations

> *This section is non-normative.*

The following sections provide reference implementations for each limit defined by this extension.
See also the reference implementation in the `VRMC_springBone` specification.

### Overview

As described in [Limit Application Order](#limit-application-order), angular constraints by a limit should be applied at all of the following times:

- Immediately after the inertia calculation
- Immediately after each collision with a collider

Accordingly, the value to which the angular constraint should be applied is `nextTail`, the world-space position of the child Node targeted by the Joint, which is used during the SpringBone inertia calculation and collision detection with colliders.
Immediately after the inertia calculation and after collision detection with each collider, apply the limit so that the direction of `nextTail` remains within the specified angular range.
This is the same point at which the distance of `nextTail` from the Joint position is constrained.

In the following reference implementations, `tailDir` is a normalized three-dimensional vector representing the world-space direction of the joint being constrained.
It is defined using `nextTail`, as shown in the following pseudocode.

```ts
var tailDir = (nextTail - joint.worldPosition).normalized;
```

> For brevity, the following pseudocode does not account for floating-point errors.
> Implementations are advised to take appropriate precautions against numerical errors according to the precision of the numeric types used, such as using tolerances when testing for singular directions and clamping intermediate values to the domains of operations such as `acos` and `asin`.

### Rotation

The following pseudocode is a reference implementation for rotating each limit using the Head-to-Tail direction and the `rotation` property.
`boneAxis` represents the direction in which the child Node targeted by the Joint extends in its rest state, expressed in local space.

```ts
// The shortest rotation from the positive y-axis direction to the vector from the joint's Head to its Tail
let axisRotation;

// The dot product of the Head-to-Tail vector and the positive y-axis direction
let dot = boneAxis.y;

if (dot == -1.0) {
  // If the Head-to-Tail vector points in the negative y-axis direction, set a rotation of 180 degrees about the x-axis
  axisRotation = Quaternion(1, 0, 0, 0);
} else {
  // Otherwise, set the shortest rotation from the positive y-axis direction to the joint's Head-to-Tail vector
  // quaternion(cross(from, to); dot(from, to) + 1).normalized
  axisRotation = Quaternion(boneAxis.z, 0, -boneAxis.x, dot + 1).normalized;
}

// The rotation that maps the limit's local space to world space
let rotation = joint.parent.worldRotation * joint.localSpaceInitialRotation * axisRotation * joint.limit.rotation;

// Map the Tail direction to the limit's local space
tailDir = tailDir.applyQuaternion(rotation.inverse);

// Apply the limit
joint.limit.apply(tailDir);

// Map the Tail direction back to world space
tailDir = tailDir.applyQuaternion(rotation);
```

### ConeLimit

The following is a reference implementation of a cone limit in pseudocode.

```ts
// Clamp angle to the range from 0 to π
let limitAngle = clamp(limit.angle, 0.0, PI);

// Compare the y component of tailDir with the cosine of the angle specified by the limit
let cosLimitAngle = cos(limitAngle);
if (tailDir.y < cosLimitAngle) {
  // Scale the x and z components by the ratio of the sine of tailDir to the sine of the angle specified by the limit
  let horizontalLengthSquared = 1.0 - tailDir.y * tailDir.y;

  if (horizontalLengthSquared == 0.0) {
    // If tailDir points in the negative y-axis direction, select the positive z-axis side
    tailDir.x = 0.0;
    tailDir.z = sqrt(1.0 - cosLimitAngle * cosLimitAngle);
  } else {
    let scale = sqrt((1.0 - cosLimitAngle * cosLimitAngle) / horizontalLengthSquared);
    tailDir.x *= scale;
    tailDir.z *= scale;
  }

  // Set the y component to the cosine of the angle specified by the limit
  tailDir.y = cosLimitAngle;
}
```

### HingeLimit

The following is a reference implementation of a hinge limit in pseudocode.

```ts
// Clamp angle to the range from 0 to π
let limitAngle = clamp(limit.angle, 0.0, PI);

let projectedLengthSquared = tailDir.y * tailDir.y + tailDir.z * tailDir.z;
if (projectedLengthSquared == 0.0) {
  // If tailDir points in the positive or negative x-axis direction, select the positive y-axis direction
  tailDir = vec3(0.0, 1.0, 0.0);
} else {
  // Project tailDir onto the hinge's YZ plane
  tailDir = vec3(0.0, tailDir.y, tailDir.z) / sqrt(projectedLengthSquared);

  // Compare the y component of tailDir with the cosine of the angle specified by the limit
  let cosLimitAngle = cos(limitAngle);
  if (tailDir.y < cosLimitAngle) {
    let sinLimitAngle = sqrt(1.0 - cosLimitAngle * cosLimitAngle);

    // If tailDir points in the negative y-axis direction, select the positive z-axis side
    let zSign = (tailDir.z < 0.0) ? -1.0 : 1.0;
    tailDir.y = cosLimitAngle;
    tailDir.z = sinLimitAngle * zSign;
  }
}
```

### SphericalLimit

The following is a reference implementation of a spherical limit in pseudocode.

```ts
// Clamp pitch to the range from 0 to π and yaw to the range from 0 to π/2
let limitPitch = clamp(limit.pitch, 0.0, PI);
let limitYaw = clamp(limit.yaw, 0.0, PI / 2.0);

// Calculate the pitch and yaw of tailDir
var pitch;
if (tailDir.y == -1.0) {
  // If tailDir points in the negative y-axis direction, set pitch to π to select the boundary on the positive z-axis side
  pitch = PI;
} else if (abs(tailDir.x) == 1.0) {
  // If tailDir points in the positive or negative x-axis direction, set pitch to 0
  pitch = 0.0;
} else {
  pitch = atan2(tailDir.z, tailDir.y);
}
var yaw = asin(tailDir.x);

// Constrain pitch using the pitch specified by the limit
if (abs(pitch) > limitPitch) {
  isLimited = true;
  pitch = limitPitch * sign(pitch);
}

// Constrain yaw using the yaw specified by the limit
if (abs(yaw) > limitYaw) {
  isLimited = true;
  yaw = limitYaw * sign(yaw);
}

// Recalculate tailDir using pitch and yaw
tailDir = vec3(
  sin(yaw),
  cos(yaw) * cos(pitch),
  cos(yaw) * sin(pitch)
);
```
