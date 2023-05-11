# 3D_Terrain_Mapping-Computer_Vision


## Introduction

Early implementation of landscape mapping was all done manually, where people would go to the area measure change in elevations and use concentric circle to represent elevation or depression. With the progress in modern day technology and development of UAV's have made this task much easier. These UAV's can be flown remotely and can hold attachments which makes recording data much faster and efficient. LiDAR and Cameras are the two main attachments for remote aerial landscape mapping. With the recent development of the LiDAR technology, it is a great tool for mapping. LiDAR would be a better option for small distances, but as the distance increases, the LiDAR accuracy decreases. There are sensors that can operate with greater distances, but the cost to purchase them is also extremely high[1]. This is where using commercial grade cameras can help, they do not provide the ease of use like LiDAR but can help cut costs. Cameras can produce amazing results with methods like structure from motion[2], photogrammetry[3]. There are two main approaches when using camera systems monocular and stereo techniques. These techniques are used for different purposes. A monocular approach is very effective when a specific location, like the Colosseum, is the target. It's like constructing a sphere around the landmark and taking pictures at each specific interval on the sphere, which would eventually give us a view from every perspective. On the other hand, stereo approach is very effective when more ground has to be covered like in landscape mapping. The main reason monocular approach is inefficient for landscape mapping is the fact how the point cloud is generated. Let's take the example of colosseum[4] there would thousands of images across which there be hundreds of thousands of keypoints. Basis for monocular approach is triangulation where image pairs are formed and each keypoint matches from each pair is mapped into 3D space interconnecting like a dome around the monument. Only the keypoints can be triangulated but one image is hardly enough to cover the massive monument this is where keypoints around thousands of images merge and make one 3D model out of it. Using the same approach for landscape mapping would result in extremely lowdensity point cloud as only keypoint matches would be plotted. To put it into hard numbers let's say a video is recorded at 60 fps with a field of view of 600 and each frame is 100 x 100 which is 10000 pixels and let's assume that first 10 frames captured a small common region which would be 100000 pixels total out of which hardly 50000 keypoints would be detected at max under ideal condition. From these 50000 points we perform matching after which these points would be plotted hypothetically assuming we lost 10000 points which would only plot 40000 points and remaining data would be lost. Alternatively, stereo approach uses correspondence points which are same points in both the images. This method takes into account every pixel and plots it. However, due to stereo concept there is small part on left side of left image and right side of right image which is not present in the other half. This information is lost but, when we use video, this can be compensated by the following pairs this giving us a really dense output. We have further discussed in the report how to achieve this.

## Methodology

We used data from a variety of sources for our study, including Google Earth, phone cameras, and simulated environment in Unity. To begin with, the process can be divided into 4 main steps, here is how the pipeline looks:

![](RackMultipart20230511-1-mudsjg_html_aeb75c19ad1fe6f3.jpg)

### Camera Calibration

we must calibrate our camera, which can be done by taking numerous pictures of a chessboard (printed on paper for our work, we have used a 9x6 size) from different angles. It is crucial for this task that the chessboard be steadily mounted, ideally on a wall or on a flat surface, as any warp would add inaccuracy to image and would affect the calibration parameters. A sharp image might produce superior outcomes, thus it is also crucial to keep the area well-lit. The minimum number of photos needed for calibration is 50, but this number can vary. With the help of these photographs, we can begin to calculate the locations of the square's vertices, which are known with a fair amount of precision and can be used to compare with the images, looking for variations within each image, and provide a few key parameters that will be useful in the following steps. Two main parameters from these are camera intrinsic matrix and the distortion coefficient which are as follows:

 ![Shape1](RackMultipart20230511-1-mudsjg_html_6da551c152b94315.gif)

This matrix gives us focal length of the camera[5], [6].

This list gives us the distortion due to lens[5], [6].

These parameters can be used to correct the images, these images might look perfect but when the image is captured it is warped which is then corrected by internal software which is not perfect. The warp can be due to lens imperfections which might lead to barrel or pin cushion images which can be corrected by using this method. We apply same transformation to our input stereo images. Below showcased an example of the scenario.

![](RackMultipart20230511-1-mudsjg_html_dee3967e59163c0a.jpg)

The slight warps in the right image are due to corrections we made by our code. These changes are majorly notable near the vertices. We have a stereo camera setup so in order to calibrate them we need to take the images from both cameras at same time and use them to individually calibrate each camera. Final step we need to correct our input stereo images.

### Calculating Disparity

Our goal here is to extract depth information from these stereo images in order to achieve this we will use the corresponding pixels in each image and find the disparity. For example, try this keep your palm in front of your face and try to keep some easily notable object exactly behind that palm when you close one of your eyes the notable object might slightly or completely disappear. But when you open your eye again you start noticing that you could see behind your hand slightly that is depth perception. Disparity can be calculated with the relation that the distance between the points in the scene and the camera are inversely proportional to each pixel value of a disparity map

![](RackMultipart20230511-1-mudsjg_html_57f49434732daf07.jpg)

Few key information about the setup is as follows:

