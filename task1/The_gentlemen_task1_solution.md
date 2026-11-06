# Olympus Tech 4.0 - Part One: Signal in the Static

**Team:** The gentlemen



## 1. Overview

Our approach was:

1. Load and inspect every FITS frame.
2. Build a reference image representing the fixed sky.
3. Subtract the reference from every frame.
4. Detect possible sources using the measured noise level.
5. Link detections that follow a smooth path across the full sequence.
6. measure the object's position and signal strength.
7. Test whether the detection could be caused by noise or image artefacts.

The final result is a reliable detection in all 20 frames. The source follows a smooth, curved trajectory from approximately **(39.80, 44.67)** to **(65.53, 53.86)** pixels.

---

## 2. Loading and Displaying the FITS Frames

The FITS files were extracted from `/content/fits.zip`, naturally sorted by filename, and loaded as floating-point image arrays. Missing pixel values, if present, were replaced with the median value of the corresponding frame.

Every raw frame was plotted with:

- X and Y pixel coordinates;
- a grayscale intensity display;
- a labelled colorbar showing flux in ADU;
- a common display range for easier comparison between frames.

The fixed stars remain in the same positions throughout the sequence. The moving object is too faint to follow confidently in the raw-frame montage alone, which is why reference subtraction is necessary.

---

## 3. Median Reference and Difference Images

### 3.1 Building the reference

The background level changes slightly between frames. We first measured the typical sky level in each image and corrected these small differences.

A median reference image was then constructed from the aligned frames. Taking the median preserves sources that remain fixed, such as stars, while rejecting sources that move between frames.

In the final version, pixels around the preliminary object trajectory and confirmed sharp artefacts were excluded before the reference was calculated. This prevents a faint copy of the moving object from being included in the reference.

### 3.2 Difference images

Each corrected frame was subtracted from the median reference:


Fixed stars largely disappear because they are present in both images. A moving source remains as a positive point-like feature because it is present at a different position in each frame.

The green circles in the montage mark the recovered object position. Its steady movement from left to right is much clearer in the difference images than in the original frames.


## 4. Candidate Detection and Source Identification

The noise level was measured directly from each difference image. Candidate sources were required to be clearly brighter than the surrounding noise and to cover more than one connected pixel.

This rejected most random noise fluctuations. Very sharp one-pixel events were treated separately because they are more consistent with cosmic rays or hot pixels than with an astronomical point source.

The final moving source was distinguished from other candidates because it:

- was detected consistently across all 20 frames;
- followed a smooth path;
- had the same point-like shape as the stars;
- remained strong enough to detect in every frame;
- did not behave like a random isolated pixel;
- appeared only when the frames were combined using the correct direction and timing of motion.

The earlier method flagged 28 possible sharp artefacts, but 7 of these were actually bright pixels belonging to the moving object. The improved method identified only **3 sharp artefacts**, with **none located on the object**.

The residual panels show what remains after subtracting the fitted object model. They contain mostly ordinary noise and no strong source-shaped pattern, indicating that the measured positions and source shape are reasonable.

---

## 5. Reconstructed Trajectory

The object's position was measured by fitting a circular point-source model to a small region around it in each frame. The model accounts for the fact that each detector pixel records light over a finite area.

The resulting path is gently curved. We compared straight-line, quadratic, and cubic descriptions of the motion:

| Motion model | 2D RMS error | BIC | Interpretation |
|---|---:|---:|---|
| Linear | 0.736 px | 597.6 | Does not describe the curvature |
| Quadratic | **0.213 px** | **66.0** | Best balance of fit and simplicity |
| Cubic | 0.205 px | 69.3 | Slightly lower RMS, but unnecessary complexity |

The quadratic model was selected because it is strongly better than the straight-line model, while the cubic model does not provide a meaningful improvement.

### 5.1 Measured motion

- Starting position: approximately **(39.80, 44.67) px**
- Ending position: approximately **(65.53, 53.86) px**
- Mean apparent speed: approximately **1.46 px per frame**
- Mid-sequence horizontal velocity: **1.357 ± 0.007 px per frame**
- Mid-sequence vertical velocity: **0.477 ± 0.007 px per frame**

The horizontal acceleration is consistent with zero. The vertical acceleration is clearly detected:

- Horizontal acceleration: **−0.0013 ± 0.0026 px per frame²**
- Vertical acceleration: **+0.0470 ± 0.0028 px per frame²**

This explains why the object follows a curved path rather than a perfectly straight one.

### 5.2 Possible object type

The source is point-like and moves smoothly against the fixed background stars. This is consistent with a nearby moving object, such as an asteroid, comet, or artificial satellite.

The available data do not include sufficient timing, pixel-scale, or sky-coordinate information to identify the object type with certainty. We therefore avoid assigning a definite physical classification from image positions alone.

---

## 6. Noise Level and Signal Strength

