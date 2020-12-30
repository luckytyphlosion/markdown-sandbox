A note on diff syntax:
```diff
 this is some code
-delete red - lines
+add green + lines
```

Say you want to insert in the palette for graphics/object_events/pics/people/lass.png.

1. Go to src/data/object_events/object_event_graphics.h
2. Add the following line to the end of the file:
    ```c
    const u16 <variableName>[] = INCBIN_U16("graphics/object_events/pics/people/<spriteFile>.gbapal");
    ```
    Where \<spriteFile> is the name of the sprite png **without the .png extension** (in this case, \<spriteFile> is lass), and  \<variableName> is a name we give to the included palette to reference it elsewhere.
    For example, for lass.png, we could give this variable the name `gObjectEventPalette_Lass`. Then the line would be
    ```c
    const u16 gObjectEventPalette_Lass[] = INCBIN_U16("graphics/object_events/pics/people/lass.gbapal");
    ```
    Ensure that the variable name is unique (as in, so that two variables don't have the same name), otherwise you'll get an error.

3. Open src/event_object_movement.c.
Find the end of the list where the `OBJ_EVENT_PAL_TAG`s are defined. You can do this by Ctrl+F'ing for the phrase `#define OBJ_EVENT_PAL_TAG_NONE`. The list should look something like (but not exactly):
    ```c
    #define OBJ_EVENT_PAL_TAG_33 0x1122
    #define OBJ_EVENT_PAL_TAG_34 0x1123
    #define OBJ_EVENT_PAL_TAG_35 0x1124
    #define OBJ_EVENT_PAL_TAG_36 0x1125
    #define OBJ_EVENT_PAL_TAG_NONE 0x11FF
    ```
    Now, add an `OBJ_EVENT_PAL_TAG` define **above** the `#define OBJ_EVENT_PAL_TAG_NONE 0x11FF` line. The general structure of this define is as follows
    ```c
    #define OBJ_EVENT_PAL_TAG_<num> (0x1101 + <num>)
    ```
    where \<num> is the number of the previous `OBJ_EVENT_PAL_TAG` number plus one. So for example, here the last `OBJ_EVENT_PAL_TAG` number is 36, thus our number is 37, so we would do:
    ```c
    #define OBJ_EVENT_PAL_TAG_37 (0x1101 + 37)
    ```
    The way we define the `OBJ_EVENT_PAL_TAG` value is different from the other examples `(0x1101 + 37)` vs. `0x1126`, but this isn't functionally different.
    Overall, for our example, the result should look like:
    ```diff
    #define OBJ_EVENT_PAL_TAG_33 0x1122
    #define OBJ_EVENT_PAL_TAG_34 0x1123
    #define OBJ_EVENT_PAL_TAG_35 0x1124
    #define OBJ_EVENT_PAL_TAG_36 0x1125
    +#define OBJ_EVENT_PAL_TAG_37 (0x1101 + 37)
    #define OBJ_EVENT_PAL_TAG_NONE 0x11FF
    ```

4. While still in src/event_object_movement.c, look for the end sprite palette list, by searching for the following phrase: `#include "data/object_events/berry_tree_graphics_tables.h"`. The end of the list should look something like:
    ```c
        {gObjectEventPalette33, OBJ_EVENT_PAL_TAG_33},
        {gObjectEventPalette34, OBJ_EVENT_PAL_TAG_34},
        {gObjectEventPaletteCutTree, OBJ_EVENT_PAL_TAG_35},
        {gObjectEventPaletteWally, OBJ_EVENT_PAL_TAG_36},
        {NULL,                  0x0000},
    };

    #include "data/object_events/berry_tree_graphics_tables.h"
    ```

    (The #include isn't actually part of the list, but it's a convenient way to get there).

    Now, insert an entry for the new palette **above** the `{NULL, 0x0000}` line. The general structure of this entry is as follows:
    ```c
        {<variableName>, OBJ_EVENT_PAL_TAG_<num>},
    ```
    where \<variableName> was the name we gave to the included palette in Step 2, and \<num> was the number we came up with in Step 3. For our example, the line would look like:
    ```c
        {gObjectEventPalette_Lass, OBJ_EVENT_PAL_TAG_37},
    ```
    Overall, this will look like:
    ```diff
        {gObjectEventPalette33, OBJ_EVENT_PAL_TAG_33},
        {gObjectEventPalette34, OBJ_EVENT_PAL_TAG_34},
        {gObjectEventPaletteCutTree, OBJ_EVENT_PAL_TAG_35},
        {gObjectEventPaletteWally, OBJ_EVENT_PAL_TAG_36},
    +   {gObjectEventPalette_Lass, OBJ_EVENT_PAL_TAG_37},
        {NULL,                  0x0000},
    };
    ```

5. Open src/data/object_events/object_event_graphics_info.h.
Find the graphics info structure of the overworld sprite you're replacing. In our example, we're replacing the **lass** sprite, so we would Ctrl+F for "**lass**". This gets us to this part of the table:
    ```c
    const struct ObjectEventGraphicsInfo gObjectEventGraphicsInfo_Scientist1 = {0xFFFF, OBJ_EVENT_PAL_TAG_2, OBJ_EVENT_PAL_TAG_NONE,
    256, 16, 32, 4, SHADOW_SIZE_M, FALSE, FALSE, TRACKS_FOOT, &gObjectEventBaseOam_16x32, gObjectEventSpriteOamTables_16x32,
    gObjectEventImageAnimTable_Standard, gObjectEventPicTable_Scientist1, gDummySpriteAffineAnimTable};
    
    <<<
    const struct ObjectEventGraphicsInfo gObjectEventGraphicsInfo_Lass = {0xFFFF, OBJ_EVENT_PAL_TAG_3, OBJ_EVENT_PAL_TAG_NONE,
    256, 16, 32, 5, SHADOW_SIZE_M, FALSE, FALSE, TRACKS_FOOT, &gObjectEventBaseOam_16x32, gObjectEventSpriteOamTables_16x32,
    gObjectEventImageAnimTable_Standard, gObjectEventPicTable_Lass, gDummySpriteAffineAnimTable};
    >>>
    
    const struct ObjectEventGraphicsInfo gObjectEventGraphicsInfo_Gentleman = {0xFFFF, OBJ_EVENT_PAL_TAG_2, OBJ_EVENT_PAL_TAG_NONE,
    256, 16, 32, 4, SHADOW_SIZE_M, FALSE, FALSE, TRACKS_FOOT, &gObjectEventBaseOam_16x32, gObjectEventSpriteOamTables_16x32,
    gObjectEventImageAnimTable_Standard, gObjectEventPicTable_Gentleman, gDummySpriteAffineAnimTable};
    ```
    (\<<< and \>>> are just highlighters and are not in the actual file. Do not add \<<< and \>>> to the files).

    We can identify that `gObjectEventGraphicsInfo_Lass` is the variable name of the graphics info variable. Now, on the same line as the variable name, look for `OBJ_EVENT_PAL_TAG_<anyNum>`, where \<anyNum> is just any number. Replace \<anyNum> with the \<num> you came up in step 3, so in our case, we would get `OBJ_EVENT_PAL_TAG_37`

    Overall, this would look like:
    ```diff
    -const struct ObjectEventGraphicsInfo gObjectEventGraphicsInfo_Lass = {0xFFFF, OBJ_EVENT_PAL_TAG_3, OBJ_EVENT_PAL_TAG_NONE,
    -256, 16, 32, 5, SHADOW_SIZE_M, FALSE, FALSE, TRACKS_FOOT, &gObjectEventBaseOam_16x32, gObjectEventSpriteOamTables_16x32,
    -gObjectEventImageAnimTable_Standard, gObjectEventPicTable_Lass, gDummySpriteAffineAnimTable};
    +const struct ObjectEventGraphicsInfo gObjectEventGraphicsInfo_Lass = {0xFFFF, OBJ_EVENT_PAL_TAG_37, OBJ_EVENT_PAL_TAG_NONE,
    +256, 16, 32, 5, SHADOW_SIZE_M, FALSE, FALSE, TRACKS_FOOT, &gObjectEventBaseOam_16x32, gObjectEventSpriteOamTables_16x32,
    +gObjectEventImageAnimTable_Standard, gObjectEventPicTable_Lass, gDummySpriteAffineAnimTable};
    ```

Once step 5 is done, the palette should be replaced. Run `make` to verify your changes.
