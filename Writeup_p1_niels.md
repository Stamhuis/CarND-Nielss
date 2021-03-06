# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image0]: ./Output/result0.jpg "result0"

[image1]: ./Output/result1.jpg "result1"

[image2]: ./Output/result2.jpg "result2"

[image3]: ./Output/result3.jpg "result3"

[image4]: ./Output/result4.jpg "result4"

[image5]: ./Output/result5.jpg "result5"

[image6]: ./Output/result6.jpg "result6"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps. 
1-gray: Convert image to greyscale for canny edge detection
2-detect edges: Apply canny edge detection to see what pixels in the image have a high delta value in brightness difference. In order for an even smoother Canny detection I applied another Gaussian blur besides the one already in Canny itself
3-masked edges only: Select a region of interest by drawing vertices that roughly select only the lane in front of the car up until the horizon
4-hough transform: Through colliding sinus functions in Hough space defined by Rho and Theta(x / y axis resp.), lines can be detected in a simple bitmap image with only pixels, which is the output of the canny edge detection.  
5-Weighted image: Draw these lines on the original image to have clear feedback of what the algorithm sees. 
*-Extra: For the challenge the frame size of the video was adjusted so it could run on the same pipeline. 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by:
The lines defined in the output of the Hough transform function are defined with 4 coördinates, x1, y1 and x2 and y2. 
1 First, based on the slope a comparrison and separation takes place: positive and above 0.5 but under 0.8 are left lanes. The negative slopes in the same negative range are right lanes. 
Slopes of 0 or a little more / less are noise and not needed. Only slopes closely 0.5-0.8 are interesting becasue they have the right angle of a lane line. 
2 Second, the numpy polyfit function is used to fit a single line on the averages of the slopes for left and right.
3 Thirdly, Extrapolation. Any point y on a line can be found with the function y = m*x + c, where m equals slope value and c a constant to position the line in the axis.
Using this function the lines can be extrapolated on the bottom side of the frame in case a dotted line occurs in the image/video. 
Extrapolation happens by first finding the max x value based on the y max value of the frame, in this case 540 pixels of height (top left is 0,0 x,y)
This x,y value is then taken as one of the two points on basis of which a line is drawn. Same goes for right. 
4 Both lines are then plotted on the original image so a clear feedback results to the designer to finetune the code pipeline. 


These are the steps taken to plot the line detection on the existing image.

![alt text][image0]

![alt text][image1]

![alt text][image2]

![alt text][image3]

![alt text][image4]

![alt text][image5]

![alt text][image6]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when an unclear image with a lot of shadows / light is used as input for the pipeline.
In this case the canny edge detects many different lines and the hough transform isn't able to detect a useful selection of lines, resulting in an empty array of line coördinates, causing the draw lines function to fail.

Another shortcoming could be that the pipeline is currently aimed for 960x540 resolution images. The input video can of course be resized but a proper pipeline would
scale all the different variables based on a broad set of roads that a vehicle could drive on.


### 3. Suggest possible improvements to your pipeline

To make the pipeline more stable I would go for a frame by frame comparisson of the array of slopes and its values. In case of a "difficult" image, meaning many edges to detect due to bad / overlighting scenery,
The hough transform can have a hard time detecting the lane lines correctly because of the amount of noise. If the conditions for the slopes are taken too thight, no lines are generated by Hough transform, resulting in empty arrays.
Lowering the hough transfrom conditions with minimal line lengts and maximal gaps would result in lower quality of hough transformation of "easy" images. 
In case of an empty array, the value of the previous frame could be used and averaged with the 2 frames before to maintain the average until a next frame shows up that has less shadows / high differences / obstacles on the road, 
resulting in an array that does have values, making the function work again. Hereby the pipeline becomes more stable and not "stumble" over tricky frames. 

Make smarter use of the histogram plot of the canny edge detection. If a certain range of interest is known for an image, the focus could be shifted. Lets call it a dynamic canny parameter function. 

To tackle the resolution issue, I would propose to use relative values for the vertices and line extrapolation, working with percentages or total height / width. 
