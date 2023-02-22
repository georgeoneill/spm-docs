# Using Dartel <span id="Chap:dartelguide" label="Chap:dartelguide"></span>

Dartel[^1] is a suite of tools for achieving more accurate inter-subject
registration of brain images. It consists of several thousand lines of
code. Because it would be a shame if this effort was wasted, this guide
was written to help encourage its widespread use. Experience at the FIL
would suggest that it offers very definite improvements for VBM studies
-- both in terms of localisation[^2] and increased sensitivity[^3].

## Using Dartel for VBM <span id="Sec:dartel_vbm" label="Sec:dartel_vbm"></span>

The following procedures could be specified one at a time, but it is
easier to use the batching system. The sequence of jobs (use the *Batch*
button to start the batch manager) would be:

- **Module List**

  - **SPM$\rightarrow$Spatial$\rightarrow$Segment**: To generate the
    roughly (via a rigid-body) aligned grey and white matter images of
    the subjects.

  - **SPM$\rightarrow$Tools$\rightarrow$Dartel Tools$\rightarrow$Run
    Dartel (create Template)**: Determine the nonlinear deformations for
    warping all the grey and white matter images so that they match each
    other.

  - **SPM$\rightarrow$Tools$\rightarrow$Dartel
    Tools$\rightarrow$Normalise to MNI Space**: Actually generate the
    smoothed "modulated" warped grey and white matter images.

Further details of the steps are described next.

### Using Spatial$\rightarrow$Segment

*Note: This subsection will be elaborated on later.*

The first step is to classify T1-weighted scans[^4] of a number of
subjects into different tissue types via the Segmentation routine in SPM
(Ashburner and Friston 2005), which can be found under
SPM$\rightarrow$Spatial$\rightarrow$Segment. With this option, the
"imported" tissue class images (usually rc1.nii and rc2.nii) would be
generated directly. It is also suggested that *Native Space* versions of
the tissues in which you are interested are also generated. For VBM,
these are usually the c1\*.nii files, as it is these images that will
eventually be warped to MNI space. Both the imported and native tissue
class image sets can be specified via the Native Space options of the
user interface.

Segmentation can require quite a lot of memory, so if you have large
images (typically greater than about $256\times256\times150$) and trying
to run it on a 32 bit computer or have relatively little memory
installed, then it may throw up an out of memory error.

### Using Dartel Tools$\rightarrow$Run Dartel (create Template)

The output of the previous step(s) are a series of rigidly aligned
tissue class images (grey matter is typically encoded by rc1\*.nii and
white matter by rc2\*.nii -- see Fig
<a href="#Fig:VBM" data-reference-type="ref"
data-reference="Fig:VBM">1.2</a>). The headers of these files encode two
affine transform matrices, so the Dartel tools are still able to relate
their orientations to those of the original T1-weighted images. The next
step is to estimate the nonlinear deformations that best align them all
together (Ashburner 2007). This is achieved by alternating between
building a template, and registering the tissue class images with the
template, and the whole procedure is very time consuming. Specify
*SPM$\rightarrow$Tools$\rightarrow$Dartel Tools$\rightarrow$Run Dartel
(create Template)*.

- **Run Dartel (create Template)**

  - **Images**

    - **Images**: Select all the rc1\*.nii files generated by the import
      step.

    - **Images**: Select all the rc2\*.nii files, in the same subject
      order as the rc1\*.nii files. The first rc1\*.nii is assumed to
      correspond with the first rc2\*.nii, the second with the second,
      and so on.

  - **Settings**: Default settings generally work well, although you
    could try changing them to see what happens. A series of templates
    are generated called Template_basename_0.nii,
    Template_basename_1.nii etc. If you run multiple Dartel sessions,
    then it may be a good idea to have a unique template basename for
    each.

The procedure begins by computing an initial template from all the
imported data. If u_rc1\*.nii files exist for the images, then these are
treated as starting estimates and used during the creation of the
initial template. If any u_rc1\*.nii files exist from previous attempts,
then it is usually recommended that they are removed first (this sets
all the starting estimates to zero). Template generation incorporates a
smoothing procedure (Ashburner and Friston 2008), which may take a while
(several minutes). Once the original template has been generated, the
algorithm will perform the first iteration of the registration on each
of the subjects in turn. After the first round of registration, a new
template is generated (incorporating the smoothing step), and the second
round of registration begins. Note that the earlier iterations usually
run faster than the later ones, because fewer "time-steps" are used to
generate the deformations. The whole procedure takes (in the order of)
about a week of processing time for 400 subjects.

