---
date: 2026-11-23
tags: [research, data-format, mcap, hdf5, lerobot, recording]
---
# Robot data formats: MCAP, rosbag, HDF5, Zarr, LeRobot, RLDS

Companion to the explainer page `2026-11-23 Robot Data Formats.html` (same folder). Closes the review of [[2026-11-19 UMI-style recording and training pipelines]]: Martyn chose on 2026-11-23 raw MCAP recording, offline alignment, then LeRobot export (see also [[2026-11-23 RL5 sync test findings]]). Purpose: one place to cite when the thesis explains the format choice.

Method: official specs, docs and source code fetched 2026-11-23 (links inline); bib papers from [[2026-11-19 UMI-style recording and training pipelines]]. Papers not in `msc-thesis/thesis/literature.bib` are marked "not in bib". Own arithmetic is marked "computed". Anything not checked is marked "unverified".

## Words used below

- **Format**: the rule for how bytes are laid out in a file, so that another program can read them back.
- **Stream / topic / channel**: one sensor's sequence of messages (e.g. camera images, controller pose).
- **Timestamp**: the time attached to one message. ns = nanoseconds.
- **Episode**: one demonstration from start to stop.
- **Raw log**: everything as it arrived, each stream at its own rate. **Training dataset**: streams already aligned to one clock tick (e.g. 30 Hz), ready for a policy.
- **Chunk**: a block of data stored together, often compressed, so a reader can jump to it without reading the rest.
- **Index**: a table at a known place in the file that says where each chunk or message is.
- **Schema**: the description of a message's fields (names and types).
- **Compression**: shrinking data; *lossless* gets every bit back, *lossy* (e.g. video) throws some detail away.

## Two jobs, two kinds of format

Robot learning data passes through two stages, and formats are built for one or the other:

1. **Recording (log formats)**: write many streams at their own rates, as they arrive, without losing anything if the computer stops. MCAP, ROS 1 bag, rosbag2 (`.db3`).
2. **Training (dataset formats)**: give a training program fast, random access to aligned frames grouped into episodes. HDF5 (robomimic, ACT style), Zarr (UMI, Diffusion Policy), LeRobot (Parquet + MP4), RLDS (Open X-Embodiment).

Some projects use one training format for both jobs (FastUMI writes HDF5 directly from ROS, zhaxizhuoma2025 III-A), but then the alignment is fixed at record time.

## Formats

### MCAP (`.mcap`), log format

