# Clique Enhanced for The Burning Crusade

This is an enhanced, bugfixed version of **Clique** for The Burning Crusade! (TBC 2.4.3)

It is a major rewrite, based on the final Wrath of the Lich King version of Clique (r143), backported for The Burning Crusade and then improved with tons of fixes and enhancements.

## Project Statistics

* `1523` lines of coded added, `924` lines of code deleted. Not counting whitespace! The whole Clique Enhanced project is around 3400 lines of code, which means that *almost half of all Clique code* has been rewritten.

* Almost `40` bugfixes. Many very serious bugs and reliability issues have been fixed.

* Over `30` enhancements, with several new features and tons of "quality of life" and ease-of-use improvements.

## Download

**Download: [CliqueEnhancedTBC-master.zip](https://github.com/VideoPlayerCode/CliqueEnhancedTBC/archive/master.zip)** (Put the inner "Clique" folder into your WoW's "Interface/AddOns" folder.)

## Full List of Changes in Clique Enhanced

```
commit a9754023a0f202024f04466750080a704b8f963c
Author: VideoPlayerCode
Date:   Wed May 15 12:58:01 2019 +0200

    Add author credits to all source code files

    Clique:
    Original code by Cladhaire.

    Clique Enhanced:
    Enhancements and bugfixes by VideoPlayerCode.

commit b0e142c44793ac578bcc3507da84bf9524e6d45e
Author: VideoPlayerCode
Date:   Fri May 10 19:33:59 2019 +0200

    Enhance: Added sounds when opening/closing Clique

    The Clique window was always opening/closing in complete silence,
    which provided absolutely zero audio-feedback to the player.

    To give the window some life, we now play the soft "opening/closing
    the friends panel" Blizzard UI sounds when you toggle Clique!

commit 079eb3a3cabbf2ec38d73a7d5a4a53baae5811aa
Author: VideoPlayerCode
Date:   Fri May 10 19:16:34 2019 +0200

    Bugfix: Refuse to toggle Clique in combat

    There was already an "OnShow: if in combat, immediately hide again"
    handler, and a "when entering combat, hide Clique" event handler.

    But this commit adds some code which gets deeper into the root of the
    problem, by simply making "Clique:Toggle()" refuse to do anything
    if the player is currently in combat!

    All of these "hide in combat" tricks are necessary because Clique
    isn't allowed to modify game bindings while the player is in combat,
    so it makes no sense to even show the Clique frame at all.

    This commit also cleans up some if/elseif/else nesting a bit...

commit 1cbfdb3f62633745047fec08919c500a14da482e
Author: VideoPlayerCode
Date:   Fri May 10 19:09:21 2019 +0200

    Bugfix: Use IsShown when toggling visibility

    The old code used "IsVisible", which is useful for some things, but
    demands that the entire parent frame hierarchy is visible too.

    Basically: IsVisible = Currently visible on screen. IsShown = Has
    been marked as visible, regardless of whether it's being shown or
    is invisible (due to its parents being hidden).

    When toggling, we want to use "IsShown" to ensure that we properly
    toggle frame states even when their parents are temporarily hidden.

commit 976000ff9e2a466aa10aef1c913c846e39996019
Author: VideoPlayerCode
Date:   Fri May 10 17:15:11 2019 +0200

    Enhance: Improved validation of spellbook clicks

    Plenty of prior commits have already dealt with improving the important
    "Clique:SpellBookButtonPressed(frame, button)" function, which is the
    core function responsible for validating and adding bindings to Clique.

    However, it wasn't validating whether the spell-ID was within bounds
    of the maximum limits of the current spell-tab. Therefore, extra
    validation has now been added to ensure perfect reliability.

    * Aborting if ID is > MAX_SPELLS (1024), which is Blizzard's max valid
      ID for API calls; anything higher would cause errors to be thrown.

    * Aborting if ID is higher than the max ID on the currently active
      spell tab/school, to ensure that we can never get spells from
      other tabs. This fix wasn't as important since Clique in WotLK
      (which this backport / rewrite is based on) ensured that the binding
      function is never called on buttons that lack spells. But it's
      still a good improvement for enhanced function reliability.

    * Aborting if we can't retrieve the spell's texture, which indicates
      that the spell ID is invalid. This is just an extra check in addition
      to the "abort if spell name couldn't be retrieved" check which we
      already added in a prior commit. They both complement each other.

    * The code which transforms the clicked button (such as "LeftButton")
      into the button's number (such as 1) was doing some incorrect
      validation of "invalid button" return values. It didn't check for
      an empty string until AFTER we had already added "harmbutton"
      or "helpbutton" to the final string, which meant that the validation
      never saw empty strings (undetectable buttons) if the user was
      adding bindings on the harmful or helpful bindings pages. Now
      we instead check the "GetButtonNumber" return value immediately,
      and abort if it's an empty string (meaning the button was invalid).

    * We now cache the "SpellBookFrame.bookType" variable in a local var
      instead of constantly fetching it globally. A minor improvement.

commit e0ad5c80f6563ef0d6eeb9aa48a2af41a4fe395f
Author: VideoPlayerCode
Date:   Fri May 10 16:59:57 2019 +0200

    Bugfix: Use frame reference from function call

    The important "Clique:SpellBookButtonPressed(frame, button)" function
    had been improperly updated by the original addon authors. They had
    added the "frame" argument to the function to make it more reliable,
    but were still using the ancient "this" variable in the code instead;
    in fact the "frame" reference wasn't used anywhere at all...

    That has now been fixed to use the frame reference provided by caller.

commit 3dca573874402ef29994eeebe5080745bf7685a6
Author: VideoPlayerCode
Date:   Mon Apr 29 01:12:36 2019 +0200

    Bugfix: Auto-close the Custom Action Icon selector

    When you closed the Custom Action editor (the "Custom" / "Edit" window),
    it didn't properly close its Icon Selector frame, which meant that the
    next time you opened the Custom editor, it still had a completely
    unwanted and unrelated "Icon Selector" floating on top of it, if you
    hadn't manually closed the selector BEFORE closing the Custom window.

    We now AUTOMATICALLY close any lingering Icon Selector window when the
    user presses either "Cancel" or "Save" on the Custom Action editor.

    Note that we do NOT auto-close it "OnHide", since the Custom editor
    can be hidden by all kinds of things that don't mean that the user
    wanted to end their editing. For example, closing and re-opening
    the spellbook would temporarily hide and then re-appear Clique!

commit 3cadf5634f92e6e918fba8d5137a796b2b983a6f
Author: VideoPlayerCode
Date:   Sun Apr 28 06:37:10 2019 +0200

    Bugfix: Proper layering/ordering of Clique windows

    This is a complete rewrite of Clique's previous, poor attempt at
    layering frames properly (so that their contents, or various
    Blizzard frames, don't poke through each other's frame layers).

    The old code placed all pop-up windows ("Options", "Frames", "Edit",
    "Profiles", "Custom", etc) on the "DIALOG" frame-strata, and then
    gave them identical frame-levels of "parentlevel + 5". That solved
    the "Blizzard GUI" conflicts, but had the terrible side-effect of
    ensuring that those Clique pop-up windows couldn't layer properly
    on top of EACH OTHER, since they all had the same strata and same
    level, which meant that WoW's code had no idea how to layer Clique's
    windows. The end-result was therefore STILL a mess of windows with
    widgets that poked through each other, in a super ugly way.

    Furthermore, the old code used unreliable, independent "OnShow"
    handlers which didn't guarantee that the windows were processed
    in the correct order (since there's no guarantee of what order
    "OnShow" will fire on each window). There were even issues with
    one of those "OnShow" handlers never even firing, since the old
    code overwrote that one with a different "OnShow" handler, hehe.

    All of this has now been COMPLETELY rewritten with a new technique
    that guarantees correct processing order, applies proper frame-stratas
    and frame-levels to all windows for perfect layering/ordering, and
    efficiently ONLY executes when it's necessary, which is typically only
    once (the first time) per game session! After that, it intelligently
    detects if it ever needs to re-apply the frame-order fixes again, and
    handles everything automatically.

    This new function ONLY executes during the main CliqueFrame's "OnShow"
    handler, since there's absolutely ZERO reason to run it on every
    sub-window's "OnShow" events too. The game client REMEMBERS the strata
    and level you give to windows, and it WON'T change unless some other
    code explicitly sets different values. And IF that happens, we treat
    that as a low-priority issue which gets fixed on our next main-"OnShow".

commit 8868cbdbd1e14132985e1b9fdf02bf472a8e1ae3
Author: VideoPlayerCode
Date:   Sun Apr 21 19:54:08 2019 +0200

    Clean up Blizzard raid-frame creation hook

    The code was a bit disjointed; defining a local function, but only using
    it several lines further down. It also lacked comments. This has now
    been cleaned up a bit...

commit cbb8b29316af98035fe39fcf01336f25b8b474ed
Author: VideoPlayerCode
Date:   Sun Apr 21 04:20:08 2019 +0200

    Enhance: Improved debugging information

    The "/clique debug" mode has been enhanced in several ways:

    * The "OnAttributeChanged" hook is now only registered once, rather than
      every time the slash command is executed. This fix avoids getting
      multiple printed messages the more times you run the debug command!

    * The attribute printing only happens WHEN you run the slash command,
      and DOESN'T continually happen AFTER that anymore. If you want to see
      all of the frame attributes again, simply run the slash command again.

    * After running "/clique debug", a "debug" flag is now set on the main
      Clique addon object, which tells various functions to display more
      debug information:

    - When you enter or exit combat, the "UseOOCSet" and "UseCombatSet"
      functions are responsible for writing all of your out-of-combat
      or in-combat bindings to all registered frames. Those functions
      now display benchmarking information when the "debug" flag has been
      enabled! Thankfully, those benchmarks reveal that even HUGE sets
      of bindings are assigned in less than 10 milliseconds, which means
      that no optimization of the frame-binding code is necessary!

commit 14f95d6d12203af2cb2848c3e1ea0e501460e212
Author: VideoPlayerCode
Date:   Sun Apr 21 03:52:37 2019 +0200

    Bugfix: Only glow Clique spellbook tab if visible

    The "Clique" spellbook tab (the one below the normal spell school tabs)
    will now ONLY glow while Clique's window is ACTUALLY visible.

    (Before this patch, it wasn't tied to the visibility of the Clique, which
    meant that the tab could glow (as if Clique was visible) even while
    Clique's window was hidden for various reasons!)

commit 2181d9ccf414c150d2f9e938011b8731f6eb6b65
Author: VideoPlayerCode
Date:   Sun Apr 21 03:39:11 2019 +0200

    Enhance: Update unit-tooltip based on combat state

    Whenever you enter or exit combat, we now automatically update your
    currently displayed unit "binding help" tooltip (if any), so that you
    always see the ACTIVE bindings for that unit.

    For example, let's say that you're hovering over yourself ("player")
    and seeing the "friendly out of combat bindings" list. But while
    you're hovering, something attacks you and puts you in combat state.
    The tooltip will now AUTOMATICALLY update to show your "friendly
    in-combat bindings" instead!

    This enhancement applies to ALL unit tooltips. They now all update
    themselves IMMEDIATELY upon entering or exiting combat, thus
    ensuring that you ALWAYS see the currently active bindings, without
    having to manually move your cursor away from the unit and then back
    onto it again (which is what you had to do in the past). Now it's
    all done automatically for you!

    Enjoy!

commit ab60ff3fd3fdec74ef3a104b3090ebeb0ca303f4
Author: VideoPlayerCode
Date:   Sun Apr 21 03:17:13 2019 +0200

    Bugfix: Always show the CORRECT unit tooltips

    The old code was using `UnitCanAttack("player", "mouseover")` as its
    attempt to detect whether the hovered unit was hostile or friendly.

    However, that was EXTREMELY BUGGY. The game will OFTEN consider your
    "mouseover" to be something other than the unit whose name/data is being
    displayed in the game's tooltip!

    And as a result, you would very often see things like "Hostile"
    keybindings listed when hovering over your own player (FRIENDLY) frame
    or other friendly units, and vice versa...

    Therefore, the code has now been completely rewritten to the CORRECT way
    of detecting what unit tooltip you're currently looking at!

    NO MORE guessing based on "mouseover"!

    We now query the tooltip directly: `GameTooltip:GetUnit()`, which
    returns the name of its unit AND the unit identifier (such as "party1").

    We then take the latter value (the identifier), and verify that
    `UnitExists()` so that we don't proceed if that identifier is invalid.

    Next, we check `UnitCanAttack("player", unit)` to detect whether
    that identifier is hostile or friendly.

    This new solution gives us 100% accuracy, and you'll NEVER again see
    friendly tooltips for hostile units or hostile tooltips for friendly
    units! Everything now works perfectly! :-)

commit 130065522c4717ff06a5acfe9c5d8a26411ea200
Author: VideoPlayerCode
Date:   Sun Apr 21 03:08:41 2019 +0200

    Bugfix: Safer way of hooking external functions

    The old code simply overwrote external functions via "SetScript". Ew.

    Now we perform CORRECT hooking (via HookScript) instead, which even
    supports hooking secure handlers without tainting them!

commit aad126e05f6a2e62db8f970f8858ad815712f810
Author: VideoPlayerCode
Date:   Sun Apr 21 02:17:38 2019 +0200

    Enhance: Automatically assign Custom Action icons

    Whenever you bind a spell directly in Clique by clicking on a spellbook
    entry, you always get a very nice and completely accurate spell icon.

    However, adding "Custom" actions was nowhere near as polished. The icon
    was ALWAYS an ugly "?" (question mark), and it was tedious to change the
    icon. Especially if you wanted the action's icon to reflect something
    like the actual item icon for a "Use Item" action. That WASN'T EVEN
    POSSIBLE, since the manual icon selector doesn't contain item icons!

    But that's all in the past now! This patch adds AUTOMATIC icon
    assignments for ALL custom action types, to ensure that you can always
    get a quick "at a glance" view of your actions and what they do!

    We now automatically scan the action (when you save it) and all of its
    parameters to determine the most appropriate icon for action type.

    The scanning is done as follows:

    * If the user manually sets a custom icon, EXCEPT for the "?" (question
      mark) icon, then we use their manually assigned icon.

    * If the user keeps the default "?" (question mark) icon, or sets
      the icon to the question mark again, then we auto-detect the icon.

    - "Change ActionBar": An "eye" icon, meant to indicate the fact that
      you're changing your "action bar view".

    - "Action Button": A red and white "swirl" icon, since it's absolutely
      impossible to know what's going to sit on a specific action bar button
      at any given time. So this non-descript icon is intentional.

    - "Pet Action Button": The famous "sad baby seal face" pet icon.

    - "Cast Spell": A case-insensitive scan is performed in the player's
      spellbook and pet book, and if the spell is found, we use the highest
      rank's icon for that spell. If nothing is found, we keep the "?".

    - "Use Item": If the player links the item by shift-clicking on an item
      to create an item-link in the "Item Name" field, then we accurately
      extract the exact item icon. However, if the player MANUALLY TYPES the
      item name into the field, then we perform a case-insensitive scan in
      the client's item cache (all items seen in bag, bank, mail, etc) to
      attempt to find the icon. If nothing is found, we keep the "?".

    - "Run Custom Macro": A dark-blue, tranquil night sky. Just something
      to indicate the "endless possibilities of macros". Just kidding;
      actually, there just isn't any truly appropriate "macro" icon in the
      game. But the night sky is very distinct from normal spell icons, so
      your macros will stand out in the list.

    - "Stop Casting": A red "minus sign" on a red background.

    - "Target Unit": A green minotaur head with a red "crosshair" on it.

    - "Set Focus": A black silhouette against a blue background, with a
      white, glowing dot in the person's "chest".

    - "Assist Unit": The silhouette of two hands reaching up against
      a yellow, "glowing" starburst background, meant to indicate "helping"
      someone.

    - "Click Button": A tiny, blue crystal shard radiating blue pulses,
      which kind of looks like a "highlighted button".

    - "Show Unit Menu": A white, glowing orb with blue outline.

    With these new icons, you'll never be faced with a horrible list of ugly
    "?" question mark icons again.

    TIP: If you have old bindings that still use the old "?" icon, simply
    edit them and save them again to apply the auto-assigned icons!

commit b4a57ee0535394bc5866bc575d38c0cbe673775c
Author: VideoPlayerCode
Date:   Sun Apr 21 01:47:03 2019 +0200

    Enhance: Better item-link scanner in Custom editor

    This improves the item handling in "Custom Action" editor fields.

    * No more code repetition (DRY - Don't Repeat Yourself).

    * Search-pattern improved to find the earliest, shortest-possible
      match (basically, the first item in the string). The old pattern
      was greedily eating as much as possible of the input, if there
      were multiple items in the input string.

    * Now extracts the discovered item-link and runs the "GetItemInfo"
      API on it to retrieve the most accurate, cleaned-up item name.

commit d71c4817c9a60720f54e93cade162d73aa06a764
Author: VideoPlayerCode
Date:   Sun Apr 21 01:35:28 2019 +0200

    Bugfix: Fix "Cast Spell" and "Use Item" validation

    * The code that was SUPPOSED to ensure that the "Cast Spell" action
      had a spell name provided in arg1, actually wrongly did "if arg1
      OR some other argument is provided", despite the fact that arg1
      (the spell name to cast) must ALWAYS be provided for this action.

      This meant that if you provided secondary parameters such as "spell
      target" but not the spell name itself, the custom action still
      saved itself as if nothing was wrong.

      Therefore, the entire validation was rewritten to just demand
      that "arg1" (spell name) is provided. That's the most important
      argument! Every other argument is completely optional!

    * Also reject any "Use Item" actions where the user has provided the
      bag slot and item slot to use AND an item name to use. That's not
      valid. They can use EITHER the bag/slot system OR the item name
      system, but not both at the same time!

    * Improved grammar/clarity of some of the validation error messages.

    * Cleaned up some references to "entry.arg1" (and arg2) that look
      cleaner and more unified as just "arg1".

commit e995398d1cd0277f3329b65b69fdfca532149c19
Author: VideoPlayerCode
Date:   Sun Apr 21 01:07:45 2019 +0200

    Enhance: Rename "Show Menu" to "Show Unit Menu"

    The old action description wasn't helpful enough. The player would
    definitely wonder "what menu?".

    It actually refers to the player's context menu (the one that
    contains options such as "Whisper" and "Invite"). Therefore, the
    action has been renamed to "Show Unit Menu" instead.

commit 557599dc647cd43dcade466c07c668788664b0c0
Author: VideoPlayerCode
Date:   Sun Apr 21 01:05:22 2019 +0200

    Enhance: Improve "Pet Action Button" help grammar

    The old help message was a mess... ;-)

commit ed119304c0f3ecd8180ca12e56da3689d2123ba7
Author: VideoPlayerCode
Date:   Sun Apr 21 01:01:52 2019 +0200

    Bugfix: Render "click" action in bind-listing

    * The list of bindings couldn't properly display "click" bindings.
      Support has now been added for that action type.

    * Re-arranged the display-code in the same order as the list of action
      types in the "Custom Action" window, with comments for each type. And
      added a "MISSING xx FORMAT!" error message to avoid any future risk of
      problems such as the lack of a "click" renderer.

commit 556dcd2b20cbab46528f84f0dfa4cccf4f16aa67
Author: VideoPlayerCode
Date:   Sun Apr 21 00:52:34 2019 +0200

    Bugfix: Require button name for "click" action

    The "Custom Action" input validation wasn't verifying that a button
    named had been specified for the "Click" action type. This has now
    been fixed.

commit b5ea489de476e5975630577edcaed2a21c053b78
Author: VideoPlayerCode
Date:   Sat Apr 20 23:54:17 2019 +0200

    Bugfix: Refuse to bind passive spells in Custom

    When the user creates a Custom Action, they were previously able to type
    names such as "Frost Resistance" (which is a Racial Passive), and the
    action would happily add itself to the list.

    That's bad, since passive spells CANNOT BE CAST, so it makes zero sense
    to bind them. That's why the main binding method (clicking spells in the
    spellbook) refuses to add passives!

    This situation has now been fixed. The Custom Action window will now
    refuse to save any "Cast Spell" binding that refers to a Passive or
    Racial Passive spell.

commit 1e916166ca2611cc93b1c2028ae9d01ccb5ba485
Author: VideoPlayerCode
Date:   Sat Apr 20 23:50:42 2019 +0200

    Bugfix: Properly display invalid custom actions

    When the user types unexpected data into the Custom Action window, such
    as typing a word inside of a "Bag Number" field, it would break all of
    Clique permanently! The whole list display of all bound actions would
    actually become completely corrupt and crash the addon, so that the user
    couldn't even edit their custom action to fix it!

    That has now been solved, by removing type-restrictions on "argument
    formatting", so that we don't DEMAND that certain arguments are numbers,
    etc. Instead, everything is rendered/formatted as strings, which means
    that the "corrupted binding list" situation can't happen anymore.

commit e4dcb633e7119a2f7636fe5794acf3698c5c0e2f
Author: VideoPlayerCode
Date:   Sat Apr 20 22:49:15 2019 +0200

    Bugfix: Remove all old, unused StaticPopupDialogs

    * Removed "CLIQUE_DELETE_PROFILE", which seems to have been the old
      "profile deletion" system in Clique before the Profiles-list window
      with its easy "Delete" button was added.

    * Removed "CLIQUE_COMBAT_LOCKDOWN", which was inconsistently only called
      from a single code location: When trying to Delete an action while in
      combat. And furthermore, the fact is that Clique ALWAYS HIDES ITSELF
      in combat, so there's no way a user could even click the Delete button!

    * Added a bunch of "if InCombatLockdown() then return end" as safeguards
      to a bunch of risky button actions (such as Delete, Edit, Save (Custom),
      etc). Yet again, this code should never run since the GUI always hides
      itself in combat. But it's a nice, short safeguard against any danger.

commit 1ae0beb87489715cdc69f192727a644bee818662
Author: VideoPlayerCode
Date:   Sat Apr 20 22:27:01 2019 +0200

    Enhance: New, robust clickset data storage system

    * This patch rewrites the entire clickset data storage system, into a
      robust, clean and modern locale-specific system instead.

      The old system stored clicksets as "profile.clicksets[translated
      table key] = data", such as "profile.clicksets['Out of Combat']",
      which was EXTREMELY FRAGILE for multiple reasons. If you EVER changed
      the translation (even changing the CASE of a SINGLE letter), the old
      clicksets would be "lost forever" since the data keys would no longer
      match. And furthermore, if you had identical key-translations in
      multiple game client locales, then their data tables would mix and
      overwrite each other, despite the fact that all bindings MUST be kept
      separated between locales (since all spells/items are localized in
      each game language and therefore the bindings are not portable between
      different game locales).

      To solve all of this, the new system now stores all data robustly and
      independently as "profile.clicksets[GAME_LANGUAGE][NEUTRAL SET KEY]",
      such as "profile.clicksets.enUS.HARMFUL".

      This ensures that there's ZERO risk of clashes between the clicksets
      of different locales, and also ensures that we are now free to edit
      the visual GUI translations without ever losing our bindings, since
      they are now keyed in a neutral, completely locale-independent way.

    * Data stored in the old system is automatically upgraded to the new
      storage system. Which means that the upgrade is seamless for people
      coming from older Clique versions.

    * The English text labels for the various set names have been improved,
      so that their capitalization is now in uniform title-case. Instead of
      "Harmful actions" we now have "Harmful Actions", etc. This is possible
      thanks to the new storage system decoupling the data from translation.

    * This patch also improves the efficiency of the clickset dropdown menu.
      It previously re-built a table of translated clickset names on every
      "OnShow". We now only build it ONCE, during initial GUI creation.

commit 10e5781b947fd6227fcf1a3bdbef9e8504b3ae9d
Author: VideoPlayerCode
Date:   Wed Apr 17 19:58:06 2019 +0200

    Bugfix: Select dropdown "Default" on profile swap

    The dropdown menu was hardcoded to link to the table of the currently
    loaded set. So if you swapped to a different profile (which of course
    has different set tables), the menu still stayed on its old selection,
    rather than correctly showing the fact that the set-swapping moved the
    visual list view to the "Default" set instead.

    For example:

    - Load Profile A.
    - Select clickset "Out of Combat".
    - Load Profile B.
    - The visual list of items would show items from the "Default" set,
      but the dropdown menu would still (incorrectly) say "Out of Combat".

    That has now been fixed by decoupling the dropdown menu from the
    clickset tables (no more hardcoded, direct table links), and by
    always refreshing the dropdown selection whenever you change
    to a different profile.

commit ba6a2c18bc896c1d1e9c9de6753d535c2de8199d
Author: VideoPlayerCode
Date:   Tue Apr 16 21:09:23 2019 +0200

    Enhance: Made "unit tooltips" option per-character

    This option was previously independently saved within every profile
    themselves, which meant that when you switched between profiles,
    your tooltip visibility choice would constantly change.

    That was very annoying...

    It is now a per-character setting instead, just like everything else
    within the Clique "Options" window.

    Also did some refactoring (of how we link profile data to the Clique
    object whenever you load/change profile), and added code to ensure
    that the old "tooltip" setting is auto-migrated so that you get a
    seamless transition if you upgrade from older versions of Clique.

commit a4fa8fea809eb958fc5e3cbe583e8f9d2586ff44
Author: VideoPlayerCode
Date:   Tue Apr 16 18:35:21 2019 +0200

    Bugfix: Fixed the fragile Passive spell detection

    * The old code was trying to detect Passive/Racial Passive spells
      by literally looking for those translated strings in all game
      languages. Not even all languages had such translations.

      That was VERY fragile! All "Passive"-translations have now been deleted,
      and the code has been rewritten to use the reliable "IsPassiveSpell()"
      game API for passive spell detection instead, hehe.

    * There was also some refactoring of the "SpellBookButtonPressed"
      function to make it more readable, safer, and slightly faster.
      Basically by exiting if certain vital data is missing, avoiding
      some redundant/pointless variable creations, and by not creating
      the "entry"-table until ALL validations have succeeded (to avoid
      leaving behind table memory garbage if the binding fails).

commit 1fa61361ac2b04dd63d846a1c1bc4532a53a45ef
Author: VideoPlayerCode
Date:   Tue Apr 16 17:57:10 2019 +0200

    Bugfix: Correctly toggle Delete/Edit/Max buttons

    The Delete/Edit/Max bottom-bar buttons are supposed to light up
    appropriately based on your binding-list selection. However,
    due to a silly bug in the order of the code, it was applying
    the button toggling BEFORE re-sorting the list elements.

    This meant that if you added or deleted a binding, and that caused
    your "selection" to move to another list entry, then the bottom-bar
    buttons did NOT properly update to reflect the NEW selection.

    So, for example, if you select a rankless/max-rank spell, which means
    that the "Max" button is disabled, and then you ADD/DELETE some
    binding which causes your SELECTION to land on a binding WITH a
    rank, then the "Max" button did NOT get properly enabled, and vice
    versa...

    The fix was simple: FIRST re-sort the list's latest contents, THEN
    toggle the bottom-bar buttons based on the RESULTING list selection.

    (The old code was doing that completely backwards. Mind-boggling.)

commit 9df7ba6b2b088b20a344d1a5a5490584e950c150
Author: VideoPlayerCode
Date:   Tue Apr 16 08:09:42 2019 +0200

    Rename SpellBookButtonPressed entry table

    The "t" table name wasn't consistent with the fact that these kinds of
    binding data tables are named "entry" everywhere else in the code.

    That has now been fixed, to improve code clarity.

commit 57c2ec7d4c9afa53ed0a80f8efaf84b7e12b1020
Author: VideoPlayerCode
Date:   Tue Apr 16 07:45:36 2019 +0200

    Enhance: Added "auto-bind max spell rank" feature

    Adds an intelligent new option named "Bind spells as rankless when
    clicking highest rank". This is enabled by default for all players.

    Whenever you click on a spell to bind it, we will automatically
    analyze it to detect whether you've clicked on the highest-rank
    version of your spell. If that's the case, we bind it as the
    max-rank version. (If NOT, we keep your clicked lower-rank choice.)

    For example, "Freezing Trap" instead of "Freezing Trap(Rank 3)".

    Any time such a rankless/max-rank binding triggers, the game will
    cast the player's highest available rank of that spell, rather
    than a specific/lower rank.

    This feature will be particularly useful for people who are playing
    on lower-level characters who don't have the endgame ranks of spells
    yet. With this new system, your bindings will automatically become
    "max-rank" which means that they'll automatically cast your highest
    available rank even as you continue to level up and earn better
    ranks of your spells. (If you instead bind specific, lower-level
    ranks, as Clique would do without this new feature, then it would
    have continued casting low-level versions of your spells.)

    NOTE: If you don't like this automatic max-rank feature, you can
    manually click on the newly added "Max" button in the Clique GUI
    instead, to turn individual spells into rankless versions on-demand.

commit 3236850b95e2fef05fb95c7f8e7d67b311b05e98
Author: VideoPlayerCode
Date:   Tue Apr 16 07:07:02 2019 +0200

    Bugfix: Remove redundant "ListScrollUpdate" call

    The "ListScrollUpdate" is ALWAYS called at the end of "ButtonOnClick",
    so there was absolutely no reason to call it inside of the delete
    handler.

    This commit also cleans up the delete handler's code a bit, for
    readability.

commit 64fc19c865f226c53bd8b888b3223bab92e0384e
Author: VideoPlayerCode
Date:   Tue Apr 16 06:52:27 2019 +0200

    Enhance: Added Max-button to quickly set max rank

    * This commit re-introduces a feature which existed in old versions of
      Clique. It's a "Max" button, which lights up (clickable) every time
      you select a Clique spell binding which isn't the maximum possible rank
      of the spell. If you click on the "Max" button, the spell binding's
      rank information is erased and turned into a "cast the maximum rank
      of the spell" action instead. It's just a much faster and more
      convenient way than going into the "Edit" view and clearing the rank
      field manually.

    * To fit this new button, the "Profiles" button was moved to the top
      left of the window, next to the "Preview" feature's button, which
      is actually a very clean and logical location. The "Max" button was
      then put on the bottom bar, grouped as "Delete/Edit/Max", just like
      in older versions of Clique!

commit c8cbcda60eab5eb1e88a869da2f6d34cee8b8e76
Author: VideoPlayerCode
Date:   Tue Apr 16 06:06:55 2019 +0200

    Rename internal Set/DeleteAction to clearer names

    These functions had absolutely horrible names. "SetAction"?
    "DeleteAction"? Anyone reading the code and seeing the calls
    to them would be confused about what their purpose is...

    What they actually did was simply loop over all registered frames
    and run "SetAttribute" / "DeleteAttribute" on them.

    Therefore, the functions have been renamed to "SetAttributeAllFrames"
    and "DeleteAttributeAllFrames" for code readability purposes.

commit 55038be7fff83c777ebea4326257aa21dbf11236
Author: VideoPlayerCode
Date:   Tue Apr 16 05:53:21 2019 +0200

    Enhance: Extend custom macros to 1024 characters

    * Custom macros can now be up to 1024 characters long (previously
      their limit was set at 255, same as a standard WoW macro). This
      massive upgrade is possible thanks to the fact that Blizzard
      actually allows up to 1024 characters in custom "macrotext" frame
      attributes. This should help a lot of advanced users! Enjoy!

    * Added some validation code to ensure that macros are never longer
      than 1024, since Blizzard truncates longer macros.

commit b7a58af1592bcc681de4956f61b01b29b965bbb6
Author: VideoPlayerCode
Date:   Tue Apr 16 05:24:23 2019 +0200

    Replace ancient table.getn calls with new operator

    Lua 5.0 and older used "table.getn(table)". Lua 5.1 (used by TBC) and
    newer allow the much shorter "#table" notation instead.

commit 26ad5542a9433d84900a28b3e237d75190636f7a
Author: VideoPlayerCode
Date:   Mon Apr 15 20:11:36 2019 +0200

    Enhance: Document [target=clique] macro feature

    The macro-text field actually takes a special "/cast [target=clique]
    Some Spell" unit specifier, which gets translated into whatever unit
    is bound to the clicked frame, such as "/cast [target=player] Some
    Spell" if you click the player frame, or "/cast [target=party1] Some
    Spell" if you click the first party frame, etc.

    (This translation happens at bind-time, inside the "macro" parsing
    section of "Clique:SetAttribute()".)

    People usually achieved this via "[target=mouseover]", which works too,
    and is probably preferable since it's more portable (can be copy-pasted
    into a normal non-Clique macro too), but anyway, now the secret
    "[target=clique]" feature is documented at last!

    Unfortunately the rephrasing of the macro help text means that all
    other translations will need updates, which will most likely never
    happen, but so what... it's one tiny help-field in a very specific
    section of the GUI, and most people understand at least *some* English
    and can read the text fine even if their game is in another language,
    and Clique itself was never a well-translated project, with tons of
    missing translations everywhere anyway...

    Lastly, this commit cleans up the other English help-texts of
    the custom editor, since many of them were quite ugly (with some
    ending in punctuation while others had no punctuation, etc).

commit 70bedc512e77f382847875ee91eb13c5d6f962f5
Author: VideoPlayerCode
Date:   Mon Apr 15 19:36:18 2019 +0200

    Bugfix: Hide custom macro-text & refactor loading

    * The "Custom Action Editor" had a bug, in that it never hid the huge
      "custom macro-text" field when you closed the custom editor
      (regardless of whether you used its "Cancel" or "Save" buttons),
      which meant that if you opened the Custom editor, browsed to Macro,
      then closed the editor, and then opened another Custom editor, it
      would still show the "macro-text" field in the middle of the window,
      which of course wasn't intended.

      The Custom Editor now always hides the macro-text field when it's
      closed, and only shows it again when necessary.

    * Refactored the code a bit, as well, for clarity and to remove
      redundant statements...

      The worst offender was a variable named "entry", which was very
      confusing to anyone reading the code, since that's the name used for
      all variables that contain a Clique binding entry. But this one only
      contained data about how to display the current "Custom Editor"
      window section and had NOTHING to do with actual bindings. So
      it was renamed to "customSection" for clarity.

      Furthermore, the "Custom Editor" loading code was refactored a bit
      to ensure that all "custom radiobutton state" handling was in the
      "CustomRadio()" function, rather than the huge mess of having some of
      the code inside the function and some of it handled outside/immediately
      afterwards by the caller (haha).

commit 6312d372d3c40fb16aee5c2b62286318a5459b07
Author: VideoPlayerCode
Date:   Mon Apr 15 19:03:48 2019 +0200

    Bugfix: Fixed all bugs in custom action saving

    * If you created a "Run Custom Macro" binding with some macro-text,
      and then decided to EDIT that binding again and deleted the macro-text
      and set a value in the macro-index field instead, then you ALWAYS
      received an error message saying "You must specify EITHER a macro index,
      or macro text, not both".

      This bug happened because custom macro-texts were saved in the hidden
      "CliqueCustomArg2" text field, and then retrieved again while saving
      the macro, even if the ACTUAL, big macro-editing field was emptied by
      the user! Which meant that it was IMPOSSIBLE to ever get Clique to
      "forget" about old macro-text. You had to DELETE your custom action
      and create a brand new one, if you wanted to clear the macro-text...

      It has now been fixed, by automatically blanking out "arg2" if no
      macro-text has been specified in the big macro editing field.

    * Furthermore, ALL text-fields are now automatically trimmed so that
      any leading or trailing whitespace is removed. This ensures that we
      save nice and clean values that Blizzard's UI expects, such as
      target="player" instead of target="player  ". It also helps us better
      detect empty text fields, such as "  " which would have been treated
      as non-empty if we didn't trim it.

commit 151a2daf9e9cc5ef7bd8fdbef052d947213557b8
Author: VideoPlayerCode
Date:   Mon Apr 15 18:18:53 2019 +0200

    Bugfix: Correctly set the target unit for actions

    The old code was completely bugged. It wasn't setting the "target unit"
    attribute of your click-actions if the action itself didn't have a target
    unit specified.

    What that meant is that if there was a lingering "target" specifier from
    an old action, then your NEW action would WRONGLY target THAT old unit.

    NOTE: A very easy way to trigger the bug would be to set up an action such
    as "Set Focus", and set its target to "player", so that clicking ANY frame
    would always target player. Then edit your action and remove the
    "player" target (make it an empty string again). Clicking on any frame
    would STILL make "player" the focus target. The correct result should be
    that the clicked frame becomes the focus, but due to the lingering
    attribute, that was completely broken...

    * We now ALWAYS set ALL attributes for a given action, to ensure that we
      overwrite any lingering, old attributes. This fixes all broken
      "targeted" actions.

    * This commit also cleans up the SetAttribute/DeleteAttribute code by
      removing unused variables and restructuring the code a bit, as well
      as commenting out a useless "assert()" statement.

commit e93105eb72d082370c61a6b76231b93e8da06b9e
Author: VideoPlayerCode
Date:   Mon Apr 15 18:06:47 2019 +0200

    Bugfix: Clean away legacy Clique "delete"-var bug

    Clique always saved a "delete = true" variable inside every entry, but
    never used that value anywhere. It makes no sense at all to tag entries
    as "deleted", since "deleting the entry from A FRAME" is a common action
    and has NOTHING to do with actually deleting the entry itself.

    We could have just removed the bugged line entirely, but instead we'll
    set the value to "nil" now to automatically clean up all player's legacy
    config databases if they come from older Clique versions.

commit f704547e1d1124b94e3cd17567d55b4c97e2266b
Author: VideoPlayerCode
Date:   Mon Apr 15 16:33:42 2019 +0200

    Enhance: Show "Clicked" as targetunit if no target

    * The target/focus/assist custom actions can omit target unit, in
      which case they automatically apply on the clicked frame instead.

      This change makes it obvious that it's applying to the clicked unit,
      by now saying "Set Focus Unit: (Clicked)", etc, rather than saying
      "Set Focus Unit: " (an empty string was used in the past).

    * The patch also fixes some statements, which were checking if "arg1"
      is non-nil, which it ALWAYS is since "arg1" was built using
      "tostring(entry.arg1)" (which EVEN TURNS nils into "nil" as a
      string). The code has now been bugfixed to check if entry.arg1
      exists before outputting "arg1", just like all the other code
      already properly does in the function.

commit acf72dedb9726bf037775990d5252ea950072fe8
Author: VideoPlayerCode
Date:   Mon Apr 15 16:19:34 2019 +0200

    Enhance: Automatically remove spell rank if empty

    Some spells, such as "Attack", have no rank information and just return
    an empty string when you call "GetSpellInfo" on them. That made Clique
    show the spell as "spell: Attack ()" or "spell: Basic Campfire ()",
    since it tried to put the (empty) rank information in the parenthesis.

    That was ugly... and has been fixed as follows:

    * Whenever the user binds a new spell, we now instead automatically
      clear its rank variable if it contains an empty string, so that we
      save the spell without rank data. This means a cleaner looking GUI
      for the player. Bindings such as "spell: Attack ()" are now properly
      rendered as "spell: Attack" instead.

    * The code which displays the list of bound spells has been changed,
      to automatically remove the rank from any legacy Clique spells (bound
      before this change took place), so that even old data is properly
      rendered without the ugly " ()" rank parenthesis.

    * That code has also been optimized in two ways: The "rank" variable
      is now marked as LOCAL (it was previously polluting the global table),
      and the rank calculation now only takes place for "spell"-type
      entries, since those are the only ones that use the rank variable.

commit a5d17446337bf98f149bba9f2fa21aa6bb28da35
Author: VideoPlayerCode
Date:   Mon Apr 15 15:54:37 2019 +0200

    Bugfix: Use localizations as button texts

    This project is still faaaaaar from perfectly localized. There's tons
    of hardcoded strings. But this commit at least applies localized strings
    to all important main buttons such as "Options", "Delete", etc.

commit 56d9a92312fb323321f181b47deb92ed7164ebd5
Author: VideoPlayerCode
Date:   Mon Apr 15 15:43:23 2019 +0200

    Enhance: Add close-button to icon select window

    Clique didn't have ANY way to "stop changing icon". So if you ever
    opened the icon select window, you were FORCED to click an icon (and
    change your binding's icon) to get rid of the window, which was
    ridiculous.

    This update adds a normal "close" button in the top-right corner, to
    let people abort the icon-changing window.

commit d56e444ba6928ec3ed28f07399562db609a9396e
Author: VideoPlayerCode
Date:   Sat Apr 13 16:57:26 2019 +0200

    Enhance: Clean up the crazy code whitespace

    This has been annoying me for so long. The entire Clique project was
    written extremely sloppily with alternating Tabs and Spaces as
    indentation EVERYWHERE, even on lines next to each other, and trailing
    whitespace everywhere.

    And while I've been patching and enhancing and fixing the code, I've
    tried to just ignore the problem and instead make my edits use the same
    indentation as their surrounding lines (to generate smaller diffs).

    But it's just a total clusterfuck. I should have cleaned up all
    whitespace earlier, to save lots of time and energy trying to match
    these insane whitespace problems...

    So here it is, at last... a commit with ONE purpose: Nuke everything,
    fix all indentation to use 4 spaces, and clean everything up!

commit 73151a2c300647f3692d6c718b17b86662c99960
Author: VideoPlayerCode
Date:   Sat Apr 13 16:16:27 2019 +0200

    Enhance: Nicer "Trigger on Down-click" label

    Apostrophes (') aren't "quote marks", tsk tsk... ;-)

commit 5c8bf04959e729d4ef23024439e916a336cdfba7
Author: VideoPlayerCode
Date:   Sat Apr 13 16:12:40 2019 +0200

    Enhance: Added "Unitframe Tooltips" to Options

    Adds a "Show your active bindings in unitframe tooltips" checkbox to the
    Clique Options window, so that the user doesn't have to manually type
    "/clique tooltip".

    This change is intended to highlight the fact that the tooltips were
    completely rewritten and redesigned from scratch, and are now extremely
    clean, useful and bug-free at last (as described in the "Major rewrite
    of binding tooltips" commit). They are now a first-class feature of
    Clique, and users are encouraged to try them out!

commit 1c705c240a127a08f749e159fb020fc558314c6c
Author: VideoPlayerCode
Date:   Sat Apr 13 15:49:42 2019 +0200

    Enhance: Added "Live Preview" button to GUI

    This adds a new "Preview" button to the GUI, which allows the user to
    quickly preview all of the final, active bindings that result from their
    various click-sets, with detailed information about the entire COMBAT
    and OUT OF COMBAT unitframe bindings.

    The "Preview" button has three functions, depending on how you click it,
    which allows very quick and easy access to all of the information that
    a player could want to know:

    * Left-Click = Show ALL bindings in a unified view containing BOTH the
      harmful unit and helpful unit actions, where it's up to the user to
      read the binding suffixes such as "LeftButton (harm)" to figure out
      which actions will run on which types of units. This view is great
      for a complete overview of "Okay, what's going on with all of my
      sets? What's the final list of all generated bindings?".

    * Shift-Left-Click = Show ONLY helpful-unit bindings, meaning things
      that will run on friendly players and NPCs.

    * Shift-Right-Click = Show ONLY harmful-unit bindings, meaning things
      that will run on attackable units (including attackable neutral units).

    * The "Clique:ShowBindings()" function has been updated to implement
      the filtering that makes all of this possible, as mentioned in the
      previous commit.

    * Window toggling was implemented, so that if you request to open the
      same view again, it actually closes the window instead. This means
      that the user no longer has to manually move the mouse cursor to the
      "X" close-button of the preview window. Instead, just click the
      "Preview"-button again, with the same modifier and mouse button,
      to close the active window. Very convenient!

    * Whenever you edit your bindings (or change the active Clique profile),
      the live preview window will *automatically* update to immediately
      reflect the new results! This aspect was actually implemented in the
      previous commit, but is mentioned here since this is a closely related
      followup expansion of the previous patch.

    * This new preview window feature uses the newly redesigned "/clique
      showbindings" window to achieve the live preview, which means that
      you can also preview your bindings via that slash command.

commit 7cf666777c53c40552a50ab419cf4c798e5559b3
Author: VideoPlayerCode
Date:   Sat Apr 13 02:39:50 2019 +0200

    Bugfix/Enhance: Major rewrite of binding tooltips

    Whew... this was a lot of work. *Everything* about the old tooltips was
    just extremely buggy and ugly garbage. Nobody would ever like to play
    with the old "tooltips" enabled. But the new system is finally clean and
    bug-free and will be a joy to use for players!

    * The first thing, which players will notice immediately, is that the
      unit-frame tooltips (enabled via "/clique tooltip") are no longer an
      absolute freaking mess. The IN-COMBAT tooltips previously displayed
      the ENTIRE "Default" click-set, AND the ENTIRE "Harmful" OR "Helpful"
      click-set (depending on whether you hovered over a friendly or hostile
      unitframe). That meant that the tooltip was extremely long and redundant,
      showing you Default bindings that didn't even exist on the frame when
      your higher-priority "harm/help" bindings were overriding them. It was
      a total mess and very hard to read. And the OUT-OF-COMBAT tooltips were
      even worse; they only displayed the "tt_ooc" list, built from the
      internal "ooc_clickset", which prior to my "Correct the OOC binding
      priorities" patch DIDN'T EVEN INCLUDE every bound key (so the DEFAULT-
      set keys were NEVER listed in the "Out of combat" bindings list). But
      that wasn't even the worst aspect; the fact is that it just blindly
      dumped the internal "ooc_clickset" data, with ZERO differentiation
      between harmful and helpful bindings. So let's say you had a "harmful
      unit" out-of-combat binding on LeftButton, and you hovered over a
      friendly unit, then you would see THE HARMFUL ACTION (even though it
      would NEVER run on your friendly unit), and the player had absolutely
      ZERO ways to know if the binding was TRULY active or not...

      All of this has been completely rewritten from scratch. We now build
      an intelligent assortment of "harmful" and "helpful" tooltip data, for
      both the OOC and IN-COMBAT states. And then we display a SINGLE,
      appropriate tooltip list showing EVERY key bound on that frame, and
      nothing else. No redundanct, huge tooltips. Just a single, clean
      section with the EXACT bindings available to use right now!

    * Secondly, the entire "/clique showbindings" summary tooltip has been
      rewritten. It was an absolute, useless, unreadable mess. It previously
      displayed FOUR separate sections labeled "Default bindings:", "Hostile
      bindings:", "Helpful bindings:" and "Out of combat bindings:". If you
      wanted to figure out if something was bound, you'd have to know that
      in-combat, the "help/harm+default" sets are active, and out of combat
      the "ooc+help/harm+default" sets are active. You'd then have to read
      up-and-down through the list to figure out WHICH bindings have higher
      precendence and which ones conflict and override each other, etc. To
      make matters even worse, the "Out of combat bindings" set just used
      the raw, internal "ooc_clickset" data, and didn't tell you which of the
      bindings were used on friendly units and which were on harmful units,
      so you'd have NO way of knowing if an out-of-combat binding was active
      on your intended target or not. And it also suffered from the bug
      mentioned above, where the OOC set didn't even include the "Default"
      bindings AT ALL... Furthermore, since the "showbindings" tooltip just
      replicated the individual binding lists that were already available in
      Clique's GUI, you didn't gain ANY useful info from it that wasn't
      already readable by viewing the different click-sets in the Clique
      editor window. And as if all of this wasn't horrible enough, the window
      even had a stupid, very thick blank area on its right side, caused by
      having a "SetPadding" command which only applied padding on the right
      side. Ugh...

      It was, in short: Completely unusable, buggy, incorrect and useless.

      Yet again, all of this has been completely rewritten from scratch!
      We now generate two intelligent, unified listings. One named "Combat
      bindings:" and another named "Out of combat bindings:". They are built
      in the exact same way that the game engine parses binding priorities,
      and they don't display ANY redundant information! You can instantly
      and effortlessly see what keys will be active when you're in combat
      or out of combat. But to help us differentiate between what actions
      (in these unified, compact lists) are used on "helpful" or "harmful"
      units, we now add suffixes to these bindings, to help the user quickly
      see what's going on. For example, "LeftButton (harm)  Attack" or
      "LeftButton (help)  Holy Light", or "(all)" in case a binding isn't
      just for a specific type of unit.

      To ensure that the unified lists remain readable, the entire lists are
      sorted by the binding (such as "Shift-LeftButton") and then by their
      unit-type ("all", "harm" or "help"). This guarantees a uniform order
      of "All > Harm > Help", for each possible key. However, an upcoming
      commit will add the ability to ONLY view the bindings that are active
      on either hostile units or helpful units, to make it even easier to
      read when you're only interested in seeing the data for a specific
      unit type. The code is already written, but not included here, to make
      this commit shorter and clearer.

      And lastly, *automatic updates* have been implemented on the window,
      so that the "showbindings" window will refresh itself *live* whenever
      you edit your bindings (or change the active Clique profile), thus
      ensuring that the listing stays fresh and that you DON'T have to
      manually run "/clique showbindings" again to check your new updates!

    * Massive amounts of poorly written, repeated code was removed during
      all of these rewrites. Yikes. Who decides to just copy code over and
      over again to change tiny aspects of an algorithm? Make a function!
      And read up a bit about DRY programming: Don't Repeat Yourself...

commit d682507017275cae00c3c85fd00ab15dc3c00fb7
Author: VideoPlayerCode
Date:   Tue Apr 9 17:15:30 2019 +0200

    Enhance: Explain binding priorities in dropdown

    The "click-set" editor's dropdown menu now has a tooltip which clearly
    explains the binding priorities when in combat or out of combat, so that
    players can better understand what the different binding types are for
    and how they interact with each other and override each other.

commit ce041e33c791d12d5d560ddaa4010b92433163cc
Author: VideoPlayerCode
Date:   Mon Apr 8 19:12:05 2019 +0200

    Bugfix: Properly show frame name in debug mode

    The "/clique debug" mode was outputting "table: 0xBLABLA" instead
    of a proper frame name in the messages. That's now been fixed.

commit 76c0a9e156ce4deca2a31fde6d5d5708d919150e
Author: VideoPlayerCode
Date:   Mon Apr 8 18:48:54 2019 +0200

    Bugfix: Safely queue in-combat registrations

    * Any frame registration changes (additions or removals) while IN-COMBAT
      are now placed in a queue and SAFELY applied AFTER combat ends instead.

      Any attempt to register a frame queues a "register" request, and any
      attempt to unregister queues an "unregister" request. And in case of
      multiple in-combat requests on the exact same frame (ie. first register,
      then unregister), only the final request is kept. That way we guarantee
      that we apply the caller's final intended state when we process the queue.

      This whole rewrite was done because Blizzard NEVER allows addons to modify
      attributes in-combat (otherwise addons would be able to automate things
      like spell-casting by making decisions and re-binding spell clicks in
      combat), so it was pointless for Clique's old code to even attempt this,
      and it just created half-applied attributes and broken frame states.

    * The "CanChangeProtectedState" modification in this patch is related to
      the same issue. That function checks whether our addon is allowed to
      modify attributes on secure/protected frames. The answer to that
      question is always YES whenever the player isn't in combat. We still
      preserve that check just to be extra safe, but we move it higher up as
      a FATAL error, since the functions now only run out-of-combat and
      should always work.

commit 3ec61b3e81aed979c79e26a55722a6a290e9d0dc
Author: VideoPlayerCode
Date:   Mon Apr 8 17:52:33 2019 +0200

    Bugfix/Enhance: Correct the OOC binding priorities

    * This is both a severe bugfix, and an enhancement (since it changes the
      binding behavior and makes it more consistent).

      The old code was generating an "OOC" (out of combat) set that contained
      the OOC bindings plus any HARM/HELP bindings that didn't conflict with
      any of the OOC bindings. But that was insufficient. It DIDN'T include
      the DEFAULT set bindings, despite them ALSO being bound while out of
      combat. As a result, the "OOC" set didn't contain all keys that were
      actually bound while out of combat. And thus any attempts to look at the
      "ooc" set to show the user a list (such as in tooltips) of OOC bindings
      didn't properly include the also-active DEFAULT bindings!

      Everything has now been rewritten with a robust algorithm which builds
      an "OOC" set that contains all OOC bindings, and then all harm/help
      bindings that DON'T use the same binds as any OOC bindings, and LASTLY
      it ALSO adds any DEFAULT (basically "fallback") bindings that aren't
      used by OOC or by BOTH HARM+HELP (because if both types of units are
      bound, then the default binding would never trigger anyway).

      As a result of this change, the generation of the OOC clickset is now
      completely consistent with the ACTUAL final "out of combat" bindings.

    * This patch also moves the initial "Clique:Enable()" call to
      "Clique:RebuildOOCSet()" much higher up, so that it actually builds the
      out-of-combat set early and applies it properly AT LOGIN, BEFORE all
      frames are registered. Otherwise the OOC clicks wouldn't be active
      until the player enters and then exits combat to re-apply the set!

    * Lastly, all repeated ApplyClickSet/RemoveClickSet calls have been
      replaced with clean, unified functions named "UseOOCSet" and
      "UseCombatSet", to reduce repetition and eliminate bugs by ensuring
      that all clicksets are applied identically to all frames.

commit 87569c96f56334296eec70e23e3b386b85b1d648
Author: VideoPlayerCode
Date:   Sun Apr 7 19:18:10 2019 +0200

    Bugfix: More efficient Clique "in use" detection

    Now stops scanning tables as soon as it finds a non-empty clickset,
    meaning that we definitely KNOW that Clique is "in active use".

    Furthermore, we set the "inuse" flag to FALSE when not in use, rather
    than "nil", to avoid triggering pointless nil-garbage collection.

commit 7dce145666e5e3d504f5a1e275f8ff46f975aab5
Author: VideoPlayerCode
Date:   Sun Apr 7 18:58:03 2019 +0200

    Bugfix: Faster "CheckBinding" and safer table keys

    * We enforce that keys in the binding-tables must always be strings.
      This should already be happening by defualt, since a string is
      concatenated with a number, but now we make it an EXPLICIT, visual
      requirement to ensure that key-equality will always be guaranteed.
      Otherwise things like key 2 ~= "2" and would fail to detect dupes.

    * The CheckBinding routine was pretty retarded. It looped through all
      keys of a table, looking for a key that matched the input, and if so
      it returned the value. Why the heck loop? Now it's been replaced with
      a single, rapid hash table lookup. ;-)

commit b7a13d34bfc73de555fd1a1abbb941b8053a8abe
Author: VideoPlayerCode
Date:   Sun Apr 7 18:46:41 2019 +0200

    Bugfix: Make SpellBookButtonPressed more reliable

    If the function is triggered in combat, or if it's unable to detect
    which mouse button was pressed, then abort the processing.

commit 9e64759ff591a5d01498cda0e99526c3109163e5
Author: VideoPlayerCode
Date:   Sun Apr 7 18:41:33 2019 +0200

    Bugfix: Only allow initialization once per session

    The "Clique:Enable()" function is now only allowed to run once, to ensure
    that we don't run initialization steps more than once per game session.

commit 70e61aa06f51862e96f2a4c36e684848fbb0013e
Author: VideoPlayerCode
Date:   Sun Mar 31 00:03:44 2019 +0100

    Bugfix: Fix the "Copy Clique Profile" command

    * "/clique profile copy <name>":
      This command was completely broken. It DIDN'T apply any of the copied
      data! It has now been fixed so that all copied bindings are applied,
      and the GUI properly updates to show the newly merged bindings.

    * "/clique profile reset":
      This command was working, but showed the wrong chat message when
      outputting the result of the command. That has now been fixed.

    All profile editing/switching handlers have also been refactored to
    avoid code repetition.

commit 0b18ca96935d5dfacbb575575ee4d3d3c8d0e7d2
Author: VideoPlayerCode
Date:   Sat Mar 30 22:59:43 2019 +0100

    Enhance: Added an easter egg to reduce my boredom

    What does it do!?

    Perhaps it enhances your luck?

    Better RNG in the game?

    Well... Just to be safe, I'll run MY game with this enabled! ;-)

commit ba1adecd77fd43113fe0bbb16e8326ba0030da68
Author: VideoPlayerCode
Date:   Sat Mar 30 21:02:26 2019 +0100

    Bugfix: Improve GetButtonNumber and GetButtonText

    Both functions were very simplistic and buggy.

    They have now been rewritten to intelligently handle any input and
    to properly validate both the input and the final return values.

    This fixes bugs all over Clique, such as this CliqueOptions line:
    "CliqueCustomButtonBinding.button = self:GetButtonNumber(entry.button)",
    which previously always became an invalid value, since GetButtonNumber
    didn't properly handle being given already-numeric input!

commit 5fcf7fb3755709de709c2ef0223d6570f3d6d3f1
Author: VideoPlayerCode
Date:   Sat Mar 30 02:03:41 2019 +0100

    Enhance: Rewrite ancient getglobal/setglobal calls

    WoW before The Burning Crusade didn't have the _G global table and
    required the use of function calls instead ("getglobal/setglobal").

    The performance of those functions is much lower than direct table
    reading. So this patch rewrites all ancient calls to direct _G table
    access instead.

commit 58c2fababeb3cf19dc5e7259635917a82d84db7b
Author: VideoPlayerCode
Date:   Sat Mar 30 01:07:17 2019 +0100

    Bugfix: Fix TONS of bugs in frame registration

    * The "_G.ClickCastFrames" global was very broken. Its purpose was to
      automatically register/unregister frames when other addons attempted
      to write to it. However, it DIDN'T properly unregister frames when
      given "falsy" values; instead, it actually REGISTERED the frame again.
      And it only actually attempted to unregister frames (stop reacting
      to clicks) if you gave it "nil", but in that case it also DELETED the
      frame from Clique's internal list. This whole "__newindex" metatable
      function has now been rewritten, cleaned up, and fixed so that it
      properly unregisters frames if given "falsy" values, and ONLY deletes
      frame information if given "nil". Furthermore, we now immediately update
      the "Clique Frame Editor" window to reflect all changes done this way.

    * The "_G.ClickCastFrames" global now has a "__index" metatable function
      to allow other addons to see (read) the true/false enabled-state of
      any frame.

    * All frame registrations are now done via individual, direct, clean
      "Clique:RegisterFrame" calls, rather than roundabout "raw-write a
      bunch of stuff from a lot of different places into Clique.ccframes,
      then loop through all those frames and register them all at once".

    * The "Clique:RegisterFrame" and "Clique:UnregisterFrame" functions were
      very sloppy and buggy. They have now been completely revamped to take
      into account every scenario, and to internally handle the updating of
      the "Clique.ccframes" table (rather than the caller needing to do that
      manually).

    * "Clique:RegisterFrame" now correctly unregisters the frame if told
      to register a blacklisted (by the user) frame.

    * "Clique:RegisterFrame" was wrongly reading "ClickCastFrames[frame]"
      to determine whether a frame wasn't already registered, and if nothing
      was found, it forcibly added the frame to "Clique.ccframes". The
      reason why this was wrong is simple: The "ClickCastFrames" table is
      ALWAYS empty and therefore always returned nil. This has been fixed
      to check the real, internal "Clique.ccframes" table instead.

    * All updates to the "registered frames" list now automatically causes
      dynamic refreshing of the "Clique Frame Editor" window, to always see
      the latest info about available frames and their registration state.

    * Fixed a serious bug, where frames refused to set/delete attributes
      if the frame was in the user's "frame blacklist". That prevented
      us from calling the functions to "clear frame data" from frames
      we unregister, for example. That's now been fixed to always allow
      editing frame attributes, to ensure "unregister" truly unregisters
      all configured click-attributes in EVERY situation!

    * The "Clique:SetClickType" function has been fixed so that it restores
      the default Blizzard "AnyUp" click-behavior when a frame is no longer
      registered (not "enabled") in Clique.

    * We now also call "Clique:SetClickType" from "Clique:UnregisterFrame"
      (and obviously from "Clique:RegisterFrame" as always), to ensure that we
      restore the default Blizzard "react to mouse-up" click behavior on any
      frames we no longer take over via Clique; otherwise the click behavior
      would be "remain incorrect" if the user enables "react to mouse-down
      click events" (which changes every registered frame's click-behavior)
      and they then disable (unregister) a specific frame. If we DON'T update
      the click type during unregistering, then that frame WOULD NOT behave
      as Blizzard intended (the "AnyUp" click style) after it's unregistered!

    * The Clique "scrolling text list" code has now been extended to allow
      the marking of "disabled" entries in the list.

    * The "Clique Frame Editor" window now says "(disabled)" after any
      frames which are NOT registered by Clique; either because of calls
      to "Clique:UnregisterFrame", or because of the user's manual disabling
      (blacklisting). Furthermore, the per-frame checkbox-buttons are now
      unchecked if the frame isn't registered, exactly the same way as
      they're unchecked when a frame is blacklisted. The purpose of this
      change is to ENSURE that if a frame isn't registered, the user will
      VISUALLY see "PlayerFrame (disabled)" with a missing checkmark, and
      can then easily click the checkmark ONCE to mark it and un-blacklist
      (even though it wasn't blacklisted) and re-register the frame for
      clicks again! It makes these actions very obvious for the user. And
      if they HAVEN'T blacklisted a frame, but it's currently disabled (hence
      showing with an unchecked box and "(disabled)", and they DO NOTHING,
      and then the frame gets re-enabled again (for some reason), then the
      list WILL immediately and automatically show that frame as "enabled"
      again. So this enhancement is truly seamless for the user!

    Fun times... The frame registration system is now a LOT more reliable!

commit a855af2399fc05da6eca72d5684f64a10f89adee
Author: VideoPlayerCode
Date:   Fri Mar 29 16:13:00 2019 +0100

    Bugfix: Restore non-empty spellbutton highlights

    When you hover over a spellbook button which doesn't contain a spell,
    the Clique overlay's "highlight" was being hidden so that it wouldn't
    show up as as highlighted button.

    But when you then switch to another page/spell-school tab, where there
    *is* a spell on that button again, it wouldn't show any highlight anymore
    since the highlight texture layer had been permanently hidden.

    This bug has now been fixed, to always restore the highlight visibility.

commit 831aa0faae098daf22a9fb97279ca8b5533b6e5b
Author: VideoPlayerCode
Date:   Fri Mar 29 03:59:31 2019 +0100

    Bugfix: Update window title on profile change

    * Refactored window title updating into a single function.

    * Now properly updates the title when changing profile (mainly done by
      opening the "Profiles" panel and clicking "Set" on another profile,
      or by clicking on "New" and creating a new profile). Before this fix,
      the old profile's title would stay permanently even when you added
      new profiles or switched to other profiles, which meant that the
      window title was always wrong. That's completely fixed now.

    * Changed window title to "Clique Enhanced", since this is a very
      different, improved version of Clique.

commit acb5cc20fa934f0bf9b6defeaf63dfcbb47eecba
Author: VideoPlayerCode
Date:   Fri Mar 29 03:42:35 2019 +0100

    Bugfix: Duplicate OnShow, improve ValidateButtons

    * The "CliqueFrame" OnShow handler was retardedly being set twice in a
      row, right next to each other... which meant that only the final
      handler was ACTUALLY being used (the 1st one was totally ignored).
      This has been fixed by merging important functions from the "lost"
      handler into the actual handler, and deleting the "lost" handler.

    * Fixed excessive calls to "ValidateButtons". It was being called twice
      every time "Clique:ButtonOnClick" ran, since that function first ran
      "ValidateButtons" and then "ListScrollUpdate" which in turn ran
      "ValidateButtons" again. This has been fixed to only run it once.

commit 0c37abec3fdee94d431fea2965fc4f488a4bcae5
Author: VideoPlayerCode
Date:   Fri Mar 29 03:32:21 2019 +0100

    Bugfix: Apply FrameLevel to CliqueOptionsFrame

    All other frames apply a FrameLevel when they're shown, to guarantee
    that they show up above other frames (reduces Z-ordering issues).

    The "CliqueOptionsFrame" lacked that code. This has now been fixed.

commit 86d87626a00d486581a98cd20bf00fdca37643a6
Author: VideoPlayerCode
Date:   Fri Mar 29 03:22:01 2019 +0100

    Enhance: Make window title fit all text

    The default width was hardcoded as "200 pixels", which made no sense,
    and didn't allow anything but the shortest character-names to fit in the
    Clique titlebar.

    This has been changed so that the title text dynamically adjusts its
    width based on the width of the parent-frame, to fit all text.

commit 902e1f0a2c5de106c4e57657558dafc296236278
Author: VideoPlayerCode
Date:   Fri Mar 29 03:08:58 2019 +0100

    Enhance: Nicer tab-tooltip and StopCasting text

    * The standard Blizzard spellbook's "Clique" tab now has a nicer
      tooltip, saying "Clique Configuration" rather than "Clique
      configuration", to better match the capitalization of all Blizzard
      spell-tab tooltips.

    * Rewrote "Cancel Pending Spell" text to "Stop Casting Current Spell",
      since the prior made no sense. The new text better matches the long
      description for the actual "Stop Casting" custom-action, as well as
      matching the terminology players are used to via "/stopcasting".

commit f41313183e81a7919493f79cb18ecb0737a1079b
Author: VideoPlayerCode
Date:   Fri Mar 29 02:58:10 2019 +0100

    Bugfix: Ensure SetProfile never gets empty input

    The "Dongle" profile database library doesn't verify that the input is
    non-empty. But rather than patch that library, we'll just ensure that
    Clique never calls it with empty input. There must ALWAYS be a profile
    name!

commit 3430c793a230cc377e9d4fcc1829a39955d93223
Author: VideoPlayerCode
Date:   Fri Mar 29 02:51:57 2019 +0100

    Enhance: Improve pairs/ipairs performance

    These functions are called a lot and should be put on the local stack to
    avoid countless global table lookups.

commit 303c940efbf2fa340a719a8f06147be586dd43e2
Author: VideoPlayerCode
Date:   Fri Mar 29 01:22:21 2019 +0100

    Enhance: Better performance of button name lookups

    The backporting is complete... it's now time to fix various bugs and
    enhance code.

    Starting with the "button 1 = LeftButton, etc..." lookup code. The
    author tried to be clever by making any number above 3 automatically
    generate a string via "Button"+number, such as "Button4". However,
    that's just pointless extra work, since Button4 and Button5 are
    officially part of the game (and anything above 5 isn't usable).

    So we'll simply hardcode the 4th and 5th button names. Anything above
    that would automatically use the auto-generation code, but that should
    never happen since WoW TBC only support 5 buttons.

commit 40eac32504f4e7c1c8da958ee254b85f6e6135b6
Author: VideoPlayerCode
Date:   Fri Mar 29 00:55:09 2019 +0100

    Make dropdown menus work in TBC again

    The TBC client provides the clicked menu element via the implicit "this".

commit d11c5a727298f97c1ca49b78e178718aa353b805
Author: VideoPlayerCode
Date:   Thu Mar 28 18:42:22 2019 +0100

    Ensure Clique's frame is parented to Spellbook

    It makes no sense to parent Clique's main window to the "pullout tab".
    Instead, we'll parent it to Blizzard's SpellBookFrame, exactly like
    older versions of Clique did.

commit 70446d450e69c2571813d70e9526e20bc6373f7b
Author: VideoPlayerCode
Date:   Thu Mar 28 18:37:53 2019 +0100

    Fix all "OnModifiedClick" hooks

    In TBC, these handlers aren't given the clicked frame as an argument;
    instead, they use "this" to retrieve the clicked frame. They do get a
    "button" argument which is just "LeftButton" or whatever mouse button
    was used, but that's useless for our purposes.

commit fe66a2a5aace8848fa491bc9f9d97dd08d4aa38d
Author: VideoPlayerCode
Date:   Thu Mar 28 18:17:41 2019 +0100

    Re-parent Clique pullout tab to SpellBookFrame

    This ensures that the Clique tab visibility depends on the Blizzard
    spellbook frame's visibility, exactly like all official Blizzard
    tabs. It made no sense to parent it to the 1st (top left) spell-
    button inside Blizzard's frame.

commit 81f9bac25046f013605e91ccb7b584e51186feec
Author: VideoPlayerCode
Date:   Thu Mar 28 17:42:37 2019 +0100

    Make "target of target" click-casting work again

    Removed all WotlK frame names, and re-added TBC's "target of target" frame.

commit eac93edc832af870bfe727921d9224bcefc7e772
Author: VideoPlayerCode
Date:   Thu Mar 28 17:32:15 2019 +0100

    Remove Blizzard_ArenaUI code

    The Blizzard_ArenaUI addon only exists in WotLK.

commit 14d00e97f08a0474d9a6bc1026d434f083f3a0bd
Author: VideoPlayerCode
Date:   Thu Mar 28 17:06:53 2019 +0100

    Make StaticPopupDialogs work again in TBC

    The TBC StaticPopupDialogs system uses "this" instead of passing a
    "self" parameter to the functions. And the dialogs are slightly
    re-arranged, so that reading the textbox is done differently.

commit 246e88b301bf675080a6a7195490a53018aebd52
Author: VideoPlayerCode
Date:   Thu Mar 28 16:19:49 2019 +0100

    Make list-scrolling work again in TBC

    The TBC client doesn't take all those extra arguments. ;-)

commit a4c519b57bd0c3c6b58b3d20bc5009dddb7e785c
Author: VideoPlayerCode
Date:   Fri Mar 29 01:52:31 2019 +0100

    Emulate "table.wipe" for TBC clients

    This function erases all table entries and then returns the table.

commit f10b2332000fc50b3f737fa6c71579078d984db5
Author: VideoPlayerCode
Date:   Thu Mar 28 16:07:46 2019 +0100

    Remove code which used max rank of WotLK spellbook

    The code was attempting to read the Wrath of the Lich King "Show all
    spell ranks" checkbox state, and if only the highest rank was being
    shown in the book, it added the spell without a rank requirement.

    This concept doesn't exist in TBC, and is therefore being removed.

commit 788afaf35569925c7bbfb160ca42f858d79cfd90
Author: VideoPlayerCode
Date:   Thu Mar 28 15:59:49 2019 +0100

    Remove "CanChangeAttribute" call

    This API doesn't exist in TBC and wasn't used here in Clique for TBC.
    It's not necessary at all.

commit c3f9c8577fe816fb2a3f9df38fdd921ab1568637
Author: VideoPlayerCode
Date:   Thu Mar 28 15:50:00 2019 +0100

    Remove WotLK Dual Specialization switching code

    This feature doesn't exist in the TBC clients. It was introduced by
    Blizzard in Wrath of the Lich King.

commit 2994945288d74db887f923b7b2e289704b1082ec
Author: VideoPlayerCode
Date:   Thu Mar 28 15:30:11 2019 +0100

    Update TOC file to make TBC version identifiable

    This is mainly to help TinyBook identify that the correct version is
    installed.

commit f02bccf7357aeb3b4323c13c14ca8f1c0faa22fa
Author: VideoPlayerCode
Date:   Thu Mar 28 15:27:04 2019 +0100

    Add gitignore file

commit cd7c4b1cd54c68598a33ddc49310c72d5aa292b7
Author: VideoPlayerCode
Date:   Thu Mar 28 15:14:25 2019 +0100

    Import Clique r143

    This was the final version of Clique for Wrath of the Lich King,
    before the r146 Cataclysm rewrite began.

    The main reason for forking this project is that there's no easy
    way to direct TinyBook users to a working, non-bugged version
    of Clique for TBC, since the official CurseForge page has deleted
    version r102, which was pretty decent (at least after TinyBook's
    injected code fixes). But there needed to be a single, all-in-one,
    working version of Clique for TBC. So this is it.

    License included with the original project permits forking and
    modifying the code. We'll be modifying APIs to make it compatible
    with 2.4.3 again, as well as doing a ton of bugfixing/enhancing.

    Quoted from the original project license:

    "Redistribution and use in source and binary forms, with or without
    modification, are permitted..."

    (And if Cladhaire sees this project, he is welcome to import any
    of my bugfixes and enhancements into the latest Clique code.)
```
