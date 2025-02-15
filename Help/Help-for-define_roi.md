# Help for the function [<i>define_roi</i>](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Tractography-Functions/define_roi.m), v. 1.1.0

## Introduction

This help file contains information about
1) [Purpose of the Program](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md#1-purpose)
2) [Usage of the Program](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md#2-usage)
3) [Automated Segmentation Algorithm](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md#3-Automated-Segmentation-Algorithm)
4) [Syntax](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md#4-Syntax)
5) [Example Code](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md#5-Example-Code)
6) [Acknowledgements](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md#6-Acknowledgements)


## 1. Purpose

The function <i>define_roi</i> is used to digitize the aponeurosis of muscle fiber insertion. The digitized points are used to reconstruct a mesh; the mesh is used as the seed surface for fiber tracking.  

## 2. Usage
The mesh is a required input to [<i>fiber_track</i>](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Tractography-Functions/fiber_track.md) and [<i>fiber_quantifier</i>](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-fiber_quantifier.md); it may be visualized using [<i>fiber_visualizer</i>](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-fiber_visualizer.md). The required inputs are the anatomical image, the muscle mask, and a structure define the user's options for defining the mesh; an optional structure allows plotting of the results.  The output is the mesh reconstruction of the aponeurosis. There are two options for defining the aponeurosis:
1) <i>Manual selection</i>: A single figure window, containing three panels, is displayed. The center panel shows the current slice, the left-hand panel shows the preceding slice, and the right-hand panel shows the upcoming slice. An interactive tool is opened that allows the user to adjust the center panel's window and level settings. In the center panel, the edge locations of the mask are indicated. Text prompts in the command window guide the user through the following steps.  First, the user zooms the image to the area of interest, selecting Enter when finished. Then the user defines the aponeurosis with a series of left mouse clicks. The selected points can form a line or close to form a polygon. A right mouse click is used to complete the definition. At each slice, the user is given the option of repeating the procedure in case of error.

2) <i>Automatic selection</i>: A single figure window, containing three panels, is displayed; each panel shows the current slice. The algorithm described in Section 3 is used to present an initial estimate of the aponeurosis’s location. This estimate is presented as magenta points in the left-hand panel. In the center panel, the edge locations of the mask are indicated and the initial segmentation results are shown as semi-transparent red pixels. In the right-hand panel, the smoothed edge pixels are shown for the segmented region (magenta points) and the dilated region (cyan points). The dilation process is discussed below.

   The user interacts with the center panel. The user can correct erroneous assignments in the initial estimate by using the left mouse button to select voxels for removal from the initial assignment and the right mouse button to select voxels to add to the initial assignment. The user selects Enter to proceed. In subsequent slices, the finalized assignment from the preceding slice and the results of an edge detection algorithm are incorporated into the initial segmentation estimate and the process is repeated. The figure below illustrates this selection process.

   <img src="https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Images/FigureS2.png" 
alt="Figure S2" width="960" height="334" border="5" />
   <b><i>Example window from <i>define_roi</i></b>, showing (from left) the previous slice; an overlay of the selected pixels in the current slice; the smoothed points (purple) and the smoothed, dilated points (cyan).</i>

3) <i>From mask</i>: This is a special instance, applicable when the user has already defined the mesh and now wishes to redefine it, either with a different number of rows and columns or with a dilation step.

For the manual and automated selection processes, the user is given the option of repeating erroneous slices; screen prompts direct this process

In voxels containing partial-volume effects (two types of tissues contained in a single voxel), the diffusion tensor data may be erroneous. This may lead to errors in the estimation of the fiber direction. One way to avoid this at the muscle-aponeurosis interface is to define the points just outside the aponeurosis. Alternatively, the user may dilate the mesh (that is, widen it within the imaging slice). This option is specified in the input structure <i>dr_options</i>.

After the points have been selected, the mesh is initially formed with the dimensions set in dr_options.  To smooth the mesh, it is then down-sampled by a factor of four. Finally, the smoothed mesh is interpolated at the desired resolution. 

A file called <i>roi_mesh_file.mat</i> is automatically saved in the working directory. The user is advised to rename this file promptly. 

The mesh may be viewed using [<i>fiber_visualizer</i>](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-fiber_visualizer.md), either as part of the function call to <i>define_roi</i> or directly from the command line.

[Back to the top](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md)

## 3. Automated Segmentation Algorithm
The algorithm is illustrated in the figure below. In the first slice analyzed, the muscle mask is eroded (boundary pixels are removed) and then multiplied by the anatomical image slice. This resulting image includes only the pixels inside of the muscle of interest. Then, a k-means clustering algorithm is used to segment the remaining image data into three clusters.  The cluster corresponding to the highest signal intensities is assumed to represent the muscle.  The other clusters are presented to the user as the initial estimate of the aponeurosis’s location. Also, a Sobel edge detection is performed within the masked region of the image and used to estimate the aponeurosis location.  These images are combined using the weights [1, 1] for the k-means and edge regions, respectively; voxels with sums of 2 are included in the initial estimate. A Savitsky-Golay filter is applied to the pixel positions and used to form a smoothed curve that indicates the location of the roi_mesh points for that slice.  The user can correct pixel locations, as described above; the <i>roi_mesh</i> points are automatically updated.

<img src="https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Images/FigureS3.png" 
alt="Figure S3" width="480" height="728" border="5" />

<b><i>Flowchart of automated segmentation algorithm in define_roi</i></b>

In subsequent slices, the preceding aponeurosis segmentation is incorporated into the initial estimate.  These images are combined using the weights [1, 1, 1.5] for the k-means, edge, and preceding regions, respectively; voxels with sums greater than 2 are included in the initial estimate. A Savitsky-Golay filter is applied to the pixel positions and used to form a smoothed curve that indicates the location of the roi_mesh points for that slice.  The user can correct pixel locations, as described above; the roi_mesh points are automatically updated.