The average noise level in the difference images is:

> **2.80 ± 0.02 ADU per pixel**

The source has:

- mean fitted flux: approximately **124.9 ADU**;
- matched-filter SNR range: **8.7 to 13.2** per frame;
- mean matched-filter SNR: **11.0**;
- mean peak-pixel SNR: approximately **5.6**.

An SNR of about 11 means the full point-source pattern is much stronger than the expected random noise. The peak-pixel SNR is lower because the object's light is distributed across several pixels.

The noise is almost uniform over most of the image. Frames where the moving source passes close to a fixed star have slightly more uncertainty because the star adds extra photon noise.

---

## 7. Reliability Tests

### 7.1 Injection and recovery

Artificial point sources with similar brightness to the detected object were inserted into source-free parts of the real difference images. The same fitting process was then used to recover their positions.

The recovered artificial sources had:

- X-position scatter: **0.141 px**
- Y-position scatter: **0.149 px**
- Combined 2D positional uncertainty: approximately **0.206 px**

The actual trajectory has a 2D RMS error of **0.213 px**, which is very close to this independently measured uncertainty. This indicates that the remaining trajectory scatter is mainly caused by image noise, rather than an incorrect motion model.

The average injection bias was less than 0.01 pixel in both directions.

The source detector achieved approximately:

| Injected SNR | Recovery rate |
|---:|---:|
| 3 | 7% |
| 4 | 28% |
| 5 | 54% |
| 6 | 85% |
| 7 | 97% |
| 8 or higher | 100% |

Since the real object's per-frame SNR is between 8.7 and 13.2, it lies in the range where the recovery rate is effectively 100%.

### 7.2 Negative-image test

The detection procedure was also run on inverted difference images. Real positive sources become negative in these images, so any positive detections provide an estimate of the false-alarm rate.

The negative images produced an average of only **0.10 false detections per frame**. After also requiring a candidate to follow the same smooth path through all 20 frames, the estimated probability of a random noise track became extremely small.

### 7.3 Shift-and-stack test

The images were shifted so that the recovered object position aligned between frames, and then combined. This produced a clear point source with an empirical significance of **35.7σ**.

The same test was repeated using noise-only tracks and the same spatial path in reverse time:

- real time-ordered trajectory: **35.7σ**;
- strongest tested noise-only track: **2.6σ**;
- reversed-time path: consistent with noise at approximately **−0.6σ**.

This provides strong evidence that the signal is associated with the recovered motion and is not a fixed star or random image fluctuation.

---

## 8. Improvement Over the Initial Analysis

The updated method was compared directly with the original pipeline using the same data:

| Metric | Initial result | Final result | Improvement |
|---|---:|---:|---:|
| 2D trajectory RMS | 0.242 px | **0.213 px** | **12.0% lower** |
| Leave-one-out RMS | 0.285 px | **0.253 px** | **11.2% lower** |
| Median positional uncertainty | 0.155 px | **0.138 px** | **11.0% lower** |
| R² for X | 0.99970 | 0.99969 | No meaningful change |
| R² for Y | 0.99510 | **0.99676** | Improved |
| Mean matched-filter SNR | 10.30 | **11.03** | **7.1% higher** |

The tiny reduction in X R² is not meaningful. The more useful measures trajectory RMS, cross-validated error, uncertainty, Y R², and signal strength all improved.

The final RMS is also close to the 0.206-pixel precision measured with artificial-source tests. This suggests that there is little room for further improvement without collecting higher-quality data.

---

## 9. Limitations

The main limitations are:

1. Only 20 frames are available.
2. The object passes near a fixed star in part of the sequence, increasing the local noise.
3. A constant point-source width is assumed across the frames.
4. The median reference introduces a small amount of shared noise between difference images.
5. No observation times, pixel scale, telescope information, or sky-coordinate calibration were supplied.
6. Pixel speeds therefore cannot be converted into angular or physical velocities.
7. The image sequence alone is insufficient to determine the object's exact physical type.

Despite these limitations, the source is detected independently in every frame, follows a well-supported trajectory, and passes multiple false-alarm and injection-recovery tests.

---

## 10. Conclusion

We successfully recovered a faint moving point source from 20 noisy FITS images.

Median-reference subtraction removed the fixed sky, noise-based detection identified candidate sources, and trajectory linking isolated the only source that moved consistently through the entire sequence. The final quadratic trajectory has a **2D RMS error of 0.213 pixels**, compared with **0.242 pixels** in the initial analysis.

Artificial-source tests measured an expected positional precision of approximately **0.206 pixels**, closely matching the observed trajectory residuals. The object has a mean per-frame matched-filter SNR of **11.0**, and combining all frames along the path produces a **35.7σ** detection.

These results show that the moving signal is reliable, the measured curvature is real, and the final trajectory precision is close to the limit allowed by the image noise.
