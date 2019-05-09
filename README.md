# cellsOnEdge
Analysis of location and composition of bacterial colony edges in microscopy data

## Introduction

When capturing the dynamics of bacterial colony expansion, it is often useful to use tiles of images to monitor the position and composition of the colony edge at high resolution over long periods of time. For example, the following image is taken from a 2 mm x 0.15 mm x 12 hr dataset, and has been stitched together from 20 adjacent fields of view:

While this approach can provide a combination of excellent spatial resolution and long sampling times, the resulting datasets are typically too large to analyse efficiently by hand. cellsOnEdge is a suite of tools designed to automatically find the position and composition of the colony edge within these datasets.

## Usage

### Part 0: Preprocessing
Begin in Fiji/ImageJ.

1. Brightfield images should be pre-processed to allow easy binarisation of the colony image (i.e. to allow cells and background to be distinguished with a simple threshold). They should also be split into separate frames, with each field of view stored in a separate folder. The scripts ProcessBulkBFSingleFrames63X.ijm and ProcessBulkBFSingleFrames20X.ijm perform these jobs. These two scripts contain settings for processing brightfield data from a Zeiss Observer microscope with 20X and 63X PlanApo objectives, but you will probably need to find parameters that fit your own dataset.

2. Fluorescence images should be split into separate frames, with each field of view stored in a separate folder. Despeckling is also advised. These jobs are performed by the script ProcessBulkFluo.ijm.

### Part 1: Finding the colony edge
Now switch to Matlab.

1. At this point you can use one of two scripts: PlotEdgeCoordsWithLinearToLinearModels.m or PlotEdgeCoordsWithExponentialToLinearModel.m. Both of these will extract the coordinates of the colony edge at all times, but they vary in the model that they fit for plotting:

   - PlotEdgeCoordsWithLinearToLinearModels.m fits a model consisting of two constant expansion rates, one early and one late.
   - PlotEdgeCoordsWithExponentialToLinearModel.m fits a model consisting of an exponentially increasing expansion rate, smoothly transitioning into a constant expansion rate later.
   
2. Edge extraction works through the following steps. Parameters involved at each stage are indicated in bold and in brackets:

   1. Stitch together brightfield tile (**Root**, **imNos**, **imAngle**).
   2. Binaraise resulting image using a global intensity threshold to estimate local coverage by cells (**imageThresh**).
   3. Cut resulting binary image into strips running perpendicular to the direction of motion (**noBins**).
   4. Find the average coverage along each strip.
   5. The edge is found as a point at which the average coverage value increases above a chosen value (**colonyThresh**). If the current timepoint is before the user-defined time of confluence (**confluenceFrame**), the edge is chosen as the *first* instance this crossing happens (moving from the colony exterior to the colony interior). if the current timepoint is after confluence, the edge is chosen as the *final* instance of this crossing).
   6. The median edge position is then propogated to the next timepoint and used as the starting point to find the edge in the next set of strips. The algorithm can only look a fixed distance away from the previous timepoint's edge position (**sampleWindow**).
   
3. Assuming you have set the output to be verbose, you should see plots similar to the following appearing at this point:

   Each red circle indicates the detected position of the colony edge in each image strip, while the connecting lines indicate the estimated profile of the edge. It is best to monitor these plots continuously as the script runs to ensure that the edge detection is working properly. If not, cancel the script and adjust the analysis parameters.

4. The script now saves the edge coordinates as the variables edgeYs and edgeXs in the root directory. The file is called 'ExtractedProfiles.mat'. It also plots timecourses of the edge position and colony expansion rate, along with the fitted model:

### Part 2: Finding the packing fraction
Now that the edge position has been found, the density of the colony at different positions can be found. Run the script PlotEdgePackingFractions.m to do this.

This script will find the density of the colony at two positions: the front (the region just behind the leading edge), and the 'homeland' (the position just behind the edge at the beginning of imaging). In the image below, the front is represented by the green rectangles while the edge is represented by the purple rectangles:

These rectanges indicate the region over which the packing fraction is calculated. Note that the 'front' window undulates to match the shape of the colony edge, preventing the inclusion of regions outside of the colony proper.

Once the script has finished running, plots indicating the density of the front and homeland will be generated:

On the left, the average packing fraction within each region at each timepoint is shown. On the right, the spatial composition of the 'front' region over time is shown.

The data underlying these plots will also be saved as the 'frontPackingFractions' and 'stationaryPackingFractions' variables in the file 'PackingFractions.mat'.

### Part 3: Finding the edge composition
If your colony consists of two separate populations of cells marked with different fluorescent labels, you can also measure the relative number of each cell type using the PlotEdgePackingFractions.m script.

## References

- Reference to my paper when it's finally out!
