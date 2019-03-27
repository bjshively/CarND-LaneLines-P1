# **Finding Lane Lines on the Road** 

[//]: # (Image References)

[result]: ./test_images_output/whiteCarLaneSwitch.jpg "Result"

---

## Reflection

### Pipeline Description

My pipeline consists of the following operations:

- Convert the input image to grayscale
- Detect edges using Canny edge finding
- Apply a mask to the image so that only a region of interest remains
- Apply Hough transformation to identify most significant lines within the image
- Draw the lines identified via Hough transformation on top of the original input image

### `draw_lines()` Improvements

The original `draw_lines()` function would directly draw the exact lines detected by the pipeline. In order to draw lane lines that cover the majority of the image, some enhancements were required.

First, for each line segment identified, I calculated the slope. Lines corresponding to the left side of the road would tend to have a positive slope (up and to the right), but in cv2 the coordinate plane is inverted with the top left corner corresponding to (0, 0). Therefore, our left lines tend to have a negative slope. Using this same logic, lines associated with the right side of the road tend to have a positive slope.

Based on the slope determination, each line is added to either a leftline or rightline list. Once all lines have been classified, the numpy `polyfit` and `poly1d` functions are used to create two regression functions that can be used to extrapolate points on the left and right lines.

Last, we use these two regression functions to plot lines from points in the bottom corners of the image up towards the center, roughly corresponding to the region of interest as masked within our pipeline.

The result looks like this:

![Pipeline output with full lane lines][result]

### Pipeline Shortcomings

One shortcoming with the pipeline as implemented is that occasionally it will fail to detect a line in an image or video frame, or in the video case, the lines detected in consecutive frames will have noticeably difference slopes. This results in the lines blinking on/off of the image, or jittering back and forth as the slope changes.

Another shortcoming is that the pipeline is currently overfitted to the specific dimensions and other characteristics of the input data. It uses a hardcoded region of interest and could therefore easily fail to identify lines accurately if the camera position is altered or even if the vehicle was in the process of making a lane change or other legal maneuver that placed the lines elsewhere in the frame.

### Possible Pipeline Improvements

A possible improvement would be to accumulate data over multiple frames as they are fed into the pipeline, and then to aggregate/average data. This would smooth out changes in the slope of the line being drawn over time, and would reduce the jitter effect that can sometimes be seen as the pipeline detects different edges frame to frame. It's important to note that too much aggregation of data could result in lane changes becoming sluggish or entirely unresponsive, which could be dangerous if a lane turns sharply.

Another potential improvement could be to dynamically adjust the vertices of the region of interest based on the density of edges detected in the Canny edge detection step. This would allow the pipeline to handle a wider variety of images by detecting which portions of the image might contain features of interest.