- Baseline – distance between the 2 cameras.
- Image – images are placed at Xi and Xr in the image below.
- Optical center - is the actual center of the camera lens Oi and Or. •Corresponding points - Pi and Pr are same points in both images.
- Epipolar Lines – Which is baseline this is fixed and will not changes with corresponding points.

Epipolar lines form the basis for this operation these lines are fixed for every pair of images and does not change while iterating over corresponding points, because if these lines changed there would be no frame of reference to plot the disparity map. The general formula for calculating disparity is given by:

![](RackMultipart20230511-1-mudsjg_html_1acf26b36c2b43cf.jpg)

The third half of the equation B/f \*z will be used later to convert disparity into point cloud. To implement this, we have to used the OpenCV library[5], [6] from which a function called stereoSGBM is used to calculate disparity. This function has 11 parameters which are as follows "mindisparity" which is 0 by default but the algorithm can shift the image to right and this value can be used to tweak it. The 2 main parameters for this function are "numdisparity" and "block size" they must be divisible by 16 and ideally always odd number respectively. "P1" and "P2" these 2 parameters control the smoothness of disparity and their values are usually 8 and 32 respectively multiplied by number of images \* block size2. "disp12maxdiff" helps us set the maximum difference. "preFilterCap" it sets the range to [- value, + value], "uniquenessRatio" threshold to determine if first match is better then second. "speckleWindowSize" and "speckleRange" are used to control the smoothness and small speckles which are the gradient blobs respectively. Finally, "Mode" this can be set to various to methods to control the disparity. These parameters can be extremely challenging to tweak. Here are some images to show results.

![](RackMultipart20230511-1-mudsjg_html_6b454bc54ca2511f.png)

As we can see the disparity doesn't really look like the real image ideal case scenario would be a very smooth gradient finish which would show lighter areas are higher elevation and darker areas are lower elevation.

### Stitch Disparity

At this point we have disparity maps, we have to link first pair of stereos to second pair and chain for the number of pairs. Essentially, as the end product we have to stitch the disparity maps. Let's examine the difficulties. The first would be a typical case scenario with altitude data and camera position provided, however given that how the data was gathered, it was difficult to extract this information with reasonable accuracy. The second thought was that there should be a mathematical approach for determining the location and that there should be a way to apply the same reasoning from camera calibration to this situation. Although the orientation of the square's vertices can be used to construct some sort of frame transformation logic, the difficulty would still be present since in a random image it would be nearly impossible to identify a separate object in each pair and significantly more difficult to obtain its frame. Alternatively, we may use keypoints and match them to acquire the best anchor points instead of hunting for objects, these points will be the same in in both images and will be detected with reasonable accuracy in both images. The idea was if we can construct some geometry from these points, we could triangulate back to the camera even though we have a stereo setup the two images should point at the center of baseline. Let's say, we take 3 matched points from each image then form a triangle and try to project such that 3 lines would somehow intersect but this is an infinite solution problem. We could always change the data and get camera parameters to calculate the extrinsic matrix.

We decided to use the current data and experimented with image stitching algorithm. We had previous experience with this algorithm and knew it could jigsaw puzzle its way and stitch them could the same be done for disparity maps. Technically no, with the results we have it would not detect any keypoints. To work around this issue, we used the left image as basis to calculate exact stitching location. The way we achieved this was by writing a custom code that takes in a list of images and their corresponding list of disparity maps. A note, the size of disparity map is same as images. We first find the images that share common area and calculate stitching pairs. This is done by first calculating SIFT(scale invariant features)[7] keypoints for a pair of images then matching them to check if they match above a threshold, it would be marked as a match. To store these matches a 2D matrix was created of size no. of image x no. of images. Where for each match lets assume for image 1 it would be row 1 and let's say 3rd image was a match then it would highlight (1,3) position as 1. Which would show row number = image number and column number in that row marked 1 would be a match. Now that we have all the pairs marked in the matrix, we can start stitching by taking 2 images using those matched keypoints to find homography. Which is a combination of rotation and translation matrix to convert from one view to another that is how to change one image with respect to another to blend them. We can use the size of the images to create 2 imaginary frames and use the homography matrix on one to align it with other then concatenate them to find the starting location of second image that is the overlap. This works because we are not only rotating the frame but also translating it. We can now use the image to transform its perspective and just paste it in place of imaginary frames. While we do this can do the exact same for disparity maps and since the operation is just copying it does not increase the overall time complexity of the code. As a result, we have stitched our disparity showcased below is one of our outputs. On the left we have the stitched disparity and on the right, we have the original stitched image. These were created with 20 images each.

![Shape2](RackMultipart20230511-1-mudsjg_html_a34b6baeedbb425c.gif)

The stitched output turned out pretty good without major errors. Let's move onto the final step and convert it to point cloud to verify our stitching results.

### Plot point cloud

Before we plot these outputs, we need to convert it to have 3 points. To do this we will use the second part of our equation that we used when calculating disparity **B / f \* z = disparity.** Where z represents depth of point in z direction. We will use OpenCV function to 'reproject image to 3D' Which takes in the disparity map and Q which is the projection matrix given as[8].

