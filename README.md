# Header-Only QOI Codec Library
A one header file library for encoding and decoding QOI files in a stream.

# Features
- A much simpler and abstracted code
- No 400MB limit found in reference QOI implementation
- No extra memory allocation while running this codec[^1]
- [Programmed in freestanding C useful in embedded systems](https://en.cppreference.com/w/c/language/conformance)

[^1]: You must provide a pointer to an existing array to run this codec

## How to Install Library
Place the *qoi.h* file into your project folder preferably in your project's include folder

Define the following below before you include *qoi.h* library in **one** of your source code file

	#define  QOI_IMPLEMENTATION
	#include "qoi.h"
	
## Minimal implementation
### Encoder
	/* 	
		Assume you open the file and
		allocated enough memory for the encoder
		before this code
	*/
	
	qoi_desc_t desc;
	qoi_enc_t enc;
	uint8_t* pixel_seek;
	
	qoi_desc_init(&desc);
	qoi_set_dimensions(&desc, width, height);
	qoi_set_channels(&desc, channels);
	qoi_set_colorspace(&desc, colorspace);

	write_qoi_header(&desc, qoi_file);  

	pixel_seek = file_buffer;
	qoi_enc_init(&desc, &enc, qoi_file);

	while(!qoi_enc_done(&enc))
	{
		qoi_encode_chunk(&desc, &enc, pixel_seek);
		pixel_seek += desc.channels;
	}

	/* 
		Write the QOI file after encoding or do something else
		or do something else after encoding
	 */
### Decoder
	/* After reading a QOI file and placed in buffer */
	
	qoi_desc_t desc;
	qoi_dec_T dec;
	qoi_pixel_t px;
	
	uint8_t *bytes;
	size_t rawImageLength, seek;
	
	qoi_desc_init(&desc);
	qoi_initalize_pixel(&px);
	qoi_set_pixel_rgba(&px, 0, 0, 0, 255);
	
	if (!read_qoi_header(&desc, qoi_bytes))
	{
		printf("The file you opened is not a QOIF file\n");
		return;
	}
	
	rawImageLength = (size_t)desc.width * (size_t)desc.height * (size_t)desc.channels;

	seek = 0;
	if (rawImageLength == 0)
		return;

	qoi_dec_init(&dec, qoi_bytes, buffer_size);
	/* Creates a blank image for the decoder to work on */
	bytes = (unsigned  char*)malloc(rawImageLength * sizeof(unsigned  char) + 4);

	if (!bytes)
		return;

	/* Keep decoding the pixels until
	all pixels are done decompressing */
	while (!qoi_dec_done(&dec))
	{
		px = qoi_decode_chunk(&dec, px);
		/* Do something with the pixel values below */
		bytes[seek] = px.red;
		bytes[seek + 1] = px.green;
		bytes[seek + 2] = px.blue;
		
		if (desc.channels > 3) 
			bytes[seek + 3] = px.alpha;
		
		seek += desc.channels;
	}
	
	/* Use the pixels however you want after this code */

## How to Run Example Programs
### Encoder

    qoi_enc <input file> <width> <height> <channels> <colorspace> <output file>
Input file must be raw RGB or RGBA file

## Software Requirements
 - C99 compiler or C++ compiler
 - [CMake 3.1](https://cmake.org/)

### Decoder
This program only outputs raw RGB or RGBA files depending on the amount of channels in a QOI file

	qoi_dec <input file> <output file>

## References
Thank you to the authors of the source code being used for inspiration for this source code
### Official QOI References
- [QOI Reference Implementation (phoboslab)](https://github.com/phoboslab/qoi)
- [QOI Specification](https://qoiformat.org/qoi-specification.pdf)

### Other QOI Programs
- [Mini-QOI (sharaiwi) (decoder only)](https://github.com/shraiwi/mini-qoi)
- [QOI Explained (YouTube: Reducible)](https://youtu.be/EFUYNoFRHQI?t=1411)
- [FFmpeg](https://github.com/FFmpeg/FFmpeg)
	- [Encoder](https://github.com/FFmpeg/FFmpeg/blob/master/libavcodec/qoienc.c)
	- [Decoder](https://github.com/FFmpeg/FFmpeg/blob/master/libavcodec/qoidec.c)
### Others
- [Byteswap code](https://stackoverflow.com/a/4240014)
