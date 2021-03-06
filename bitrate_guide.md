Bitrate guide for BELABOX on Jetson Nano
========================================

Video quality is influenced by many factors, including the camera's output quality, scene complexity, the amount and speed of motion, resolution, framerate, encoder used and bitrate. The table below is intended to provide a starting point when using BELABOX with Jetson Nano's H265 encoder for IRL streaming and it's based on my subjective evaluation and [VMAF](https://github.com/Netflix/vmaf) assessment. Note that in the context of live streaming, video quality is almost always lower compared to recorded video in the interest of stability over mobile connections (and data transfer costs).

| Resolution and framerate | Type of scene    | Bitrate (13" size)* | Bitrate (27" size) ** |
| ------------------------ | ---------------- | ------------------- | --------------------- |
| 1080p60                  | nature / foliage | 6000-7000 Kbps      | 8000-10000 Kbps       |
| 1080p60                  | indoor / urban   | 3000-4000 Kbps      | 5000-7000 Kbps        |
| 1080p30                  | nature / foliage | 4000-5000 Kbps      | 6000-8000 Kbps        |
| 1080p30                  | indoor / urban   | 2500-4000 Kbps      | 4000-6000 Kbps        |

\*  The bitrate to typically get good image quality on a 13" monitor - *good enough* for most viewers / good choice for stability in poorer reception areas and to reduce data costs. VMAF score typically in the 75-85 range.

\*\* The bitrate to typically get good image quality on a 27" monitor - comparable to other state of the art IRL live streaming setups. VMAF score of 85 and over.

As a general recommendation, avoid streaming at 50 / 60 fps unless it's strictily required, as it can require up to 50% higher bitrate to match the video quality of a 25/30 fps stream, for minimal noticeable differences at 1x speed in most conditions.
