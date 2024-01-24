# ANALYSING A BINARY FILE
## Latifah Mojisola Salaudeen
The CFG Introduction to Python MOOC in November, 2023 included a challenge which involved analysing a binary file for human readable content. The challenge tasks (link) included extracting video frames and syncronisation words. 

I first opened the file in a hex viewer to have an overview of it. One of the first things I noticed was that there were dates in it.

First, I'll like to open the binary file and check for images. 

```python
# Defining image markers and trailers
# PNG
pngMarker = b'\x89\x50\x4E\x47\x0D\x0A\x1A\x0A'
pngTrailer = b'\x49\x45\x4E\x44\xAE\x42\x60\x82'

# GIF
gifMarker1 = b'\x47\x49\x46\x38\x37\x61'
gifMarker2 = b'\x47\x49\x46\x38\x39\x61'
gifTrailer =  b'\x00\x3B'

## JPEG
jpegMarker = b'\xFF\xD8\xFF'
jpegTrailer = b'\xFF\xD9'

## JBIG2
jbig2Marker = b'\x97\x4A\x42\x32\x0D\x0A\x1A\x0A'
jbig2Trailer = b'\x03\x33\x00\x01\x00\x00\x00\x00'
```
```python
# open the file and check for the markers
#wrapping this in a try block to catch errors
try:
    with open("dstl_MOOC_Challenge_v1.bin", "rb") as file:
        readFile = file.read()
        if pngMarker in readFile:
            print("PNG in file")
        elif jpegMarker in readFile:
            print("jpeg in file")
        elif gifMarker1 in readFile or gifMarker2 in readFile:
            print("GIF in file")
        elif jbig2Marker in readFile:
            print("JBIG2 in file")
        else: print("No images in file")
except FileNotFoundError: 
        print("File not found.")
```
Since JPEG is in file, I'd like to get the markers, trailers, and the data in between those.
First, I'll get the indexes of the markers and trailers in the binary document.


```python
#Using regex, find the positions(index) of all the markers and trailer
import re

#A function to find Jpeg signatures in a file and return its markers and trailers
def findJpegSignatures(filePath):
    try:
        with open(filePath, "rb") as file:
            readFile = file.read()

            # defining empty lists to hold markers and trailers indices
            markersIndex = []
            trailersIndex = []

            #finditer() creates an iterable object
            markersSpan = re.finditer(jpegMarker, readFile)
            trailersSpan = re.finditer(jpegTrailer, readFile)

            for i in markersSpan:
                markersIndex.append((i.start()))

            for i in trailersSpan:
                trailersIndex.append((i.start()))

            return markersIndex, trailersIndex
        
    except FileNotFoundError: 
        print("File not found.")

    
#get data between marker and trailer
dstlFile = "dstl_MOOC_Challenge_v1.bin"

markersIndex, trailersIndex = findJpegSignatures(dstlFile)

with open(dstlFile, "rb") as file:
    readDstlFile = file.read()

for i in range(len(markersIndex)): #loop runs for the number of items in markersIndex
    outputFilename = f'{"extracted-images/pic"}{i+1}.jpeg'
    with open(outputFilename, 'wb') as jpegFile:
        jpegFile.write(readFile[markersIndex[i]:trailersIndex[i+1]])
    
    print(f"JPEG extracted and saved as '{outputFilename}'")
```

Now, I want to extract the rest of the data minus the jpeg. I can move from index 0 to the first marker; then move from trailer to marker (from trailer 1 to marker 2)

```python
markersIndex, trailersIndex = findJpegSignatures(dstlFile)

with open(dstlFile, "rb") as file:
    readDstlFile = file.read()


#empty string to hold the bin data
nonJpegData = b""

#from beginning of the file to the first marker
nonJpegData += readDstlFile[0:markersIndex[0]] 

#can't do marker 1(index 0) cos there's no trailer before
#loop runs for the number of items-1 in markersIndex (89th marker is in index 88)
for i in range(len(markersIndex)-1): 
    nonJpegData += readDstlFile[trailersIndex[i]:markersIndex[i+1]] #first iteration = trailer1 to marker 2

#append trailer 89 to the end
nonJpegData += readDstlFile[trailersIndex[len(markersIndex)]:]

with open("nonJpegData.bin", 'wb') as output_file:
    output_file.write(nonJpegData)

    print(f'Non-JPEG data extracted and saved to {"nonJpegData"}')
```

With Non-JPEG data extracted. I'd like to see what that looks like as txt.

``` python
# A function to convert a binary file to a text file
def binaryToText(inputFile, outputFile, encoding='utf-8', errors="ignore"):
    try:
        # Read input file
        with open(inputFile, 'rb') as file:
            binaryData = file.read()
        
        # Decode
        textFromBinary = binaryData.decode(encoding, errors)

        # Write decoded text to file
        with open(outputFile, 'w') as file:
            file.write(textFromBinary)
        
        print(f'Decoded text saved to {outputFile}')
    
    except FileNotFoundError as e:
        print(f"Error: {e}")
    except UnicodeDecodeError as e:
        print(f"Error decoding binary data: {e}")

#converting nonJpegData to text
inputFile = 'nonJpegData.bin'  
outputFile = 'nonJpegDataFromBinary-utf8.txt'

binaryToText(inputFile, outputFile)
```

The first thing I noticed was that there are multiple dates and time within the text. 
I went on a pattern finding 'mission', trying to see if there are patterns to 

``` python
#finding what's between the pictures
# what is the length of the characters between the images
markersIndex, trailersIndex = findJpegSignatures(dstlFile)

for i in range(len(markersIndex)-1):
    print(trailersIndex[i]-markersIndex[i+1])

print(len(readFile)-trailersIndex[89])

#it seems there are 241 characters between the jpeg images and 225 from the end of the trailer to the end of the file
```


``` python
beforeFirstImage = readFile[:markersIndex[0]]
beforeFirstImageText = beforeFirstImage.decode(encoding="ascii", errors="ignore")
print("beforeFirstImage as hex: ",beforeFirstImage)
print("beforeFirstImage as txt: "+beforeFirstImageText)
#the first characters before the first image seems to contain null data
```

Trying to see the last 225 indices (bytes) of the data
``` python
last225bytes = readFile[-225:]
# print(last225bytes)
decodedLast225Bytes = last225bytes.decode(encoding="ascii", errors="ignore")
print(decodedLast225Bytes)
```
Is there a pattern between the 241 bytes of data between the images?
``` python
bytesBetweenImages = b''
decodedbytesBetweenImages = ""
for i in range(len(markersIndex)-1):
    # print(readFile[trailersIndex[i]:markersIndex[i+1]], "\n")
    bytesBetweenImage = readFile[trailersIndex[i]:markersIndex[i+1]]
    decodedbytesBetweenImage = bytesBetweenImage.decode(encoding="ascii", errors="ignore")+"\n\n"
    decodedbytesBetweenImages+=decodedbytesBetweenImage
    
print(decodedbytesBetweenImages)

# Write decoded text to file
with open("decodedbytesBetweenImages.txt", 'w') as file:
    file.write(decodedbytesBetweenImages)

print("Decoded text saved")
```

``` python

```

``` python

```