<figure id="Fig:sharpening" markdown>
<div class="center">
<img src="../../../assets/figures/manual/dartelguide/sharpening.png" style="width:100mm" />
</div>
<figcaption> Different stages of template generation. Top row: an
intermediate version of the template. Bottom row: the final template
data. <span id="Fig:sharpening"
label="Fig:sharpening"></span></figcaption>
</figure>

The end result is a series of templates (see Fig
<a href="#Fig:sharpening" data-reference-type="ref"
data-reference="Fig:sharpening">1.1</a>), and a series of u_rc1\*.nii
files. The first template is based on the average[^5] of the original
imported data, where as the last is the average of the Dartel registered
data. The u_rc1\*.nii files are flow fields that parameterise the
deformations. Note that all the output usually contains multiple volumes
per file. For the u_rc1\*.nii files, only the first volume is visible
using the Display or Check Reg tools in SPM. All volumes within the
template images can be seen, but this requires the file selection to be
changed to give the option of selecting more than just the first volume
(in the file selector, the widget that says "1" should be changed to
"1:2").

### Using Dartel Tools$\rightarrow$Normalise to MNI Space

The next step is to create the Jacobian scaled ("modulated") warped
tissue class images, by selecting
*SPM$\rightarrow$Tools$\rightarrow$Dartel Tools$\rightarrow$Normalise to
MNI Space*. The option for spatially normalising to MNI space
automatically incorporates an affine transform that maps from the
population average (Dartel Template space) to MNI space, as well as
incorporating a spatial smoothing step.

- **Normalise to MNI Space**

  - **Dartel Template**: Specify the last of the series of templates
    that was created by *Run Dartel (create Template)*. This is usually
    called *Template_6.nii*. Note that the order of the *N* volumes in
    this template should match the order of the first *N* volumes of the
    *toolbox/Dartel/TPM.nii* file.

  - **Select according to** either *Few Subjects* or *Many Subjects*.
    For VBM, the *Many Subjects* option would be selected.

    - **Flow Fields**: Specify the flow fields (u_rc1\*.nii) generated
      by the nonlinear registration.

    - **Images**: You may add several different sets of images.

      - **Images**: Select the c1\*.nii files for each subject, in the
        same order as the flow fields are selected.

      - **Images**: This is optional, but warped white matter images can
        also be generated by selecting the c2\*.nii files.

  - **Voxel sizes**: Specify the desired voxel sizes for the spatially
    normalised images (NaN, NaN, NaN gives the same voxel sizes as the
    Dartel template).

  - **Bounding box**: Specify the desired bounding box for the spatially
    normalised images (NaN, NaN, NaN; NaN NaN NaN gives the same
    bounding box as the Dartel template).

  - **Preserve**: Here you have a choice of *Preserve Concentrations*
    (ie not Jacobian scaled) or *Preserve Amount* (Jacobian scaled). The
    *Preserve Amount* would be used for VBM, as it does something
    similar to Jacobian scaling (modulation).

  - **Gaussian FWHM**: Enter how much to blur the spatially normalised
    images, where the values denote the full width at half maximum of a
    Gaussian convolution kernel, in units of mm. Because the
    inter-subject registration should be more accurate than when done
    using other SPM tools, the FWHM can be smaller than would be
    otherwise used. A value of around 8mm (ie $[8, 8, 8]$) should be
    about right for VBM studies, although some empirical exploration may
    be needed. If there are fewer subjects in a study, then it may be
    advisable to smooth more.

The end result should be a bunch of smwc1\*.nii files[^6] (possibly with
smwc2\*.nii if white matter is also to be studied).

<figure id="Fig:VBM">
<div class="center">
<img src="../../../assets/figures/manual/dartelguide/VBM.png" style="width:100mm" />
</div>
<figcaption> Pre-processing for VBM. Top row: Imported grey matter
(rc1A.nii and rc1B.nii). Centre row: Warped with <em>Preserve
Amount</em> option and zero smoothing ("modulated"). Bottom row: Warped
with <em>Preserve Amount</em> option smoothing of 8mm (smwc1A.nii and
smwc1B.nii). <span id="Fig:VBM" label="Fig:VBM"></span></figcaption>
</figure>

The final step is to perform the statistical analysis on the
preprocessed data (smwc1\*.nii files), which should be in MNI space. The
next section says a little about how data from a small number of
subjects could be warped to MNI space.

