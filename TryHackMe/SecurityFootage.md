# Security Footage

**Category:** Packet Analysis  
**Difficulty:** Medium  
**Platform:** TryHackMe  

---

## Challenge Overview

The task is to recover video footage from network traffic captured in a pcap file. The challenge involves analyzing HTTP GET requests and following the TCP stream carrying the video data.

Unlike typical packet capture challenges, the streamed video object cannot be directly extracted using common tools like `tshark`. This requires a more creative approach to extract and reconstruct the video frames.

---

## Tools Used

- Wireshark  
- Python 3  
- `opencv-python` (OpenCV)  

---

## Initial Analysis

The provided pcap file was opened in Wireshark. By following the TCP stream of the relevant HTTP GET request, the raw data was inspected.

Within the stream, multiple JPEG images were identified by the presence of the JFIF header, indicating that the video footage was transmitted as a sequence of JPEG frames.

---

## Data Extraction

The raw data from the TCP stream was saved as a `.bin` file. Since the video frames were embedded as consecutive JPEG images within this binary file, a Python script was written to:

- Parse the `.bin` file  
- Identify image signatures (JPEG, PNG, GIF)  
- Extract each image sequentially  
- Save the images into a dedicated folder  

This extraction resulted in 541 images, each representing a frame of the original video footage.

---

## Video Reconstruction

With the extracted images saved in order, the next step was to reconstruct the video.

Another Python script was developed to:

- Read all extracted images in natural sorted order  
- Combine the images as frames into a video file (MP4)  
- Use OpenCV to write the video at a specified frame rate  

A Python virtual environment was created to manage dependencies, and the required packages were installed before running the video creation script.

The output was a playable video file revealing the security footage and ultimately the flag.

---

## Code Snippets

### Image Extraction Script

```
import os

# Folder to save extracted images
output_folder = "extracted_images"
os.makedirs(output_folder, exist_ok=True)

# Image file signatures and their respective file extensions and end markers
image_signatures = {
    b'\xff\xd8\xff': {  # JPEG start
        'extension': '.jpg',
        'end_marker': b'\xff\xd9'  # JPEG end
    },
    b'\x89PNG\r\n\x1a\n': {  # PNG start
        'extension': '.png',
        'end_marker': b'IEND\xaeB`\x82'  # PNG end chunk
    },
    b'GIF87a': {  # GIF start
        'extension': '.gif',
        'end_marker': b'\x00;'  # GIF end
    },
    b'GIF89a': {  # GIF start
        'extension': '.gif',
        'end_marker': b'\x00;'  # GIF end
    }
}

def find_images(data):
    images = []
    pos = 0
    data_len = len(data)

    while pos < data_len:
        # Find the earliest image signature in the remaining data
        matches = []
        for sig in image_signatures:
            idx = data.find(sig, pos)
            if idx != -1:
                matches.append((idx, sig))
        if not matches:
            break

        # Pick the earliest found signature
        start_idx, sig = min(matches, key=lambda x: x[0])
        end_marker = image_signatures[sig]['end_marker']
        ext = image_signatures[sig]['extension']

        # Search for the end marker starting from start_idx
        if sig == b'\x89PNG\r\n\x1a\n':
            # PNG end marker is a chunk, so we search for it after start_idx
            end_idx = data.find(end_marker, start_idx)
            if end_idx != -1:
                # PNG end marker length is 8 bytes, include it fully
                end_idx += len(end_marker)
            else:
                # If no end marker found, break
                break
        elif sig in [b'GIF87a', b'GIF89a']:
            # GIF end marker is 2 bytes
            end_idx = data.find(end_marker, start_idx)
            if end_idx != -1:
                end_idx += len(end_marker)
            else:
                break
        else:
            # JPEG end marker 2 bytes
            end_idx = data.find(end_marker, start_idx)
            if end_idx != -1:
                end_idx += len(end_marker)
            else:
                break

        # Extract image bytes
        img_data = data[start_idx:end_idx]
        images.append((img_data, ext))

        # Move position forward
        pos = end_idx

    return images

def main():
    bin_file = "image.bin"  # Replace with your .bin file path

    with open(bin_file, "rb") as f:
        data = f.read()

    images = find_images(data)

    print(f"Found {len(images)} images.")

    for i, (img_data, ext) in enumerate(images, start=1):
        filename = os.path.join(output_folder, f"image_{i:03d}{ext}")
        with open(filename, "wb") as img_file:
            img_file.write(img_data)
        print(f"Saved {filename}")

if __name__ == "__main__":
    main()
```

### Video Creation Script

```
import cv2
import os
import re

def natural_sort_key(s):
    # Helper function to sort filenames naturally (image_1, image_2, ..., image_10)
    return [int(text) if text.isdigit() else text.lower() for text in re.split(r'(\d+)', s)]

def create_video_from_images(image_folder, output_video_path, fps=24, frame_size=None):
    # Get list of image files sorted naturally
    images = [img for img in os.listdir(image_folder) if img.lower().endswith(('.png', '.jpg', '.jpeg', '.gif'))]
    images.sort(key=natural_sort_key)

    if not images:
        print("No images found in the folder.")
        return

    # Read the first image to get frame size if not provided
    first_image_path = os.path.join(image_folder, images[0])
    frame = cv2.imread(first_image_path)
    if frame is None:
        print(f"Error reading the first image: {first_image_path}")
        return

    if frame_size is None:
        height, width, layers = frame.shape
        frame_size = (width, height)

    # Define the codec and create VideoWriter object
    fourcc = cv2.VideoWriter_fourcc(*'mp4v')  # or 'XVID', 'avc1'
    video_writer = cv2.VideoWriter(output_video_path, fourcc, fps, frame_size)

    for img_name in images:
        img_path = os.path.join(image_folder, img_name)
        img = cv2.imread(img_path)
        if img is None:
            print(f"Warning: skipping unreadable image {img_path}")
            continue

        # Resize if needed
        if (img.shape[1], img.shape[0]) != frame_size:
            img = cv2.resize(img, frame_size)

        video_writer.write(img)

    video_writer.release()
    print(f"Video saved to {output_video_path}")

if __name__ == "__main__":
    folder = "extracted_images"  # Folder where your 540 images are saved
    output_video = "output_video.mp4"  # Output video file path
    fps = 24  # Frames per second

    create_video_from_images(folder, output_video, fps)
```

---

## Flag

The final video revealed the flag embedded within the footage.
```
flag{5ebf457ea66b2877fdbca2de9ec86f31}
```

---

## Key Takeaways

- Packet captures can contain complex data streams that require creative extraction methods.  
- Following TCP streams and inspecting raw data is essential for analyzing embedded multimedia.  
- Automating extraction and reconstruction with Python greatly simplifies handling large volumes of data.  
- Virtual environments help manage dependencies and ensure reproducibility.  

---

## Conclusion

This challenge demonstrated practical skills in packet analysis, data extraction, and multimedia reconstruction from network traffic. By combining manual inspection with scripting, the video footage was successfully recovered, showcasing the importance of versatile approaches in cybersecurity challenges.