![](RackMultipart20230511-1-mudsjg_html_5bd975af1f8abb0b.jpg)

The focal length that we calculated from our above image can be used here to calculate the depth. The output from the function is a list of 3D points. Further, to remove the redundant space from the disparity as left has a small part which does not exist and right the disparity is set to the lowest value so creating mask by comparing each point in disparity map with lowest value in the map gives us a true / false matrix which can be used to mask the point cloud and retrieve colors. Finally, we have all the pieces to plot the point cloud we will be using a library called open3d[9] which has great support and makes it really easy to plot. The function takes in a list of 3d points and colors which are to be normalized by 255. Let's look at some of the results.

![](RackMultipart20230511-1-mudsjg_html_c402d36ddf2b70ab.jpg)The first image is the original image and the 2 images below are the point cloud. These point clouds are a result of only 1 stereo pair[10].

![Shape3](RackMultipart20230511-1-mudsjg_html_a40693b268d8d277.gif)

Let's check the output of our stitched image left side is the real image and right side is the point cloud. Stitched disparity for the images below is showcased in last part of stitch disparity topic.

![Shape4](RackMultipart20230511-1-mudsjg_html_7d38d76e70e73e28.gif)

## Results and Conclusion

Point cloud can usually be first visually inspected given the look of point cloud we can conclude that the points are not really showing up and has a lot of space to improve. The major cause lies with the disparity calculation given 11 parameters which is extremely challenging to tweak and get a reasonable disparity. If we use some hard numbers to compare, we can use the mask to count how many pixels were assigned lowest value which is no disparity. We want maximum number of pixels to have disparity.

| images | True (pixel has disparity) | False (pixel has no disparity) | Total (pixels) |
| --- | --- | --- | --- |
| Point cloud with 1 pair | 1084933 | 226391 | 1311324 |
| Stitched point cloud | 179146 | 20854 | 200000 |

As we can see from the results above, we have a lot of pixels under the false category these pixels are not plotted in the point cloud. Another inspection is visually detecting the disparity gradient. Highest points should be close to white while the lower areas should be close to black/gray colors. When it comes to stitched point cloud, we can visually see that the blend is almost perfect the minor issues are due to human error as these images were captured using a phone. But with a better setup those minor inconsistencies can be eliminated.

To conclude our tests, we can say that the image stitching algorithm is a success and the stitched point cloud turned out to be as expected matching the images. This method could be possibly used as an alternative to the general structure from motion method when there is no way of determining camera location.

## Future Work

We have performed one more experiment which was not mentioned in the above part, the section where we discussed the stitched disparity the output shown there was taken from a phone it was recorded as a monocular video which was then split into images and every alternate image was skipped. From the remaining images alternate images were assigned to left and right stereo folders which were then used as stereo images. The result is as shown in the above point clouds.

Some points that can be further tested and implemented:

- Find better and more consistent method to calculate disparity map, a machine learning approach might be better (MIDAS).
- Implement with known camera location and compare the outputs.
- Use tested data as ground truth to verify and scale the output to real world measurements.

## References

1. C. Serifoglu Yilmaz and O. Gungor, "Comparison of the performances of ground filtering algorithms and DTM generation from a UAV-based point cloud," _Geocarto Int_, vol. 33, no. 5, pp. 522–537, May 2018, doi: 10.1080/10106049.2016.1265599.
2. M. J. Westoby, J. Brasington, N. F. Glasser, M. J. Hambrey, and J. M. Reynolds,

"'Structure-from-Motion' photogrammetry: A low-cost, effective tool for geoscience applications," _Geomorphology_, vol. 179, pp. 300–314, 2012, doi:

https://doi.org/10.1016/j.geomorph.2012.08.021.

1. S. I. Jiménez-Jiménez, W. Ojeda-Bustamante, M. D. J. Marcial-Pablo, and J. Enciso,

"Digital terrain models generated with low-cost UAV photogrammetry: Methodology and accuracy," _ISPRS Int J Geoinf_, vol. 10, no. 5, May 2021, doi: 10.3390/ijgi10050285.

1. S. Agarwal, N. Snavely, I. Simon, S. M. Seitz, and R. Szeliski, "Building Rome in a day," in _2009 IEEE 12th International Conference on Computer Vision_, 2009, pp. 72–79. doi:

10.1109/ICCV.2009.5459148.

1. Itseez, "Open Source Computer Vision Library." 2015.
2. G. Bradski, "The OpenCV Library," _Dr. Dobb's Journal of Software Tools_, Jan. 2008.
3. T. Lindeberg, "Scale Invariant Feature Transform," in _Scholarpedia_, vol. 7, 2012. doi:

10.4249/scholarpedia.10491.

1. Prof. Didier Stricker, "3D Computer Vision ," _https://ags.cs.uni-kl.de/_.
2. Q.-Y. Zhou, J. Park, and V. Koltun, "Open3D: A Modern Library for 3D Data

Processing," Jan. 2018, [Online]. Available: http://arxiv.org/abs/1801.09847

1. Google, "Google Earth Grand Canyon," _earth.google.com_, Jun. 12, 2012.