## Spatially normalising functional data to MNI space

Providing it is possible to achieve good alignment between functional
data from a particular subject and an anatomical image of the same
subject (distortions in the fMRI may prevent accurate alignment), then
it may be possible to achieve more accurate spatial normalisation of the
fMRI data using Dartel. There are several advantages of having more
accurate spatial normalisation, especially in terms of achieving more
significant activations and better localisation.

The objectives of spatial normalisation are:

- To transform scans of subjects into alignment with each other. Dartel
  was developed to achieve better inter-subject alignment of data.

- To transform them to a standard anatomical space, so that activations
  can be reported within a standardised coordinate system. Extra steps
  are needed to achieve this aim.

The option for spatially normalising to MNI space automatically
incorporates an affine transform that maps from the population average
(Dartel Template space) to MNI space. This transform is estimated by
minimising the KL divergence between the final template image generated
by Dartel and tissue probability maps that are released as part of SPM's
segmentation (Ashburner and Friston 2005). MNI space is defined
according to affine matched images, so an affine transform of the Dartel
template to MNI space would appear to be a reasonable strategy.

For GLM analyses, we usually do not wish to work with Jacobian scaled
data. For this reason, warping is now combined with smoothing, in a way
that may be a bit more sensible than simply warping, followed by
smoothing. The end result is essentially the same as that obtained by
doing the following with the old way of warping

- Create spatially normalised and "modulated" (Jacobian scaled)
  functional data, and smooth.

- Create spatially normalised maps of Jacobian determinants, and smooth
  by the same amount.

- Divide one by the other, adding a small constant term to the
  denominator to prevent divisions by zero.

This should mean that signal is averaged in such a way that as little as
possible is lost. It also assumes that the procedure does not have any
nasty side effects for the GRF assumptions used for FWE corrections.

Prior to spatially normalising using Dartel, the data should be
processed as following:

- If possible, for each subject, use
  *SPM$\rightarrow$Tools$\rightarrow$FieldMap* to derive a distortion
  field that can be used for correcting the fMRI data. More accurate
  within-subject alignment between functional and anatomical scans
  should allow more of the benefits of Dartel for inter-subject
  registration to be achieved.

- Use either
  *SPM$\rightarrow$Spatial$\rightarrow$Realign$\rightarrow$Realign:
  Estimate Reslice* or *SPM$\rightarrow$Spatial$\rightarrow$Realign
  Unwarp*. If a field map is available, then use the *Realign Unwarp*
  option. The images need to have been realigned and resliced (or
  field-map distortion corrected) beforehand - otherwise things are not
  handled so well. The first reason for this is that there are no
  options to use different methods of interpolation, so rigid-body
  transforms (as estimated by Realign but without having resliced the
  images) may not be well modelled. Similarly, the spatial transforms do
  not incorporate any masking to reduce artifacts at the edge of the
  field of view.

- For each subject, register the anatomical scan with the functional
  data (using *SPM $\rightarrow$ Spatial $\rightarrow$ Coreg
  $\rightarrow$ Coreg: Estimate*). No reslicing of the anatomical image
  is needed. Use *SPM$\rightarrow$Util$\rightarrow$Check Registration*
  to assess the accuracy of the alignment. If this step is unsuccessful,
  then some pre-processing of the anatomical scan may be needed in order
  to skull-strip and bias correct it. Skull stripping can be achieved by
  segmenting the anatomical scan, and masking a bias corrected version
  (which can be generated by the segmentation option) by the estimated
  GM, WM and CSF. This masking can be done using
  *SPM$\rightarrow$Util$\rightarrow$Image Calculator* (*ImCalc* button),
  by selecting the bias corrected scan (m\*.img), and the tissue class
  images (c1\*.img, c2\*.img and c3\*.img) and evaluating
  "i1.((i2+i3+i4)$>$<!-- -->0.5)". If segmentation is done before
  coregistration, then the functional data should be moved so that they
  align with the anatomical data.

- Segment the anatomical data and generate "imported" grey and white
  matter images.

- To actually estimate the warps, use
  *SPM$\rightarrow$Tools$\rightarrow$Dartel Tools$\rightarrow$Run Dartel
  (create Templates)* in order to generate a series of templates and a
  flow field for each subject.

