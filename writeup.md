# **Finding Lane Lines on the Road**

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:

- Make a pipeline that finds lane lines on the road

- Reflect on your work in a written report

---

### Reflection

### 1. Pipeline

My pipeline consisted of 4 steps.

- Grayscale.

- Gaussian blur. I have found kernel size 5 to be enough, don't feel comfortable losing even more information by blurring image further

- Canny edge detection. Final version uses thresholds [50; 120]. Those thresholds served me well until "challenge" part, where there is a very sunny spot, and difference between lane line brightness and road brightness is minimal. Thresholds [10; 20] were able to find those lines even in the sunny spot, but introduced too much noise and instability, so I've returned to initial, stricter values.

- Region limitation.
    ![Region limitation example][./examples/region]

- Hough transformation. Played with values for a long time, decided to leave fairly liberal values - canny edge detection is strict and filters out most noise, so it was better to leave Hough liberal so it can still catch short lane lines far away.

That was pretty much it, I was surprised at how well it worked right from the start (apart from challenge, that is)

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by "clustering" lines from Hough output into groups (depending on how similar line parameters are - angle and offset). Then I take 2 largest (by sum of lines lengths inside cluster) clusters, and use parameters of first line to draw the guideline.

That was pretty much enough, but during my attempt at solving challenge, I devised several enhancements:

- Filter out lines with obviously wrong angle right at the start

- Use angle in radians instead of slope coefficient to estimate cluster similarity - slope increases rapidly for vertical lines, while angle increases linearly

- To draw the guideline instead of the first cluster line use average of parameters of all cluster lines - weighted by their respective lengths. Makes guideline movements not that janky.

All that increased stability in challenge, but not enough, unfortunately.

### 2. Shortcomings and improvements

- Coefficients for everything besides region are hardcoded at the moment, which is no good, because image resolution varies
    
    - Coefficients for Gaussian Blur, Canny and Hough must depend on resolution

- Very light or very dark environment (as was evident from the challenge) leads to inconsistent results
    
    - We could increase contrast of obviously-too-bright-areas right at the beginning of pipeline

    - Making Canny more liberal but Hough stricter should help in theory, but I wasn't able to find good Hough parameters
    
    - Something that I guess is mandatory for real world algorithm - reliance on previous results. For example, we could remember results from last, say, 5 frames, and make it so new guideline is slightly closer to the old results - will make movement very fluent. 
    
    Even better, we could estimate the "reliability" of our results, and if they are too bad (for example, drastically differ from previous frames), we could ignore our results for couple of frames whatsoever, and use result from previous reliable frame.

    I didn't implement it because I thought it would be considered cheating, because initially function draw_lines doesn't have any reference to previous frames results.

- Debris or even shadows on the road will confuse the algorithm

    - we could limit the region further, so there is section for left line and right line, and the middle is filtered out

    - we could hardcode "ideal" theoretical guidelines parameters, and choose cluster not depending on its size, but depending on its deviation from "ideal" guidelines.

Overall, less reliance on contrast and more reliance on previous results and our "guess" where line should be would be a good improvement I think.

