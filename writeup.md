#**Finding Lane Lines on the Road** 

##Writeup

---

**Finding Lane Lines on the Road**


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

The goal of the project is to isolate the two lines that signal the driving lane.
In this process we need to treat the rest of the image as noise that needs to be cleaned

My pipeline consisted of the next steps:

1. Input image is converted to grayscale.
    + This first step is essential since we narrow down the color selection to 255 and finding the white lanes is easier. 
2. Two color masks are computed and combined, for white and yellow color.
    + The mask for the grayscale image is computer, using the gray tones ranging from 200 to 255.
    + The yellow line though is neither white nor black. We convert the input image to hsv and use a google search to 
      reveal ranges of hsv-converted yellow color. It turns out that [h, s, v] has to be between [20, 100, 100] and 
      [30, 255, 255]. Using this range we compute the yellow mask.
3. The color masks are applied to clear all colors outside the selected color ranges.
    + The two masks are combined using bitwise or operation and applied to the original image using a bitwise "and"
      operation.
4. Gaussian smoothing is applied.
5. Canny edge detection is applied to reveal edges.
6. Calculation of a four sided polygon to be used as a mask to define the region of interest.
   + For the images, the task was straightforward, but a lot of experimentation was needed in order to find good values
     for all the video cases.
7. Hough transform is applied to convert the points in the image to lines, using the region of interest mask. 
   + Experimenting with the parameters was necessary.
8. The lines are averaged and extended to their extreme points to output two single lines for left and right. 
   + The y axis is inverted so we can work in cartesian plane. For each frame we compute an average slope, x0 and y0 
     values, using all lines computed. Lines having slope around 0 are excluded from the average since they introduce
     corner cases (divisions by zero) and crashes. They also have nothing to offer as means of classifying left or right
     lines.
   + The lines are classified to belong to the left or right lines (for positive slopes: left lines). The image is 
     also split vertically in two halves and this information is also used to improver the classification of the lines.
   + [x0, y0] belong to the "current" averaged line. We solve y0=slope*x0+b regarding b, and compute the b value.
   + For the region of interest, we definitely know the upper and lower y values, so simply we do x=(y-b)/slope
     to computer upper and lower x.
9. The averaged lines are smoothed based on their endpoints using moving average.
   + We have computed an averaged line, but as the video runs, the line jitters a lot. A running average, takes into 
     account the endpoints of each line computed per frame. The jittering is improved a lot.
   + Special attention has been given to the calculation of proper initial conditions for the running average. 
10. The lines are drawn on top of the original color image (annotation).
11. The results are saved to relevant files.


###2. Identify potential shortcomings with your current pipeline

1. One potential shortcoming would be what would happen in the case the road is full of closed turns. A global calculation 
for the region of interest mask would be very difficult.  
2. Another shortcoming could be the case we would need longer annotation lines to define the left and right lines (to give
more time to stop the car for example in case of accidents). Changing the parameters computing the roi mask leads 
sometimes to crashes (for example switching the line `upper_boundary = image_shape[0]/2 + image_shape[0]/8.5` to 
`upper_boundary = image_shape[0]/2 + image_shape[0]/9`).
3. Light conditions could play significant role to the output of the white and yellow masks (for example a rainy day).

###3. Suggest possible improvements to your pipeline

1. A possible improvement would be to average the left and right line sets using weighted linear regression. Despite the 
fact that masking eliminates most of noise, still there are frames that include unwanted lines, still *in* the roi.
These situations produce unwanted effects (more jitter, faulty line recognition). Those lines far from the average
could have low weight in the regression algorithm.