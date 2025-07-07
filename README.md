# TrafficComposer
The official repo for the FSE 2025 paper "Multi-modal Traffic Scenario Generation for Autonomous Driving System Testing".

[Project Homepage](https://hcss.cs.purdue.edu/trafficcomposer/) | [GitHub](https://github.com/TrafficComposer/TrafficComposer/) | [arXiv](https://arxiv.org/pdf/2505.14881)

<p align="center">
  <img src="./resources/pipeline.jpg"/>
</p>

## Multi-Modality Traffic Scenario IR Generation

TrafficComposer extracts information from both textual and visual inputs and merges the information into a comprehensive traffic IR.

To run TrafficComposer, user first needs to rename the `trafficcomposer/config_template.py` to `trafficcomposer/config.py`, and set the corresponding file paths.

### 1. Extract information from the textual description:

```
cd trafficcomposer/gen_textual_ir
python gpt_text_parser.py
```

By default, TrafficComposer uses GPT-4o (*gpt-4o-2024-05-13*).


### 2. Extract information from the reference image:

TrafficComposer proposes a visual information extractor using two pre-trained computer vision models, [YOLOv10](https://github.com/THU-MIG/yolov10) and [CLRNet](https://github.com/Turoad/CLRNet).

#### 2.1 Extract lane divisions from the reference image:

To extract the lane dividing lines in the reference image, user needs [CLRNet](https://github.com/Turoad/CLRNet).
User would need to follow [CLRNet](https://github.com/Turoad/CLRNet)'s instructions to install it and download the pre-trained weight file(s).

By default, the weigh file should be saved at `trafficcomposer/gen_visual_ir`.

```
cd trafficcomposer/gen_visual_ir
python extract_lane.py clrnet/configs/clrnet/clr_dla34_culane.py --load_from culane_dla34.pth --gpus 0
```

#### 2.2 Detect vehicles and pedestrains from the reference image:

To extract the positions of actors in the reference image, user needs [YOLOv10](https://github.com/THU-MIG/yolov10) to detect each actor in the reference image.
User would need to follow [YOLOv10](https://github.com/THU-MIG/yolov10)'s instructions to install it.

```
cd trafficcomposer/gen_visual_ir
python extract_actor.py
```

#### 2.3 Generate the visual IR:
```
cd trafficcomposer/gen_visual_ir
python gen_visual_ir.py
```

### 3. Align the information from the two modalities to a comprehensive traffic IR.

```
python trafficcomposer.py
```


## Cite
```
@article{10.1145/3729348,
  author = {Tu, Zhi and Niu, Liangkun and Fan, Wei and Zhang, Tianyi},
  title = {Multi-modal Traffic Scenario Generation for Autonomous Driving System Testing},
  year = {2025},
  issue_date = {July 2025},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  volume = {2},
  number = {FSE},
  url = {https://doi.org/10.1145/3729348},
  doi = {10.1145/3729348},
  journal = {Proc. ACM Softw. Eng.},
  month = jun,
  articleno = {FSE078},
  numpages = {24},
  keywords = {Autonomous Driving System, Software Testing, Traffic Scenario Generation}
}
```
