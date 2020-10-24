OutgoingLightUpdate
-----

This page relates specifically to `mx.kenzie.wellspring.packet.specific.OutgoingLightUpdate`.

### Components
1. Types
2. Sections
3. Masks
4. Manifolds

#### Types
Light is divided into two types: sky and block.
Sky light has a white hue. Block light has a yellow hue. Sky light displayed brightness is affected by the time of day.

#### Sections
This packet relates to a particular chunk, at a given x/z coordinates.

The chunk is divided into 18 sections. The first section is below the world (y -16 to -1). The subsequent 16 sections are the natural chunk sections, from y 0 to 255 inclusive. The final section is above the height limit, from y 256 to 271 inclusive.

Each section (a 16 by 16 cube) has two distinct strata: sky light and block light. These two are independent of one another.

Every position within a section can have a light value (0 to 15) for both sky and block light.

#### Masks
Masks are used to notify the client of whether to expect an update for a section.

There are two masks for both sky and block, making four masks in total.

The `empty` mask marks a section as having NO non-zero light in it for this type.
The other mask marks a section as having non-zero light in it. If this mask is marked, the client will expect an entry in the manifold.

Each mask is an integer. The 18 least significant bits of the integer are used as `1/0` binary boolean values.

*Nota Bene: entries are read from right-to-left.* 

The 18 least significant bits of a mask can be obtained using `mask & ((1 << 18) - 1))`.

*Exempli Gratia:*

Below is the mask for a chunk that has only one light entry, within the 0-15 section.
```
Mask: 000000000000000010
Empty mask: 111111111111111101
```
As you can see, all sections bar 0-15 are marked as empty. That section is marked as having a light present.

The 0-15 section is the SECOND section (remember: there is one below bedrock) - so it is the second entry from the right.

>To mark a section as true: `mask |= (1 << section)`
>
>`(1 << section)` shifts the `1` bit N spaces to the left.
>
>What you are effectively doing is creating an empty mask with ONLY this bit set, and then merging it with the original mask.
>
>The inclusive `OR` operator means that if a bit is set in either of the masks, it will be set in the result.
>Therefore, it will add your `1` if not present.

> To mark a section as false: `mask ^= (1 << section)`
>
>`(1 << section)` shifts the `1` bit N spaces to the left.
>
>WARNING! Unlike the previous option, this one REQUIRES that the bit was set in the mask.
>
>You are using the exclusive `XOR` operator. This means that if both masks have the `1` set, the result will be `0`. However, if the original did not have an entry here, it would be set to `1`.

To check whether a bit is set at a particular entry, you can use `(mask & (1 << section)) > 0`.

#### Manifolds
The manifold is the most complicated part of the light update packet. There is one for each light-type.
It is a list of between zero and eighteen (inclusive) byte arrays.

Each byte[] corresponds to a particular chunk section, and holds the light levels for every position in this section.

The arrays in the list correspond to the entries in the first (not-empty) mask.

*Exempli Gratia: for the mask `000000000000010010`, the manifold should contain two byte arrays, for sections two and five.*

The list does NOT contain null elements. The size of the list should be the same as the number of set entries in the mask.
The mask is then used to tell the client which sections each `byte[]` corresponds to.

The length of each `byte[]` must be exactly `2048`.

Each byte in the array corresponds to two blocks. These are in a linear order, the index of which can be found using `(x + z*16 + y*256)`.

The byte is split into two 'nibbles' - which are the lower and upper halves of the byte, each with 4 bits. These 4 bits allow for a level of 0-15 (inclusive) to be stored.

This means that we can store the light for all 4096 blocks from the section in the byte array.

##### Reading Values

>To read from the upper half of a byte, use:
`(byte & 0xF0) >> 4`

>To read from the lower half of a byte, use:
`byte & 0x0F`

##### Writing Values

>To write to the upper half of a byte, use:
>
>`byte &= 0x0F` (Wipes the upper half.)
>
>`byte |= ((level << 4) & 0xF0)` (Merges in our number to the final four bits.)

>To write to the lower half of a byte, use:
>
>`byte &= 0xF0` (Wipes the lower half.)
>
>`byte |= (level & 0x0F)` (Merges in our number to the first four bits.)