In principle (for a random effects model), you could run the first level
analysis using the native space data of each subject. All you need are
the contrast images, which can be warped and smoothed. Alternatively,
you could warp and smooth the reslices fMRI, and do the statistical
analysis on the spatially normalised images. Either way, you would
select *SPM$\rightarrow$Tools$\rightarrow$Dartel
Tools$\rightarrow$Normalise to MNI Space*:

- **Normalise to MNI Space**

  - **Dartel Template**: Template_6.nii,1 is usually the grey matter
    component of the final template of the series. An affine transform
    is determined using this image.

  - **Select according to** either *Few Subjects* or *Many Subjects*.
    For fMRI analyses, the *Few Subjects* option would be selected,
    which gives the option of selecting a flow field and list of images
    for each subject.

    - **Subject**

      - **Flow Field**: Specify the flow field ("u_rc1\*.nii") for this
        subject.

      - **Images**: Select the images for this subject that are to be
        transformed to MNI space.

  - **Voxel sizes**: Specify the desired voxel sizes for the spatially
    normalised images (NaN, NaN, NaN gives the same voxel sizes as the
    Dartel template).

  - **Bounding box**: Specify the desired bounding box for the spatially
    normalised images (NaN, NaN, NaN; NaN NaN NaN gives the same
    bounding box as the Dartel template).

  - **Preserve**: Here you have a choice of *Preserve Concentrations*
    (ie not Jacobian scaled) or *Preserve Amount* (Jacobian scaled). The
    *Preserve Concentrations* option would normally be used for fMRI
    data, whereas *Preserve Amount* would be used for VBM.

  - **Gaussian FWHM**: Enter how much to blur the spatially normalised
    images, where the values denote the full width at half maximum of a
    Gaussian convolution kernel, in units of mm.

## Warping Images to Existing Templates

If templates have already been created using Dartel, then it is possible
to align other images with such templates. The images would first be
imported in order to generate rc1\*.nii and rc2\*.nii files. The
procedure is relatively straight-forward, and requires the
*SPM$\rightarrow$Tools$\rightarrow$Dartel Tools$\rightarrow$Run Dartel
(existing Template)* option to be specified. Generally, the procedure
would begin by registering with a smoother template, and end with a
sharper one, with various intermediate templates between.

- **Run Dartel (existing Templates)**

  - **Images**

    - **Images**: Select the rc1\*.nii files.

    - **Images**: Select the corresponding rc2\*.nii files.

  - **Settings**: Most settings would be kept at the default values,
    except for the specification of the templates. These are specified
    in within each of the *Settings$\rightarrow$Outer
    Iterations$\rightarrow$Outer Iteration$\rightarrow$Template* fields.
    If the templates are Template\_\*.nii, then enter them in the order
    of Template_1.nii, Template_2.nii, \... Template_6.nii.

Running this option is rather faster than *Run Dartel (create
Template)*, as templates are not created. The output is in the form of a
series of flow fields (u_rc1\*.nii).

## Warping one individual to match another

Sometimes the aim is to deform an image of one subject to match the
shape of another. This can be achieved by running Dartel so that both
images are matched with a common template, and composing the resulting
spatial transformations. This can be achieved by aligning them both with
a pre-existing template, but it is also possible to use the *Run Dartel
(create Template)* option with the imported data of only two subjects.
Once the flow fields (u_rc1\*.nii files) have been estimated, then the
resulting deformations can be composed using
*SPM$\rightarrow$Utils$\rightarrow$Deformations*. If the objective is to
warp A.nii to align with B.nii, then the procedure is set up by:

- **Deformations**

  - **Composition**

    - **Dartel flow**

      - **Flow field**: Specify the u_rc1A_Template.nii flow field.

      - **Forward/Backwards**: Backward.

      - **Time Steps**: Usually 64.

      - **Dartel template**: leave this field empty.

    - **Dartel flow**

      - **Flow Field**: Specify the u_rc1B_Template.nii flow field.

      - **Forward/Backwards**: Forward.

      - **Time Steps**: Usually 64.

      - **Dartel template**: leave this field empty.

    - **Identity**

      - **Image to base Id on**: Specify B.nii in order to have the
        deformed image(s) written out at this resolution, and with the
        same orientations etc (ie so there is a voxel-for-voxel
        alignment, rather than having the images only aligned according
        to their "voxel-to-world" mappings).

  - **Output**

    - **Pullback**

      - **Apply to**: Specify A.nii, and any other images for that
        subject that you would like warped to match B.nii. Note that
        these other images must be in alignment according to *Check
        Reg*.

      - **Output destination**: Specify where you want to write the
        images.

      - **Interpolation**: Specify the form of interpolation.

      - **Mask images**: Say whether you want to mask the images.

      - **Gaussian FWHM**: The images can be smoothed when they are
        written. If you do not want this, then enter 0 0 0.