- **Inside**: `Magic, Header, Data section, [Summary section], [Summary Offset section], Footer, Magic` ([MCAP spec](https://mcap.dev/spec)). Data section = a sequence of records: **Schema** (field description), **Channel** (one stream, points to a schema), **Message** (one timestamped message on a channel). Messages can be grouped into **Chunks**, compressed with `zstd` or `lz4`; **Message Index** records locate messages inside a chunk by timestamp (spec). The optional Summary at the end holds Chunk Index and Statistics records for fast lookup (spec).
- **Timestamps**: every Message carries `log_time` ("time at which the message was recorded") and `publish_time` ("time at which the message was published"), both uint64 ns (spec).
- **Pros**: "row-oriented, append-only design to minimize disk I/O and reduce the risk of data corruption during unclean shutdowns"; "stores message schemas alongside data, so your files remain readable in the future"; any message encoding: "Protobuf, DDS (CDR), ROS, JSON, and more" ([mcap.dev](https://mcap.dev/)). Many streams at different rates in one file.
- **Cons**: not a training format: no episode or frame-grid structure, so a conversion step is always needed (no training framework among GR00T, openpi, LeRobot was found reading MCAP directly; unverified beyond those READMEs). Fewer robot-learning papers use it than HDF5 (see pipeline review).
- **Who**: "the default log format in ROS 2" (mcap.dev); ROS 2 switched rosbag2's default to MCAP in the Iron release ([ROS 2 Iron notes](https://docs.ros.org/en/rolling/Releases/Release-Iron-Irwini.html)). RAPID records MCAP with `timestamp_ns` (yin2026, App. A, D; [S] in the pipeline review: from a summary, check in the PDF before citing). Viewer: Foxglove ([mcap.dev](https://mcap.dev/)).

### ROS 1 bag (`.bag`) and rosbag2 SQLite (`.db3`), log formats

- **ROS 1 bag v2.0**: records with op codes (bag header, chunk, connection, message data, index data, chunk info); adds compression and indexing ([ROS wiki Bags/Format/2.0](http://wiki.ros.org/Bags/Format/2.0)). MCAP was "influenced by the original ROS 1 bag file format" ([ROS 2 Iron notes](https://docs.ros.org/en/rolling/Releases/Release-Iron-Irwini.html)).
- **rosbag2 `.db3`**: a SQLite database (rows of messages). Before Iron it was the rosbag2 default; ROS found that "in many cases this was not performant enough and lacked certain features desirable for offline processing", so Iron made MCAP the default; `.db3` can still be written and played ([ROS 2 Iron notes](https://docs.ros.org/en/rolling/Releases/Release-Iron-Irwini.html)). rosbag2 compression by file or message makes files no longer indexable, unlike MCAP's own chunk compression ([rosbag2 README](https://github.com/ros2/rosbag2)).
- **Who**: ROS 1 and older ROS 2 setups. Not relevant here: the thesis uses no ROS 2 (Replay Ladder decision, 2026-11-20).

### HDF5 (`.hdf5`, `.h5`), training format (sometimes recording)

- **Inside**: a file system inside one file. "The two primary objects in the HDF5 Data Model are groups and datasets"; **groups** are folders, **datasets** are N-dimensional arrays, **attributes** are name-value labels on either; datasets can be "chunked and compressed" ([HDF Group intro](https://support.hdfgroup.org/documentation/hdf5/latest/_intro_h_d_f5.html)).
- **robomimic layout** (not in bib): one file per dataset; `data/` group with attributes `total`, `env_args`; one group per demo `data/demo_0`, `data/demo_1`, … each with `num_samples`; optional `mask/` group of filter keys such as `train` / `valid` ([robomimic docs](https://robomimic.github.io/docs/datasets/overview.html)).
- **ACT layout** (ACT, Zhao et al. 2023, not in bib): one file per episode; `/observations/images/<camera>` uint8 array `(T, 480, 640, 3)`, `/observations/qpos` and `/observations/qvel` `(T, 14)`, `/action` `(T, 14)`, attribute `sim` ([ACT `record_sim_episodes.py`](https://github.com/tonyzhaozh/act/blob/main/record_sim_episodes.py)). Images stored uncompressed: 480 × 640 × 3 = 921,600 bytes per frame, ≈ 27.6 MB/s per camera at 30 Hz (computed).
- **Pros**: one file, mature, fast array slicing, widely used in imitation learning code.
- **Cons**: no official guarantee that an unclosed file survives a crash; corruption after a crash is reported ([HDF forum 3481](https://forum.hdfgroup.org/t/file-state-after-flush-and-crash/3481)); single-writer/multi-reader mode cannot append variable-length data ([HDF Group SWMR intro](https://support.hdfgroup.org/documentation/hdf5-docs/advanced_topics/intro_SWMR.html)). Multi-rate streams need a self-made layout.
- **Who**: FastUMI writes HDF5 from a ROS storage node (zhaxizhuoma2025 III-A); RoboManipBaselines uses HDF5 + MP4 per episode (murooka2026 IV-B); ACT and robomimic (not in bib).

### Zarr (`.zarr` folder or `.zarr.zip`), training format

- **Inside**: "storage of n-dimensional arrays, supporting chunking, compression, and various backends"; Zarr format 2 and 3 ([Zarr docs](https://zarr.readthedocs.io/en/stable/)). Each array is cut into chunks, each chunk stored as its own object (folder of files, zip or cloud store).
- **Diffusion Policy / UMI ReplayBuffer**: groups `data/` (one array per field, all episodes concatenated end to end) and `meta/episode_ends` (index of the last step of each episode); the code checks that every `data` array has length `episode_ends[-1]` ([UMI `replay_buffer.py`](https://github.com/real-stanford/universal_manipulation_interface/blob/main/diffusion_policy/common/replay_buffer.py)). "Stored both in-memory and on-disk with chunking and compression"; on disk "as a nested directory … or a zip file" ([Diffusion Policy README](https://github.com/real-stanford/diffusion_policy), not in bib). UMI's pipeline ends with `07_generate_replay_buffer.py -o dataset.zarr.zip` and training reads that file ([UMI README](https://github.com/real-stanford/universal_manipulation_interface)).
- **Pros**: chunks read in parallel; whole dataset can be loaded into memory; simple concatenated layout.
- **Cons**: tied in practice to the Diffusion Policy / UMI code; neither GR00T nor openpi reads it (their READMEs name LeRobot, see below).
- **Who**: UMI (chi2024) and UMI-derived work; FastUMI HDF5 → zarr ([S] in pipeline review, unverified).

### LeRobot dataset (Parquet + MP4), training format

- **Inside (v3.0, current)**: `meta/info.json`, `meta/stats.json`, `meta/tasks.parquet`, `meta/episodes/chunk-000/file-000.parquet`; `data/chunk-000/file-000.parquet` (states, actions, timestamps per frame); `videos/<camera>/chunk-000/file-000.mp4` ([LeRobot `utils.py`](https://github.com/huggingface/lerobot/blob/main/src/lerobot/datasets/utils.py)). "Many episodes per Parquet/MP4 file (v2 used one file per episode)" ([LeRobot v3 docs](https://github.com/huggingface/lerobot/blob/main/docs/source/lerobot-dataset-v3.mdx)). Low-rate signals in **Parquet** (a column table format); images as **MP4** video, decoded so frames are "synchronous with data from parquet files" ([LeRobot `lerobot_dataset.py`](https://github.com/huggingface/lerobot/blob/main/src/lerobot/datasets/lerobot_dataset.py)).
- **GR00T flavour (v2)**: "a flavor of the LeRobot v2 dataset format" with `meta/info.json`, `episodes.jsonl`, `tasks.jsonl` and an extra `meta/modality.json` that splits state/action arrays into named fields ([Isaac-GR00T README](https://github.com/NVIDIA/Isaac-GR00T)).
- **Pros**: the input format of both target models: openpi "Convert your data to a LeRobot dataset (which we use for training)" ([openpi README](https://github.com/Physical-Intelligence/openpi)); GR00T as above. Video makes images small; Hugging Face Hub hosting, streaming and a visualiser ([LeRobot v3 docs](https://github.com/huggingface/lerobot/blob/main/docs/source/lerobot-dataset-v3.mdx)).
- **Cons**: images are video-compressed, so frames are not bit-exact (codec settings not checked, unverified); format versions change (v2.1 → v3.0 conversion script, `lerobot_dataset.py`), and GR00T still expects v2.
- **Who**: openpi (π0 / π0.5), GR00T, Grabette ("the output is a standard LeRobot dataset", [Grabette blog](https://huggingface.co/blog/grabette), not in bib).

### RLDS (TFRecord via TensorFlow Datasets), training format

- **Inside**: a dataset is a `tf.data.Dataset` of **episodes**, each holding a `tf.data.Dataset` of **steps**; a step has `observation`, `action`, `reward`, `discount`, and flags `is_first`, `is_last`, `is_terminal`; episodes carry metadata such as `episode_id` and `invalid` ([RLDS README](https://github.com/google-research/rlds)). Files are TFDS records (TFRecord).
- **Pros**: one shared layout across many labs; streams large datasets.
- **Cons**: tied to TensorFlow; openpi's own example takes LIBERO data in RLDS and converts it to LeRobot before training ([openpi README](https://github.com/Physical-Intelligence/openpi)).
- **Who**: Open X-Embodiment, "1M+ real robot trajectories spanning 22 robot embodiments" ([OXE site](https://robotics-transformer-x.github.io/), not in bib), "represented using the RLDS episode format" ([OXE repo](https://github.com/google-deepmind/open_x_embodiment)).

### Others seen

- **Plain MP4 + sidecar files**: UMI's raw data is the GoPro MP4 per demo, with IMU inside the video ([S], pipeline review).
- No other format was found in the bib papers read.

## Comparison

| Format | Job | Holds many rates with original timestamps | Crash-safe writing | Read by GR00T / openpi | Precedent (bib) |
| --- | --- | --- | --- | --- | --- |
| MCAP | Recording | Yes, ns `log_time` + `publish_time` per message | Designed for it (append-only) | No, convert | RAPID (yin2026) |
| ROS 1 bag / `.db3` | Recording | Yes | `.db3` replaced for performance | No | – |
| HDF5 | Training (FastUMI also records) | Own layout needed | No guarantee | No, convert | FastUMI, RoboManipBaselines |
| Zarr | Training | No, already aligned | – (written offline) | No | UMI (chi2024) |
| LeRobot | Training | No, aligned frames | – (written offline) | Yes (GR00T v2 flavour; openpi) | – (Grabette, not in bib) |
| RLDS | Training | No, aligned steps | – | No, openpi converts | – (OXE, not in bib) |

## Why the thesis uses MCAP → offline alignment → LeRobot

1. **Record raw in MCAP.** Camera, controller pose, trigger and gripper arrive at different rates and delays. MCAP keeps every message with its own ns timestamps (spec) and is built to survive an unclean stop (mcap.dev). Precedent: RAPID (yin2026). Matches the approved proposal (`msc-thesis/docs/proposal.md`, sec. 6–7).
2. **Align offline, not at record time.** Delays per stream will be re-measured at RL5 (D435i + recorder test) and again at RL6 (Dex1-1 gripper and camera swap); see [[2026-11-23 RL5 sync test findings]]. Keeping raw data lets old recordings be re-aligned with new delay numbers. Method: subtract each delay, interpolate pose and width at camera frame times (UMI chi2024 PD1.1; ViTaMIn-B li2025 IV-B). FastUMI stores only aligned pairs (zhaxizhuoma2025 III-A), so it cannot re-align.
3. **Export to LeRobot.** It is the input format of both target models: openpi and GR00T (READMEs above). Zarr and RLDS would need a second conversion. D07 (software and data format) already names LeRobot export.

## Open

- **LeRobot version**: GR00T expects a v2 flavour plus `modality.json`; LeRobot `main` writes v3.0 and ships only a v2.1 → v3.0 converter. Which version openpi's pinned LeRobot commit (`0cf8648`, [openpi pyproject](https://github.com/Physical-Intelligence/openpi/blob/main/pyproject.toml)) writes and reads: unverified. Decide whether the exporter writes v2.1 for both, or two outputs. Matches the existing open question in `msc-thesis/docs/proposal.md` ("which LeRobot dataset version does the team's GR00T or openpi setup use?").
- **Video codec loss** in LeRobot export: codec and quality not checked; check effect on small objects if relevant.
- **RAPID MCAP detail** is [S] (from summary); check yin2026 App. A, D before citing in the thesis.
- **Thesis citations**: ACT, robomimic, Diffusion Policy, Open X-Embodiment, LeRobot are not in the bib; add to Zotero if the thesis text cites them.
