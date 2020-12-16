# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[origImage]: ./image/solidWhiteCurve.jpg "Original Image" 
[grayscaleImage]: ./image/grayscaleImage_solidWhiteCurve.png "Grayscale Image"
[gaussianBlurImage]: ./image/gaussianBlurImage_solidWhiteCurve.png "Image After Gaussian Blur"
[cannyImage]: ./image/cannyImage_solidWhiteCurve.png "Image after Canny Edge Detection"
[regionImage]: ./image/regionImage_solidWhiteCurve.png "Image After Masking Region of Interest"
[lineImage]: ./image/lineImage_solidWhiteCurve.png "Image After "

[origLineImage]: ./image/LineOrg.jpg "Original Line Image" 
[Line1Image]: ./image/Line1.png "Line 1"
[Line2Image]: ./image/Line2.png "Line 2"
[Line3Image]: ./image/Line3.png "Line 3"
[Line4Image]: ./image/Line4.png "Line 4"
[Line5Image]: ./image/Line5.png "Line 5"

[challengeInput]: ./image/challengeInput.jpg "Original Challenge Image" 
[challengeOrgLine]: ./image/challengeOrgLine.png "Challenge Original Line Image"
[challengeFilterSlope]: ./image/challengeFilterSlope.png "Challenge Filter Line Slope Image"
[challengeFilter1]: ./image/challengeFilter1.png "Challenge Filter Line 1 Image"
[challengeFilter2]: ./image/challengeFilter2.png "Challenge Filter Line 2 Image"
[challengeFilter3]: ./image/challengeFilter3.png "Challenge Filter Line 3 Image"
[challengeFilter4]: ./image/challengeFilter4.png "Challenge Filter Line 4 Image"
[chanllengeOutput]: ./image/chanllengeOutput.png "Output Challenge Image"
---

### Reflection

### 1. Pipeline Description.

The pipeline consisted of 5 steps.

1. Convert the color image to grayscale image.
2. Use gaussian blue with kernel size `ksize=(3,3)` to remove the noise.
3. Apply the Canny edge detection with lower threshold `low_threshold=50` and higher threshold `high_threshold=80`.
4. Use `cv2.FillPoly()` with polygon `vertices = np.array([[
        (0,imshape[0]),
        (imshape[1] * 10 / 21, 300), 
        (imshape[1] * 11 / 21, 300), 
        (imshape[1],imshape[0])]], dtype=np.int32)` to apply a trapezoid region of interest mask to filter out line segments.
5. Use Hough transform with `rho=3`, `theta=1 degree`, `threshold=70`, `min_line_length=20`,and `max_line_gap=5` to extract the lines.

The following table shows images after each step.
||||
|:--:|:--:|:--:|
|Original|Step 1|Step 2|
|![origImage][origImage]|![grayscaleImage][grayscaleImage]|![gaussianBlurImage][gaussianBlurImage]|
|Step 3|Step 4|Step 5|
|![cannyImage][cannyImage]|![regionImage][regionImage]|![lineImage][lineImage]|

In order to draw a single line on the left and right lanes, I modified the `draw_lines()` function so that it call filter_lines() which includes the following steps.

1. The lines are filtered so that only line with absolute slopes between 0.5 and 1.5 are included.
2. Use the slope to differentiate the left lines and the right lines respectively. Expand all the lines to upper points and bottom points.
3. Take the average of x coordinates of all the upper points of all the left lines and take the average of x coordinates of all the bottom points of all the left lines. It is the left line candidate.
4. Take the average of x coordinates of all the upper points of all the right lines and take the average of x coordinates of all the bottom points of all the right lines. It is the right line candidate.
5. Recompute the left and right lines so that the upper points of both lines would not be higher than the intersection of the left line candidate and right line candidate.

The following table shows line images after each step.
||||
|:--:|:--:|:--:|
|Original|Lines extracted using the above pipeline|Step 1|
|![origLineImage][origLineImage]|![Line1Image][Line1Image]|![Line2Image][Line2Image]|
|Step 2|Step 3|Step 4|
|![Line3Image][Line3Image]|![Line4Image][Line4Image]|![Line5Image][Line5Image]|

To handle challenge video, I modified the `draw_lines()` function so that it call `filter_lines_challenge()` which includes the following steps.

1. The lines are filtered so that only line with absolute slopes between 0.5 and 1.5 are included.
2. The lines are then pass to `filter_lines_challenge()` to filter the lines for 4 time. For each time, They filtering includes:
    1. Filter the lines
        1. If the candidate left line and right line are provideded, reject the lines which are outside of the area of the left line and the right line too much. 
        2. Use the slope to differentiate the left lines and the right lines respectively.
        3. From those left lines, select the longest line. Reject those left lines which are deviated from the longest line and their lengths are shorter than half of the longest line.
        4. From those right lines, select the longest line. Reject those right lines which are deviated from the longest line and their lengths are shorter than half of the longest line.
    2. Compute left line and right line from the above filtered lines.
        1. Expand all the lines to upper points and bottom points.
        2. Take the weighted (by line length) average of x coordinates of all the upper points of all the left lines. and take the weighted (by line length) average of x coordinates of all the bottom points of all the left lines. It is the left line candidate.
        3. Take the weighted (by line length) average of x coordinates of all the upper points of all the right lines and take the weighted (by line length) average of x coordinates of all the bottom points of all the right lines. It is the right line candidate.
        4. Recompute the left and right lines so that the upper points of both lines would not be higher than the intersection of the left line candidate and right line candidate.
        5. Return left line and right line together with those filtered lines.

The following table shows line images after each pass of filter.
||||
|:--:|:--:|:--:|
|Original|Lines extracted using the above pipeline|Filter using Slope|
|![challengeInput][challengeInput]|![challengeOrgLine][challengeOrgLine]|![challengeFilterSlope][challengeFilterSlope]|
|Pass 1|Pass 2|Pass 3|
|![challengeFilter2][challengeFilter1]|![challengeFilter3][challengeFilter2]|![challengeFilter4][challengeFilter3]|
|Pass 4|Output||
|![challengeFilter4][challengeFilter4]|![chanllengeOutput][chanllengeOutput]||

The above steps handle most of the frames of the challenge video.
However, the method highly depends on the video. it may not handle well for other challenge videos.

### 2. Potential shortcoming identification with the current pipeline

1. When there are a lot texture in the image, there will be a lot of false lines detected. It is difficult to filter those lines.
2. When there are different lighting in the road in a single image, not only the light intensity change would be considered as line, the different contrast under different region of light would also make the pipeline difficult to extract lane lines correctly.
3. The current pipeline assumes the lane lines are straight. It cannot detect the curved lane lines which should be common when there is a sharp turn.

### 3. Possible improvement suggestions to the pipeline

1. When processing video, we can consider the result of previous frame to process the current frame. The changes of those lane lines between consecutive frames should be small. The lines from the previous frame can provide a good initial guess for the current frame.
2. Use color segmentation to detect the road area first. We can then apply a region of interest mask to filter out line segments outside the road. Area outside the road is no use to us to detect those land lines. The mask is better than the current mask which is hard-coded.
3. As there may be different lightings on the road, we can try to use adaptive histogram equalization to imporove the contrast of the road area.
