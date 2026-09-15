---
url: https://github.com/enactic/OpenArm
title: "OpenArm — open-source 7DOF humanoid arm for physical AI research"
author: Enactic
date_fetched: 2026-09-15
topics:
  - misc
---

OpenArm is Enactic's open-source humanoid arm platform: a 7DOF arm (bimanual pairs give a 16-DOF action space) built around backdrivable quasi-direct-drive Damiao motors, sold as a complete bimanual system for $6,500, and released from CAD (CERN-OHL-S) to control code (Apache-2.0). The GitHub repo ingested here is the project's documentation monorepo — a Docusaurus site with versioned docs — but its docs are a deep technical map of a nine-repo ecosystem covering hardware, CAN control, ROS2, teleoperation, simulation, dataset tooling, and policy inference.

The distinctive claim is not the arm itself but the loop built around it. Teleoperation comes in three forms: bilateral force-feedback leader–follower (enabled by the motors' backdrivability, with tanh-based friction compensation, run at 500 Hz+), VR controller teleop, and the KER — a motorless, encoder-only leader arm worn as a 1.7 kg backpack, scaled to 70% with exact kinematic matching so motion maps 1:1 with no retargeting. Collected data lands in an open episode format (parquet joint series + nanosecond-timestamped JPEG frames) that converts to LeRobot v2.1/v3.0 for training ACT policies with LeRobot's tooling.

Inference is the most thought-through part: a Dora dataflow runs a 250 Hz control loop, packs five camera frames and joint positions into an Arrow IPC file on shared memory, and hands them to a "policy server" over a Unix socket with a one-page JSON contract. The model returns an action chunk at its own rate (30 Hz in the example); an executor Hermite-upsamples to 250 Hz and applies a 15 Hz low-pass filter, with the cutoff frequency specified by the policy per response. The same contract drives either a MuJoCo simulator node or real hardware, making sim-to-sim and sim-to-real deployment a config change.

The third pillar is evaluation: the OpenArm Cell is a ~100 kg standardized enclosure fixing lighting, background, camera placement, and arm position, with an area-sensor reach-in stop and a zero-position calibration jig that mechanically constrains the gripper to CAD-defined angles so assembly tolerance never enters the dataset. The thesis, stated plainly in the docs: "Model A outperforms Model B" only means something when both were evaluated under identical conditions — benchmark reproducibility treated as a hardware product.
