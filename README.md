PyScreeze
=========

PyScreeze is a simple, cross-platform screenshot module for Python 3.

About
-----

PyScreeze can take screenshots, save them to files, and locates images within the screen. This is useful if you have a small image of, say, a button that needs to be clicked and want to locate it on the screen.

NOTE - PyScreeze depends on Pillow for screenshots, whose older versions have security issues. The only secure versions of Pillow are 8.3.2 and later. However, Pillow 8.3.2 only supports Python as far back as Python 3.6. If you are installing PyScreeze for Python 3.5 or earlier, be aware that there may be security issues with the version of Pillow it uses.

If Pillow is unavailable, there are fallback methods for each OS (screencapture for macOS, gnome-screenshot for Linux, etc.)

Special Notes About Ubuntu
==========================

If your Ubuntu system uses the older X11 window system, you must install Xlib by running `sudo apt install python3-xlib`. This will require the admin password for your computer.

The screenshot() Function
=========================

Calling `screenshot()` will return an Image object (see the Pillow or PIL module documentation for details). Passing a string of a filename will save the screenshot to a file as well as return it as an Image object.

    >>> import pyscreeze
    >>> im1 = pyscreeze.screenshot()
    >>> im2 = pyscreeze.screenshot('my_screenshot.png')

On a 1920 x 1080 screen, the `screenshot()` function takes roughly 100 milliseconds - it's not fast but it's not slow.

There is also an optional `region` keyword argument, if you do not want a screenshot of the entire screen. You can pass a four-integer tuple of the left, top, width, and height of the region to capture:

    >>> import pyscreeze
    >>> im = pyscreeze.screenshot(region=(0,0, 300, 400))

The Locate Functions
====================

You can visually locate something on the screen if you have an image file of it. You can call the `locateOnScreen('calc7key.png')` function to get the screen coordinates of the 7 button for a calculator app. The return value is a 4-integer tuple: (left, top, width, height). This tuple can be passed to `center()` to get the X and Y coordinates at the center of this region. If the image can't be found on the screen, `locateOnScreen()` returns `None`. (PyAutoGUI is a module that simulates mouse clicks.)

    >>> import pyscreeze, pyautogui
    >>> button7location = pyscreeze.locateOnScreen('calc7key.png')
    >>> button7location
    (1416, 562, 50, 41)
    >>> button7x, button7y = pyscreeze.center(button7location)
    >>> button7x, button7y
    (1441, 582)
    >>> pyautogui.click(button7x, button7y)  # clicks the center of where the 7 button was found

The `locateCenterOnScreen()` function is probably the one you want to use most often:

    >>> import pyscreeze, pyautogui
    >>> x, y = pyscreeze.locateCenterOnScreen('calc7key.png')
    >>> pyautogui.click(x, y)

On a 1920 x 1080 screen, the locate function calls take about 1 or 2 seconds. This may be too slow for action video games, but works for most purposes and applications.

If speed is important, install the optional opencv library (`pip install cv2`). The `locateAll` computation will use it if available, and take less than 1 millisecond to find all matches in a full-screen search. (This does not include the time required to capture a screenshot.)

There are several "locate" functions. They all start looking at the top-left corner of the screen (or image) and look to the left and then down. The arguments can either be a

- `locateOnScreen(image, grayscale=False)` - Returns (left, top, width, height) coordinate of first found instance of the `image` on the screen. Returns None if not found on the screen.

- `locateCenterOnScreen(image, grayscale=False)` - Returns (x, y) coordinates of the center of the first found instance of the `image` on the screen. Returns None if not found on the screen.

- `locateAllOnScreen(image, grayscale=False)` - Returns a generator that yields (left, top, width, height) tuples for where the image is found on the screen.

- `locate(needleImage, haystackImage, grayscale=False)` - Returns (left, top, width, height) coordinate of first found instance of `needleImage` in `haystackImage`. Returns None if not found on the screen.

- `locateAll(needleImage, haystackImage, grayscale=False)` - Returns a generator that yields (left, top, width, height) tuples for where `needleImage` is found in `haystackImage`.

The "locate all" functions can be used in for loops or passed to `list()`:

    >>> import pyscreeze
    >>> for pos in pyscreeze.locateAllOnScreen('someButton.png')
    ...   print(pos)
    ...
    (1101, 252, 50, 50)
    (59, 481, 50, 50)
    (1395, 640, 50, 50)
    (1838, 676, 50, 50)
    >>> list(pyscreeze.locateAllOnScreen('someButton.png'))
    [(1101, 252, 50, 50), (59, 481, 50, 50), (1395, 640, 50, 50), (1838, 676, 50, 50)]

Grayscale Matching
------------------

Optionally, you can pass `grayscale=True` to the locate functions to give a slight speedup (about 30%-ish). This desaturates the color from the images and screenshots, speeding up the locating but potentially causing false-positive matches.

    >>> import pyscreeze
    >>> button7location = pyscreeze.locateOnScreen('calc7key.png', grayscale=True)
    >>> button7location
    (1416, 562, 50, 41)

Pixel Matching
--------------

To obtain the RGB color of a pixel in a screenshot, use the Image object's `getpixel()` method:

    >>> import pyscreeze
    >>> im = pyscreeze.screenshot()
    >>> im.getpixel((100, 200))
    (130, 135, 144)

Or as a single function, call the `pixel()` PyScreeze function, which is a wrapper for the previous calls:

    >>> import pyscreeze
    >>> pyscreeze.pixel(100, 200)
    (130, 135, 144)

If you just need to verify that a single pixel matches a given pixel, call the `pixelMatchesColor()` function, passing it the X coordinate, Y coordinate, and RGB tuple of the color it represents:

    >>> import pyscreeze
    >>> pyscreeze.pixelMatchesColor(100, 200, (130, 135, 144))
    True
    >>> pyscreeze.pixelMatchesColor(100, 200, (0, 0, 0))
    False

The optional `tolerance` keyword argument specifies how much each of the red, green, and blue values can vary while still matching:

    >>> import pyscreeze
    >>> pyscreeze.pixelMatchesColor(100, 200, (130, 135, 144))
    True
    >>> pyscreeze.pixelMatchesColor(100, 200, (140, 125, 134))
    False
    >>> pyscreeze.pixelMatchesColor(100, 200, (140, 125, 134), tolerance=10)
    True

Support
-------

If you find this project helpful and would like to support its development, [consider donating to its creator on Patreon](https://www.patreon.com/AlSweigart).