Suppose the image of one subject has been manually labelled, then this
option is useful for transferring the labels on to images of other
subjects.

<figure id="Fig:AtoB">
<div class="center">
<img src="../../../assets/figures/manual/dartelguide/AtoB.png" style="width:100mm" />
</div>
<figcaption> Composition of deformations to warp one individual to match
another. Top-left: Original A.nii. Top-right: A.nii warped to match
B.nii. Bottom-left: Original B.nii. Bottom-right: B.nii warped to match
A.nii. <span id="Fig:AtoB" label="Fig:AtoB"></span></figcaption>
</figure>

## Warping the Dartel template to match an individual

Sometimes the aim is to warp one of the templates created by Dartel, or
some other image that is in alignment with this template, to match one
of the scans of an individual. This can be achieved using
*SPM$\rightarrow$Utils$\rightarrow$Deformations* by:

- **Deformations**

  - **Composition**

    - **Dartel flow**

      - **Flow field**: Specify the u_rc1_Template.nii flow field.

      - **Forward/Backwards**: Forward.

      - **Time Steps**: Usually 64.

      - **Dartel template**: leave this field empty.

    - **Identity**

      - **Image to base Id on**: Specify a NIfTI image in order to have
        the deformed image(s) written out at this resolution, and with
        the same orientations etc (ie so there is a voxel-for-voxel
        alignment, rather than having the images only aligned according
        to their "voxel-to-world" mappings).

  - **Output**

    - **Pullback**

      - **Apply to**: Specify the Dartel template or any other images
        that are in alignment with the Dartel template, such as label
        maps etc, that you would like warped to match the individual.
        You can use *Check Reg* to assess this alignment.

      - **Output destination**: Specify where you want to write the
        images.

      - **Interpolation**: Specify the form of interpolation.

      - **Mask images**: Say whether you want to mask the images.

      - **Gaussian FWHM**: The images can be smoothed when they are
        written. If you do not want this, then enter 0 0 0.

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-ashburner07" class="csl-entry">

Ashburner, J. 2007. "A Fast Diffeomorphic Image Registration Algorithm."
*NeuroImage* 38 (1): 95--113.
https://doi.org/<http://dx.doi.org/10.1016/j.neuroimage.2007.07.007>.

</div>

<div id="ref-ashburner05" class="csl-entry">

Ashburner, J., and K. J. Friston. 2005. "Unified Segmentation."
*NeuroImage* 26: 839--51.
<https://doi.org/doi:10.1016/j.neuroimage.2005.02.018>.

</div>

<div id="ref-john_averageshape" class="csl-entry">

---------. 2008. "Computing Average Shaped Tissue Probability
Templates." *NeuroImage* 45 (2): 333--41.
<https://doi.org/10.1016/j.neuroimage.2008.12.008>.

</div>

</div>

[^1]: Dartel stands for "Diffeomorphic Anatomical Registration Through
    Exponentiated Lie algebra". It may not use a true Lie Algebra, but
    the acronym is a nice one.

[^2]: Less smoothing is needed, and there are fewer problems relating to
    how to interpret the differences.

[^3]: More sensitivity could mean that fewer subjects are needed, which
    should save shed-loads of time and money.

[^4]: Other types of scan may also work, but this would need some
    empirical exploration.

[^5]: They are actually more similar to weighted averages, where the
    weights are derived from the Jacobian determinants of the
    deformations. There is a further complication in that a smoothing
    procedure is built into the averaging.

[^6]: The actual warping of the images is done slightly differently,
    with the aim that as much of the original signal is preserved as
    possible. This essentially involves pushing each voxel from its
    position in the original image, into the appropriate location in the
    new image - keeping a count of the number of voxels pushed into each
    new position. The procedure is to scan through the original image,
    and push each voxel in turn. The alternative (older way) was to scan
    through the spatially normalised image, filling in values from the
    original image (pulling the values from the original). The results
    of the pushing procedure are analogous to Jacobian scaled
    ("modulated") data. A minor disadvantage of this approach is that it
    can introduce aliasing artifacts (think stripy shirt on TV screen)
    if the original image is at a similar - or lower - resolution to the
    warped version. Usually, these effects are masked by the smoothing.