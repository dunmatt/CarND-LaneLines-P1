#**Finding Lane Lines on the Road** 

##Writeup Template

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline was a bit of a departure from the provided helper functions.  I skipped grayscale entirely, did a small guassian blur, then canny edge detection, and then used masks to split the image into left and right regions of interest.  For each region I then computed the average of the slopes and offsets of the Hough lines and drew those onto the image.  Works like a charm.

The decision to split the image in half has certain ramifications though.  It assumes that the camera is pointed straight out the front of the vehicle, and that the vehicle counts as being in the lane in which the camera is.  For this project those were fine assumptions.  In a more realistic senario they'd likely not hold, but to correct for camera position and angle is just a matter of panning and spinning the image until it meets the preconditions for the pipeline (or better yet, fixing the parameterization of the pipeline so that it could be the thing that rotates and pans rather than the image).

Anyway, splitting the image is a very simple grouping step that cleanly separates the various Hough lines into groups that can be averaged to find the lane markers.  Once I had the average line in each lane marker ROI I drew the lines over the image.


###2. Identify potential shortcomings with your current pipeline

My pipeline is pretty noisy; the lanes jitter between frames as they extend into the distance.

Another huge shortcomming is that I'm using a linear model for the lane markers, so it can't represent curves.  All of the test code is highway driving which means the curves are gentle enough for it to not matter too much, but it definitely wouldn't generalize to off-highway.

The parameters I'm using don't do a very good job seeing far-distant lane markings.

The parameters I'm using are constants, they work well in this case and likely no others.  No constant will give good performance across the board, and thus I shouldn't be using them.

And finally, my current pipeline is in no sense Bayesian; knowing where the lanes were a split second ago tells us a great deal about where they are now, but as is I'm simply ignoring that information.


###3. Suggest possible improvements to your pipeline

Potential improvements depend on how much computational power is available.

At a bare minimum I should put add a low pass filter after the average line computation to eliminate the noise-jitter.

A good next step beyond that would be to choose the Canny and Hough parameters dynamically based on location in the image.  The idea is that towards the center of the image things will be further away and thus smaller and noisier, and thus good parameters to find markings there will be different from good parameters to find markings nearby (at the bottom and towards the sides of the image).  To do that I'd have to re-implement the canny and hough algorithms to accept functions in place of constant parameters, and then I'd need a hefty dataset to discover what those functions should be, but those are both doable things.

Beyond that, assuming I had access to enough data and precomputation time, I'd add a color correction step.  It would work by first classifying the image by lighting condition (eg, clear sun, cloudy, sodium-light lit tunnel, golden hour, etc) and then choose filter parameters that work well under those conditions.  This requires a lot of data, because it adds several parameters (at least brightness, contrast, and color temperature) to each of the parameter-functions being sent to canny and hough, which adds orders of magnitude to the complexity of the problem of figuring out what those parameter-functions should be.

And finally, if I could have however much processing power I want I'd throw away Hough lines entirely and replace them with Krigged lines (trained directly on the Canny edges in the ROI).  That way I could model arbitrarily curved lanes and use the vehicle speed as a prior on the permitted curviness.  The trouble here is that Krigging involves matrix inversion (which is O(N^3)), so I'd need plenty of CPU.





