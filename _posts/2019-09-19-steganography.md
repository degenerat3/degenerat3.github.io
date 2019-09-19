---
layout: post
title: Memes as an Attack Vector - Steganography
---

*Steganography: The technique of hiding secret data within an ordinary, non-secret, file or message in order to avoid detection*  

This post is about a project I did for a presentation at [RITSEC](https://www.ritsec.club). I'll be briefly describing what steganography is, use cases, then showing an example implementation.  

## What is Steganography? 

As stated above, stego is about hiding some message or data in plain sight, without non-stakeholders even knowing that data is being shared. Steganography is an example of covert communication (sharing data without others realizing it).  There are thousands of examples of covert comms out there, and not all of them are technical. [This talk about a covert channel using lightbulbs](https://www.youtube.com/watch?v=5QCOE8R9bxY) has a pretty good explanation of the topic. Steganography has a variety of implementations which envolve encoding data into photos, videos, sound files, and more. This post will be focusing mainly on stego using images, but the same basic principal can be applied to the other methods.  

## How it Works  

One common method for encoding data into images is referred to as [least significant bit](https://pdfs.semanticscholar.org/c07b/e5a1a6f72efcb0d06a642491ee55d6630585.pdf) steganography.  [This image](https://i.stack.imgur.com/FnULg.png) does a good job showing how this concept is applied to each pixel. 

Since pixel values are represented as numbers (0-255), we can make small adjustments (for example: 255 -> 253) which would not be visible to the naked eye.  These would, however, allow us to save binary data in the *least significant bits* of each pixel value.  This means we can theoretically store a few bits per pixel (exact number varies depending on the implementation) without a noticeable change in the overall image.  In an HD image there could be millions of pixels, allowing us to store a significant amount of data.  

## My Implementation 

The source code is [available here](https://github.com/degenerat3/plainsight) for those that care to follow along.  

The goal of this implementation is to store an executable binary in an image, then later extract it and execute it (in-memory execution is a WIP). This is done by encoding the binary into a base64 string, encoding that string into the image (one pixel per character, for simplicity), then writing the new image to disk.  The b64 character is encoded by finding its order (python `ord()` command), subtracting that value from 255, then using that value as the new 'Alpha' value. Alpha values (the 'A' in RGBA) represent the opacity value of a pixel, so changing it slightly won't effect the overall appearance of the image very much. This example does however produce a noticeable effect, since up to 6 bits out of 8 are changed. This could be changed by using 2 pixels for each b64 character, since that would mean only 3 bits of each pixel are effected. Decoding works in the same (but opposite) way, by examining the pixels to find the b64 string, then decoding/writing the string to disk in order to create a file.  There is a new decoding method in progress which will read the b64 string from the image and write it to a portion of memory, where it can then be executed without ever being written to disk.    

Using the `PIL` library in python allows us to easily interact with image.  With just a few function calls, we can iterate over the entire image and read all the RGBA values. Similarly, we can create a new image and write our adjusted pixel values to it.  

```python
im = Image.open(inputImg)
pix = im.load()
x_size, y_size = im.size  # Get the width and height
sizeTup = (x_size, y_size)
newImg = Image.new('RGBA', sizeTup) # Create a new image, same size
newPix = newImg.load()
counter = 0
binStr = readBin(encFile)   # turn the file into a base64 string
aVals = generateAValues(binStr) # calculate the new A values we'll be writing
for x in range(x_size):
    for y in range(y_size):
        try:
            r, g, b, a = pix[x, y]
        except:
            r, g, b = pix[x, y]
            a = 255
        if counter >= len(binStr):
            newA = a
        else:
            newA = aVals[counter]
        counter += 1
        newPix[x, y] = (r, g, b, newA)
    newImg.save(outputImg)
```

This is just an excerpt of the `encoder.py` file, the full source for the encoder, decoder, and other files can be found in [the repository](https://github.com/degenerat3/plainsight). 
A recording of the presentation and live demo can be [seen here](https://www.youtube.com/watch?v=iKUQROblTxo). The demo starts around ~[4:40](https://youtu.be/iKUQROblTxo?t=280).