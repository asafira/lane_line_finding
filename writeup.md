# **Finding Lane Lines on the Road**

## Writeup

### Arthur Safira

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./masked_image.png "My Mask"

---

### Reflection

My pipeline up until my use of `draw_lines()` mostly follows what was shown in class; namely,

1. blur grayscale image,
2. apply canny edge detection,
3. mask irrelevant portion of image,
4. apply hough transform to detect lines,

including parameter tuning. Notably and unlike in class, I used a fairly harsh mask:

![alt text][image1]

Following the above process, I also heavily modified `draw_lines()` to take the output from the hough line detection and reasonably extrapolate full lanes from it. Specifically, I

1. categorized lines into left/right lanes based off of their slope, defining a window in which reasonable lanes would have a slope (otherwise throwing away lines)
2. performed a weighted average of the categorized slopes and line endpoints, weighed by the line's original length. Intuitively, we have more trust in longer lines coming from real lane markers, hence the weighted average.
3. drew lane markings from the bottom of the image to top-most point for which there was a line detected; the line intercepted the center-of-mass of each lanes' line segments and had slope equal to average slope. (Both were calculated in 2)

Lastly, if upon categorization no lines were bucketed as right lanes, or no lines were bucketed left lanes, the entire algorithm recursively calls itself but lowers canny threshold parameters to become more sensitive to edges in the image. This is especially important in lower-contrast images, such as those found in the challenge video.


### 2. Identify potential shortcomings with your current pipeline

One shortcoming to my current pipeline is that it is still fairly susceptible to noise, and it shows both in the lanes' jitter during video playback as well as 1-3 awkward frames during the challenge video.

Another shortcoming is that the pipeline is currently somewhat slow --- I paid little to no attention to code efficiency, and there is a lot of room for improvement.

As a final shortcoming, related to the first point in this section, is that the pipeline makes no use of previous frames, acting "Markovian" despite there being potential for making better lane decisions based on previous frames. This allows our lanes to undergo shot-to-shot variations that are extremely atypical, if not just unphysical.


### 3. Suggest possible improvements to your pipeline

I believe there are large gains to be made by using some number of previous frames in deciding the lanes for the current frame in a video. Specifically, it is unphysical for a lane to change slope dramatically over the course of a single frame (~30 ms), and such instances can be suppressed with the proper prior knowledge of the previous frames' slope. Also, the frame-by-frame jitter can be reduced by averaging with previous frames and/or a more sophisticated method.

While I don't think the purpose of this assignment was to create the most efficient code, the code has a lot of low-hanging fruit for optimization, including optimizing the recursive function call (so as not to re-create variables over and over again), preallocating array sizes, and . A proper answer, however, would first look at a profiler to understand the limiting parts of the code.

I also think I should probably rely less on such a stringent mask; I am somewhat worried that there are instances where lanes will not fall in the region I created (extra-wide lanes?).
