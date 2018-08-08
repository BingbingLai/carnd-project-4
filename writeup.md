

[//]: # (Image References)

[img1]: ./writeup-images/undistorted.png "Undistorted"
[img2]: ./writeup-images/undist-2.png "Undistorted"
[img3]: ./writeup-images/color-and-gradient.png "Road Transformed"
[img4]: ./writeup-images/color-gradient-final.png "Road Transformed"
[img5]: ./writeup-images/bird-eye-view.png "Road Transformed"
[img6]: ./writeup-images/histogram.png "Road Transformed"




[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the ["./project/advanced-lane-finding-project.ipynb"](https://github.com/BingbingLai/carnd-project-4/blob/master/projects/advanced-lane-finding-project.ipynb):

- prepared "object points" (`objpoints`) and "image points" (`imgpoints`)
	- `objpoints`: (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. 
	- `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. 
 
- grayscaled the images
- calibrate camera used cv2.calibrateCamera
- undistorted images using cv2.undistort 

![alt text][img1]




### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

used the undistort function described above, and applied to the images below:

![alt text][img2]




#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

used a combination of color and gradient thresholds to generate a binary image
- code is [here](https://github.com/BingbingLai/carnd-project-4/blob/master/projects/advanced-lane-finding-project.ipynb) at section: `"Color and Gradient Threshold"` 

- first tested different color and gradient thresholds, such as, magnitude binary, x sobel binary, y sobel binary etc.
- example outputs below:
![alt text][img3]
- decided to use the following at the end: hls, sobel x and sobel y. Final image below:
- ![alt text][img4]



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

- code in the same [notebook](https://github.com/BingbingLai/carnd-project-4/blob/master/projects/advanced-lane-finding-project.ipynb),section: "Perspective Transform"
- The code for my perspective transform:


```
def bird_eye_view(undist_img, dst, src):
    img_size = undist_img.shape[1::-1]
    M = cv2.getPerspectiveTransform(src, dst)
    Minv = cv2.getPerspectiveTransform(dst, src)
    warped_img = cv2.warpPerspective(undist_img, M, img_size)
    return warped_img, M, Minv
    
def mask_area(undist_img):
    #make copy of img
    copy_undist= undist_img.copy()
    
    #use previous un-disotrted img
    #get undist w and h
    undist_shape = undist_img.shape[1::-1]
    
    #left_top and right_top has same y, same for the left_bottom and right_bottom 
    top_y = 480
    bottom_y = undist_shape[1] #720
    
    #get every corner points also get seperated x and y
    left_down = (200, bottom_y)
    left_down_x, left_down_y = left_down
    
    left_top = (550, top_y)
    left_top_x, left_top_y = left_top 
    
    right_top = (undist_shape[0]-520, top_y)
    right_top_x, right_top_y = right_top
    
    right_down = (undist_shape[0]-150, bottom_y)
    right_down_x, right_down_y = right_down
    
    #draw target area
    color = [255, 0, 0]
    cv2.line(copy_undist, left_down, left_top, color, 4)
    cv2.line(copy_undist, left_down, right_down, color, 4)
    cv2.line(copy_undist, left_top, right_top, color, 4)
    cv2.line(copy_undist, right_top, right_down , color, 4)
    
    #src points
    src = np.float32([[left_down_x, left_down_y],  #left_down 
                      [left_top_x, left_top_y],  #left_top
                      [right_top_x, right_top_y],  #right_top
                      [right_down_x, right_down_y]])#right_down
    
    #dst points
    dst = np.float32([[100, undist_shape[1]], #left_down
                      [100, 0], #left_top
                      [1100, 0], #right_top
                      [1100, undist_shape[1]]]) #right_down
    return copy_undist, src, dst  
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 100, 720        | 
| 550, 480      | 100, 0      |
| 760, 480      | 1100, 0      |
| 1130, 720     | 1100, 720      |


![alt text][img5]



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

code in section "Lane Line Detection"; used the rough steps below:

- find histogram 

![alt text][img6]

- get mid point
- find left peak points and right peak points
- apply nonzero
- find the left good indexes and right good indexes
- fit second polynomial




#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
