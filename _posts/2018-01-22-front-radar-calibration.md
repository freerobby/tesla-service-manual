---
layout: post
title:  "Front Radar Calibration"
published: true
---

## Description

In order for Autopilot to function, radar must be calibrated with the camera. A common symptom of poor calibration is frequent appearance of a "Driver Assistance Features Unavailable" message.

## Applies To

* Pre-facelift Model S vehicles with Autopilot hardware (approximately October '14 through April '16)

## Tools Needed

* 3.5mm nut driver (must be narrow) ([example](https://www.amazon.com/Kocome-Socket-Wrench-Screwdriver-3mm-14mm/dp/B01MSOQR77/ref=sr_1_1?s=hi&ie=UTF8&qid=1516660467&sr=1-1))
* Small bubble level with a 90-degree flat surface ([example](https://www.amazon.com/Spirit-Level-Trailer-Caravan-Camper/dp/B01LAY1OGI/ref=sr_1_16?ie=UTF8&qid=1516660135&sr=8-16))

## Procedure

Before you begin, turn your car off from the center screen. This is not a safety precaution; your car needs to be off for at least two minutes in order to detect a change in radar position.

![Radar Calibration]({{ "/assets/front-radar-calibration-labeled.jpg" | absolute_url }})

### Vertical Calibration

1. Sit in front of your car and look through the lower grill on the bumper cover. Notice the radar in the center, with two 3.5mm hex nuts (one on each side of the radar) behind the grill. The vertical calibration nut is the one on the right.
2. Place the flat surface of your bubble level flush against the radar.
3. Adjust the vertical calibration nut (right side of radar) until the radar is perfectly vertical according to the bubble level (the bubble should be centered).

### Horizontal Calibration

Unfortunately, to receive any feedback from the car about horizontal calibration, you need Tesla's Toolbox software, which is not available to the general public. However, you can get close enough by hand to meet the tolerances for Autopilot.

1. After performing vertical calibration, notice the other 3.5mm hex nut to the left of the radar. This nut is used for horizontal calibration.
2. Using a combination of eyeballing and measuring the distance between the radar and radar frame at each corner, adjust the horizontal calibration nut until the radar appears horizontally centered.
3. Turn on the car. Wait ten seconds. If you see a "Driver Assistance Features Unavailable" message, turn your car off and repeat step 2, ensuring it is off for at least two full minutes between tries. If the DAFU message just won't go away, get as close as you can, and proceed to step 4.
4. If you don't receive the DAFU message, nice work! Take the car for a drive and verify that Autopilot works.

Note: it can take up to 50 miles of driving for Autopilot to self-calibrate after a radar calibration. In such cases you will continue to see the DAFU message for some time even after the radar is properly adjusted.

### Gasket Removal (Recommended)

If your radar still has a beauty gasket around it, and you drive in subfreezing temperatures, it is recommended that you remove it. Instructions can be found [here]({% post_url 2018-01-22-front-radar-beauty-gasket-removal %}).

## References and Citations

I originally posted this information in response to a thread on TMC. It can be found [here](https://teslamotorsclub.com/tmc/posts/2496658).
