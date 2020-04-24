# **Finding Lane Lines on the Road**

### Requirment_1:

Build a pipeline in python openCV that takes road video as input and return an annotated video stream as output.
Two videos are available to run our code: "solidWhiteRight.mp4" and "solidYelloLeft.mp4". So our code should detect lane lines of any colour.

### Requirment_2:

Left and right lane lines shall be accurately annotated throughout almost all of the video.

### Requirment_3:

Final output of two given videos should look like "P1_example.mp4" where Solid lines are used to annotate.

Hint: Helper function in the template can be used to build the pipeline.

## Reflection

Video is nothing but a sequence of images.

Pipeline is build using the helper functions in the template to process the image. Once all the images in ".../test_images" folder is tested for proper working, the pipeline is used to test the two videos "solidWhiteRight.mp4" and "solidYelloLeft.mp4". During the process the parameter in the pipeline are adjusted to make pipeline work properly for video.

### 1. Pipeline.

One of the test image is used to explain the pipeline.

![](/examples/steps/1.jpg)
Fig 1: Test Image

#### 1.1. Grayscale the input image

In the first step, original image [3-dimensional] is converted to greyscale image [1-demnsional]. In a greyscale image 0 represents Black, 255 represents White and values in between are shades ranging from black to white.

![](/examples/steps/2.jpg)
Fig 2: Gray scaled image of test image

#### 1.2. Gaussian smoothing the Gray Image

To suppress the noisy and erroneous gradient, grey image is processed with Gaussian Blur. It is recommended to use odd values for kernel size [k = 3, 5, 7...], in this pipeline kernel size of 3 is used.

![](/examples/steps/3.jpg)
Fig 3: Blurred Image

#### 1.3. Canny edge detection

This algorithm computes gradient of the image and the brightness in the image indicates the strength of gradient at each pixel. By tracing these strong pixels edges are detected. openCV edge detection uses two threshold "lower and higher" to identify the strong pixels. John Canny has recommended a ratio of 1:2 or 1:3 for these thresholds. As the grey image each pixel has value range of 0 to 255, from the mid value lower and higher threshold is selected with ration of 1:2 which is low_Threshold = 80 and higher_Threshold = 160.

Canny edge detection has its own Gaussian smoothing with kernel size of 5, still it is recommended to use additional GausianBlur in previous step.

![](/examples/steps/4.jpg)
Fig 4: Image with edge detected

#### 1.4. Find region of Interest on the image

From the edge detected image select only the region which will correspond to the lanes lines between which Vehicle is expected to be travelling. In an image Origin (0,0) is at left-top, x-axis is left to right and y-axis is top to bottom. As we know image size is 960x540, region of interest is selected from the vehicle point of view with vertices (60,539),(420, 335), (540, 335), (890,539).

![](/examples/steps/5.jpg)
Fig 5: Edge detected image is masked to region of interest

#### 1.5. Identify and Draw Lane Lines

**1.5.1. Hough Transformation to Find lines**

OpenCV HoughLinesp() is used to find all possible lines in region of interest. Parameters of value rho = 1 pixel, theta = pi/180 [1 degree], line threshold=15 [number of points on a line], Min line length of 40 pixels and max gap between lines of 20 pixels are used to run. All identified lines with thickness of 1 are drawn on black image in Red colour and this image merged to test image is shown below.

![](/examples/steps/6.jpg)
Fig 6: Lines detected by Hough Transform and the lines drawn on Test Image.

**1.5.2. Averaging and Extrapolation**

As the Requirement_3 requires one solid line to be drawn on left and right lane lines. The draw_lines() function is modified such that lines are passed on to additional function find_lneLines() where following steps were done.
  1. slope, Y-Intercept and length of line segment are calculated using mathematical formulas.
Slope is 0 for vertical line, x/0 for horizontal line, negative for left lane and positive for right lane
  2. In-order to remove noise consider only lines with slope > 0.5 and slope < -0.5 and line length > 20.
  3. slope, Y-Intercept of right lane line [slope > 0.5] and left lane lines [slope < -0.5] are averaged separately.
  4. With the averaged slope and y-Intercept of left lane lines find the points at y = 540 and y = 320 and Construct left line segment.
  5. With the averaged slope and y-Intercept of right lane lines find the points at y = 540 and y = 320 and Construct right line segment.
Two sold lines determined from all identified Hough lines are shown below.

![](/examples/steps/7.jpg)
Fig 7: Two solid lines averaged and extrapolated from Hough lines

Thickness of final lines are increased and merged with test image.

![](/examples/steps/8.jpg)
Fig 8: Output image of pipeline

### 1.6 Video processed
![](/examples/steps/solidWhiteRight_0.gif)

First video processed with code to find lane lines.

![](/examples/steps/solidWhiteRight_1.gif)

![](/examples/steps/solidYellowLeft_0.gif)

Second video processed with code to find lane lines.

![](/examples/steps/solidYellowLeft_1.gif)

### 2. Shortcomings with your current pipeline
  1. Pipeline not working for curved road with same parameter values.
  2. In few frames of "solidYelloLeft.mp4" code failed with an error, as no right Hough lines were detected in those frames it lead to "division by Zero". So additional check was done to draw a solid left or right line only when atleast one left or right Hough line is detected. Which means there could be frames in output video where left, right or both lines could be missing which are not visible to human eyes.

### 3. Suggest possible improvements to your pipeline
  Lane lines are shaken which can be improved.
