
# lbi
A custom bitmap format to be simple, human-readable, and efficient.

## example file
[here](https://github.com/nokyoto/lbi/blob/main/colors.lbi)

### real quick
The example file may look like gibberish at first.
If you couldn't tell, LBI is a format written in binary.
Crack open your favorite hex editor and I'll teach you how this format works.

## understanding this
If you didn't notice yet, the only legible text in this is "LB" at the start.
  -- What LB means:
  LB stands for Lake Bitmap. Kind of like Microsoft Bitmaps, this number is used to identify the file format.

1. `0x4C42` - "LB"
The file format number.
2. `0x00` - Compression type
There's no compression at the moment, maybe I'll add some later.
This number will stay 0x00 for the time being.
3. `0x00` - Unused
5. `0x00000004` - Color amount
This is the number of colors used in the image. Good for validation and for specifying the actual number of colors rather than just letting the decoder guess (**VERY** dangerous)
6. `0x00000018` - Palette offset
Tells the decoder where the palette begins.
You'll learn more about palettes later.
7. `0x00040002` - Image resolution
The size of the image.
	- `0x0004` is for the width,
	- `0x0002` is for the height
creating an image that is 4x2.
8. `0x00000002` - Bits per Pixel
Where would we be without BPP? This tells the decoder how many bits make up 1 pixel.
There's a great article on BPP [here](https://www.leadtools.com/help/sdk/dh/to/introduction-bits-per-pixel-and-related-ideas.html).
	### A quick footnote on odd BPPs:
	Odd numbered BPPs (such as 3 or 5) or just BPPs that aren't multiples of 8 in general, will ALWAYS be padded to the nearest byte. For example, say you want 3BPP (8 colors). You wouldn't say `00100101 00101001`, as that extend to another byte and potentially confuse the decoder. So instead, you would say `00100100`, where the decoder can happily ignore the last 2 zeroes and continue reading. Yes, this does mean technically 2 pixels per byte, but let's be honest: at least there's a *somewhat* solution. 
9. `0x00000028` - Data pointer
This tells the decoder the offset in which the actual image begins.
10. Next 16 bytes - Palette
The palette contains all the colors in RGBA hexadecimal. Each color takes up 4 bytes.
Each color will be assigned to a bit sequence in the decoder. In our case, the first color is 0x384AD1FF.
That color will, thanks to our 2 BPP, be assigned to 00. Then for 0x7338D1FF, 01. Then 10. And so on for each color.
11. `0x05AF` - Image
Isn't it funny, how this image has 4 colors and yet fits in 2 bytes? We've come all this way for this. Simple. Number. But it's a lot more important than you'd think.
If you recall back to Bits per Pixel, you would know that the BPP represents how many bits make up 1 pixel. This also helps with USING all the colors you defined before. 2 bits per pixel means that:
	- We can only use 4 colors in our image
	- And each pixel is stored as 2 bits. (e.g. 00, 10)

	Now we know which bits represent which colors.
	Remember the sequence of `0x05AF`? Well let's break this down.
	`05` might seem hard to represent a color with. After all, we only have 4 colors in our palette.
	Well, the magic lies underneath the hexadecimal. Hexadecimal is used to represent binary values as human-readable numbers. With hex, we can simplify `1111` to just `F`.
	Knowing this, `05` in hex will be `00000101` in binary. If you recall back to the previous section---Palette---you'll start to notice. I said earlier that the color `0x384AD1FF`, could be assigned to `00`. So we can represent each color as a 2 bit value referring to palette entries 1, 2, 3, and 4. (That's `0`, `1`, `2`, and `3` if you're really technical)
	With that, binary `00000101` should make a *lot* more sense now. It represents 2 pixels of color 0, and 2 of color 1.
	We can now simply repeat this process for the next byte.
	`AF` = `10101111` = 2 pixels of color 2, and 2 of color 3.

## TL;DR
Face it. You don't need this section. You're already nerdy enough to want to read the specification of an image format.
But here's a quick review of what the bytes mean!
```
0x4C42 - File Identifier
0x00 - Compression type
0x00 - Reserved/Unused
0x00000004 - Color amount
0x00000018 - Palette offset
0x00040002 - Image size (4x2 here)
0x00000002 - Bits per pixel
0x00000028 - Image data offset
Rest - Palette and data
```

## Outro
And with that, we've constructed a beautiful, 4 color, 4x2 image. In just 42 bytes. Take a look for yourself:
```
00000000: 4C 42 00 00 00 00 00 04  LB......
00000008: 00 00 00 18 00 04 00 02  ........
00000010: 00 00 00 02 00 00 00 28  .......(
00000018: 38 4A D1 FF 73 38 D1 FF  8J..s8..
00000020: FF FF FF FF 00 00 00 FF  ........
```
> 40 bytes, the header
```
00000028: 05 AF                    ..
```
> 2 bytes, the actual image

In fact, this header *could* actually be smaller than it is now! I don't quite know how to do that though, and I think it would take away from the goal of simplicity and human-readability.

**To LBI!**
oh and
## One more thing
You might have noticed that in this file, I've not said a thing about metadata. I **do** have plans for it later, but at the moment, I won't touch it with a ten foot pole.

## And before you ask
Yes, it is **big endian**. And yes, I will keep it that way.
Same for the color values.

## Sorry, one more thing
I *did* originally have intentions for the unused/reserved byte.
The reason the 2 values after `LB` are there at all was originally just for alignment and padding. To save space, I put 2 extra things in those spots: compression type, and Bytes per Dimension.
Bytes per Dimension was simple, really; to future-proof this format, I decided to allow additional bytes for the dimensions. This means that if, somehow, this format needed to be used for an image larger than 65535x65535 (the 2 byte maximum is 0xFFFF or 65535) the image could specify how many bytes it actually needed. Later, I realized that it was completely impractical, probably wouldn't be used in this format's lifespan, and would break the layout and structure of the format altogether. I might use this byte for version number or something when it's actually important, but for now it stays `00`.

Also, originally my name for this format was BMI (bitmap image) but I quickly returned to Earth and stopped myself.

Lastly, this format will **NEVER**. **EVER**. SUPPORT PNG/TIFF/RIFF CHUNKS. They're ugly, and they do basically the exact same thing as this format already does.

## Okay, you can leave now

<sub>(C) 2025 NOKYOTO. BMP is a Trademark of Microsoft Inc.</sub>