[Back to the top](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md)

## 4. Syntax
The function define_roi is called using the following syntax:

[roi_mesh, roi_mask, roi_mesh_dilated]=define_roi(anat_image, mask, dr_options, fv_options);

The input arguments are:

* <i>anat_image</i>: The imaging data. 

* <i>mask</i>: The mask, as defined by the function [<i>define_muscle</i>](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_muscle.md) or another source.

* <i>dr_options</i>: A structure containing the following fields:

    <i>slices</i>: A two-element vector containing the first and last slices that the user wishes to digitize.
  
    <i>dti_size</i>: A three-element vector containing the numbers of rows (N<sub>R,D</sub>), columns (N<sub>C,D</sub>), and slices (N<sub>S,D</sub>) in the DTI dataset, input as [N<sub>R,D</sub> N<sub>C,D</sub> N<sub>S,D</sub>]
  
    <i>mesh_size</i>: A two-element vector containing the numbers of rows (N<sub>R,A</sub>) and columns (N<sub>C,A</sub>) desired in the output mesh.
  
    <i>n_steps</i>(optional): The number of dilation steps.
  
    <i>method</i>: a string variable set either to 'manual', 'auto', or 'mask'. 

* <i>fv_options</i>: Optional. If specified, this calls the [<i>fiber_visualizer</i>](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-fiber_visualizer.md) function to plot the mask and roi mesh.

The output arguments are:
* <i>roi_mesh</i>: A 3D matrix containing the reconstructed mesh with size n_row x n_col x 6. In the 3rd dimension, levels 1-3 hold {row column slice} coordinates and levels 4-6 hold the {row column slice} components of the normal vector to the mesh surface at the point {row column slice}.
* <i>roi_mask</i>: A 3D matrix containing a mask indicating the location of the aponeurosis. This can be useful as a fiber-tracking stop criterion or to account for the volume of non-contractile tissue. It can also be used to re-form the mesh at different density or following dilation.
* <i>roi_mesh_dilated</i>: A dilated version of the <i>roi_mesh</i>, with the number of dilation steps set in the input argument dr_options.n_steps. 

[Back to the top](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md)

## 5. Example Code

### Example 1
Given 

1.	An anatomical image with variable name anat_image and having matrix size 192x192x44, field of view 192x192 mm, and slice thickness 7 mm;

2.	The muscle mask, stored in a variable called mask; and

3.	DTI images, having matrix size 192x192x44, field of view 192x192 mm, and slice thickness 7 mm

the code below will allow the user to:

1.	Manually select aponeurosis in slices 4-31;

2.	Create a mesh of size 150 rows x 30 columns; and

3.	Visualize the outcome, using slices 14, 24, 34, and 44 of the anatomical image stack for reference.

% Set mesh options:

dr_options.slices = [4 31]; %analyze slices 4-31

dr_options.dti_size = [192 192 44]; %matrix size and # of slices in DTI images

dr_options.mesh_size = [150 30]; %mesh will have 150 rows and 30 columns

dr_options.method = ‘manual'; %digitize it manually

% Set visualization options

fv_options.anat_dims = [192 7]; %FOV and slice thickness of the images to be displayed, in mm

fv_options.anat_slices = 14:10:44; %display slices 14, 24, 34, and 44 

fv_options.plot_mesh = 1; %do plot the aponeurosis mesh

fv_options.plot_mask = 0; %don’t plot the mask

fv_options.plot_fibers = 0; %don’t plot any fiber tracts

fv_options.mesh_size = [192 192]; %rows x columns of the images used to generate the mesh

fv_options.mesh_dims = [192 7]; %FOV and ST of the images used to create the mesh

fv_options.mesh_color = [0.75 0.75 0.75]; %make the mesh light gray

% call the function:

roi_mesh = define_roi(anat_image, mask, dr_options, plot_options);
 
### Example 2
Example 2 matches Example 1, except that the mesh is automatically segmented, the ROI mask is returned, and the mesh is dilated.

% Set mesh options:

dr_options.slices = [4 31]; 	%analyze slices 4-31

dr_options.dti_size = [192 192 44]; %matrix size and # of slices in DTI images

dr_options.mesh_size = [150 30]; %mesh will have 150 rows and 30 columns

dr_options.method = ‘auto'; %automatic definition

dr_options.dilate_mesh = 'y'; %dilate the mesh within the slice plane

dr_options.n_steps = 1; %dilate it once

% Set visualization options

fv_options.anat_dims = [192 7]; %FOV and slice thickness of the images to be displayed, in mm

fv_options.anat_slices = 14:10:44; %display slices 14, 24, 34, and 44 

fv_options.plot_mesh = 1; %do plot the aponeurosis mesh

fv_options.plot_mask = 0; %don’t plot the mask

fv_options.plot_fibers = 0; %don’t plot any fiber tracts

fv_options.mesh_size = [192 192]; %rows x columns of the images used to generate the mesh

fv_options.mesh_dims = [192 7]; %FOV and dZ of the images used to create the mesh

fv_options.mesh_color = [0.75 0.75 0.75]; %make the mesh light gray

% call the function:

[roi_mesh, roi_mask, roi_mesh_dilated]=define_roi(anat_image, mask, dr_options, fv_options);

[Back to the top](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md)

## 6. Acknowledgements
People: Zhaohua Ding

Grants: NIH/NIAMS R01 AR050101, NIH/NIAMS R01 AR073831

[Back to the top](https://github.com/bdamon/MuscleDTI_Toolbox/blob/master/Help/Help-for-define_roi.md)
