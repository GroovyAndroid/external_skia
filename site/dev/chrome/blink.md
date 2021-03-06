Blink layout tests
==================

How to land Skia changes that change Blink layout test results.

Changes that affect a small number of layout test results
---------------------------------------------------------
Changes affecting fewer than ~20 layout tests can be rebaselined without
special coordination with the Blink gardener using these steps:

1. Prepare your Skia change, taking note of which layout tests will turn red
   \(see http://www.chromium.org/developers/testing/webkit-layout-tests for more
   detail on running the Blink layout tests\).
2. Check in your code to the Skia repo.
3. Ahead of the Skia auto roll including your change, manually push a change to the
   Blink LayoutTests/TestExpectations [file](http://src.chromium.org/viewvc/blink/trunk/LayoutTests/TestExpectations),
   flagging tests expected to fail as a result of your change with \[ NeedsManualRebaseline \].
4. Wait for the Skia roll to land successfully.
5. Check in another change to the Blink TestExpectations file changing the flags to
   \[ NeedsRebaseline\], which will prompt the automatic rebaseline.



Changes that affect a large number of test results
--------------------------------------------------
Where a 'large number' or 'many' means more than about 20.
Follow the instructions below:

In the following the term 'code suppression' means a build flag \(a\.k\.a\. define\).
Such code suppressions should be given a name with the form SK\_IGNORE\_xxx\_FIX.

There are dependency revisions which must be updated in this process. Updating
a dependency revision is called a 'roll'.  There are two different rolls which
concern this process, each of which happens via an Auto Roll Bot multiple
times per day, and can also be done manually:

  * Skia roll into Chromium. See
https://chromium.googlesource.com/chromium/src/+log/master/DEPS and search for
skia\-deps\-roller.
  * Blink roll into Chromium. See
https://chromium.googlesource.com/chromium/src/+log/master/DEPS and search for
blink\-deps\-roller.

### Setup
#### Code suppression does not yet exist \- Direct method
1. Make a change in Skia which will change many Blink layout tests.
2. Put the change behind a code suppression.
3. Check in the change to the Skia repository.
4. Manually roll Skia or append the autoroll with the code suppression to
   Chromium's 'skia/skia\_common\.gypi'
5. Add code suppression to Blink's 'public/blink\_skia\_config\.gyp'.
6. Wait for Blink roll into Chromium.
7. Remove code suppression from Chromium's 'skia/skia\_common\.gypi'.

#### Code suppression does not yet exist \- Alternate method
1. Add code suppression to Blink's 'public/blink\_skia\_config\.gyp' before making code
   changes in Skia.
2. Make a change in Skia which will change many Blink layout tests.
3. Put the change behind a code suppression.
4. Wait for Blink roll into Chromium.
5. Check in the change to the Skia repository.
6. Wait for Skia roll into Chromium.

#### Code suppression exists in header
1. Remove code suppression from header file in Chromium and add code suppression to
   Chromium's 'skia/skia\_common\.gypi'.
   The code suppression cannot be in a header file and a defined in a gyp file at the
   same time or a multiple definition warning will be treated as an error and break
   the Chromium build.
2. Add code suppression to Blink's 'public/blink\_skia\_config\.gyp'.
3. Wait for Blink roll into Chromium.
4. Remove code suppression from Chromium's 'skia/skia\_common\.gypi'.

### Rebaseline
1. Choose a time when the Blink tree is likely to be quiet. Avoid PST afternoons in
   particular. The bigger the change, the more important this is. Regardless,
   determine who the Blink gardener is and notify them. You will be making the
   Chromium\.WebKit tree very red for an extended period, and the gardener needs to
   know that they are not expected to fix it.
2. Create a CL removing the code suppression from Blink's
   public/blink_skia_config.gyp while simultaneously adding [ NeedsRebaseline ]
   lines to Blink's LayoutTests/TestExpectations [file](http://src.chromium.org/viewvc/blink/trunk/LayoutTests/TestExpectations).
   Then the auto rebaseline bot will take care of the work of actually checking in the
   new images. This is generally acceptable for up to 600 or so rebaselined images.
   Above that you might still use [ NeedsRebaseline ], but it's best to coordinate with
   the gardener/sheriff. This should go through the CQ cleanly.
3. Be careful with tests that are already failing or flakey. These may or may not need
   to be rebaselined and flakey tests should not be removed from TestExpectations
   regardless. In such cases revert the TestExpectations changes before committing.
4. If you are not the one handling the cleanup step, please open a Skia Issue of the
   form
   Title: "Remove code suppression SK\_IGNORE\_xxx\_FIX\."
   Comment: "Code suppression SK\_IGNORE\_xxx\_FIX rebaselined with Blink revision
   123456\." and assign it to the individual responsible for the cleanup step.

### Cleanup
1. Wait for Blink roll into Chromium, so that Chromium is using the new Skia code
   and new Blink baselines.
2. Remove the now unused old code from Skia and any defines which were introduced
   to suppress the new code.
3. Check in the cleanup change to the Skia repository.
4. Wait for Skia roll into Chromium.

