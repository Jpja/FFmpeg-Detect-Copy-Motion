# FFmpeg Detect & Copy Motion  
Detects motion in video files, and copies to new, separate video clips.

### Introduction

FFmpeg Detect & Copy Motion (FDCM) automatically detects motion in video files. Each video motion event is saved to a new, separate video file. It works on most videos with a fixed camera angle and (almost) motionless background.

FDCM is optimized for speed. A 1080p video should be processed 3-5x faster than the duration of the input file.

The extracted files are lossless copies from the input file bitstream.

FFmpeg and Python 3 are required.

### Intended Users

Vloggers and other video makers who shoot with a fixed camera (e.g. webcam or from a tripod), and:
- video is one long continuous recording
- some time is spent outside the frame
- video would be better split into separate clips

### Intended Workflow

- Import raw video files to the same folder as this .py script. Do not keep any other video files in this folder.
- Rename all files such that the first two letters remind you of that file's content. If more files contain similar content, rename them in bulk, and keep chronological ordering.
- Run the script. No arguments are needed.
- Take a coffee break...
- Watch the output clips. Delete false positives (e.g. triggered by shadows) and make sure no clip starts too late or ends too early.
   - If so, consider deleting all output files, adjust parameters in the script file and run again. Or simply edit out this part manually.
- When output clips are fine, copy them over to your video project's folder. Raw files can be deleted.

### Under the Hood

A bit simplified, this is how the motion detection works
- Run FFmpeg command `ffmpeg -i INPUT_FILE -vf select='not(mod(n\,20))',select='gte(scene,0)',metadata=print:file=TEMP_FILE -an -f null -` to get scene detection scores for every 20 frames
- Find the motion threshold score. It tries to set the threshold such that a motionless duration of >7 sec is found. If not found (e.g. there is no motionless period), fall back to default.
- Each frame gets a CHANGE score of 1 if above threshold, otherwise 0
- The TRIGGER score is set to 1 if two in a row have changed, and at least one of the next ten also has changed. If ten in a row has no change, set score to -1. Otherwise 0.
- Set times to START COPY and END COPY based on trigger scores. It cannot start copy during the first two seconds of the input file. The end of copying cannot happen if there's a new motion trigger within next 5.9 sec, and an end of copy if forced 2 sec before the end of the input file. 
- Finally subtract 2 sec from each start copy point and add 2 sec to every end copy point.
- For each start and end point found, run ffmpeg command `ffmpeg -ss COPY_START -i INPUT_FILE -t CLIP_LENGTH -c copy OUTPUT_FILE`.

All parameters can be adjusted near the top of the script file.

### Issue

FFmpeg's scene detection algorithm compares a frame with the previous selected frame. This requires a high degree of continuous motion, and an undesirable cut-off may happen if the subject is relatively motionless. FDCM compensates for this by ignoring a cut-off if there's less than 5.9 seconds to motion is detected again.

Ideally the motionless periods with the subject outside the frame should be even longer. If your raw videos have such long breaks, increase the `min_copy_break_s` parameter accordingly.

If I were to make a new project I'd experiment with OpenCV, and have it compare the current frame with the background frame PRIOR TO motion is triggered.

### Related Projects

- [FFmpeg Motion Binary Sensor](https://www.home-assistant.io/components/binary_sensor.ffmpeg_motion/)
 - I discovered this one after I made this script. TODO: Test speed/accuracy vs my script
- [Basic motion detection and tracking with Python and OpenCV](https://www.pyimagesearch.com/2015/05/25/basic-motion-detection-and-tracking-with-python-and-opencv/)
 - To download code you must submit email address, but I did and they did email me legit code
 - Uses OpenCV to visualize motion. Promisising approach. A good starting point for a similar (and possibly better) script
- [PySceneDetect](https://pyscenedetect.readthedocs.io/en/latest/)
 - Command line tool. Accurate result but oo slow for my liking. Perhaps I missed some parameter to speed it up?

### MIT License

Copyright (c) 2018 JP Janssen

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
