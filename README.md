# GSoC'21 Project Logs

This repository serves as a log for the weekly progress of my GSoC'21 project with Intel Video and Audio for Linux. The discussions with the mentors are not logged in this log.

If you have any questions, please feel free to create a new discussion in Discussion section of this repository.

## Project Details

Project Title: **Async Support for TensorFlow Backend in FFmpeg**

Project: [GSoC Link](https://summerofcode.withgoogle.com/projects/#5224576843251712)

Proposal: [Google Docs Link](https://docs.google.com/document/d/1J79_Id4XDYfMSJh94q11kHm1SjewYoLAOEX15uwNkhU/edit?usp=sharing)

Final Report: [Medium Article](https://medium.com/@shubhanshu02/async-support-for-tensorflow-backend-in-ffmpeg-695998cd439c?source=friends_link&sk=ae01e6f2a982a387658f285ce9b0d9d1)

## Abstract

This project focuses on implementing an asynchronous mechanism for model inference and batch execution in the TensorFlow backend of the FFmpeg Deep Neural Network module to boost model inference performance.

The Tensorflow backend uses the TensorFlow C API, which currently does not provide functions for asynchronous execution. The support for async behavior can be provided using multithreading on the existing TensorFlow library functions. We will implement this behavior through detached threads that work independently of each other.

Several inference frames will be combined into a single input tensor and executed together in a single batch to enable the batch mode. The DNN module authors saw a performance gain in the OpenVino backend with asynchronous batch inference against synchronous inference. A similar performance gain is expected from this project.

## GSoC'21 Deliverables

1. **Async Support in TensorFlow backend (Required):**

   Currently, the TensorFlow backend supports only the synchronous mode of model inference, which is single-threaded and slow. Using asynchronous mode in a multithreaded environment will provide us with a higher CPU utilization and faster execution due to its non-blocking nature.

2. **Async Support in the Native Backend (Optional)**

   The native backend is used for model inference when the target system does not support OpenVino or TensorFlow backend. This backend also currently supports only the synchronous model execution. We can also extend the async support in the TensorFlow backend using detached threads to the native backend.

3. **Support for Batch Mode in TensorFlow backend (Optional)**

   Loading multiple image frames as a single batch and inferring them at once is less expensive on the system than processing all frames one by one. Enabling batch inference for model inference will significantly boost the TensorFlow backend's performance if clubbed with the async mode.

## Phase of Development

This section may keep on changing as the project progresses.

1. Pre-start Refactoring of Code (Done)
2. Asynchronous Inference in TensorFlow backend (Review Phase)
3. Unification of sync and async mode from the filter's perspective.
4. Refactoring in Native Backend for Asynchronous Inference
5. Asynchronous Inference in Native backend
6. Batch Inference in the TensorFlow backend

## Logs for the Project

The daily logs of the GSoC'21 projects can be viewed [here](logs.md). The merged patch list can be viewed using the below links:

- [FFmpeg Git](https://git.ffmpeg.org/gitweb/ffmpeg.git/search?s=Shubhanshu+Saxena;st=author).
- [FFmpeg GitHub Mirror](https://github.com/FFmpeg/FFmpeg/commits?author=shubhanshu02)
- [Intel Media Staging Area](https://github.com/intel-media-ci/ffmpeg/commits?author=shubhanshu02)

## Pull Requests

- [libavfilter: Add Documentation to Queue](https://github.com/intel-media-ci/ffmpeg/pull/389)
- [libavfilter: Add Documentation to SafeQueue](https://github.com/intel-media-ci/ffmpeg/pull/391)
- [lavfi: Add Documentation to Native Backend Layers](https://github.com/intel-media-ci/ffmpeg/pull/396)
- [lavfi/dnn: Async Support for TensorFlow Backend](https://github.com/intel-media-ci/ffmpeg/pull/407)
- [lavfi/dnn: Unification of Async and Sync Modes](https://github.com/intel-media-ci/ffmpeg/pull/423)
- [lavfi/dnn: Async Support for Native Backend](https://github.com/intel-media-ci/ffmpeg/pull/425)
- [lavfi/dnn: Batch Execution in TensorFlow Backend](https://github.com/intel-media-ci/ffmpeg/pull/427)
- [lavfi/dnn: Error Specificity](https://github.com/intel-media-ci/ffmpeg/pull/438)

### Credits

Special thanks to the mentors Yejun Guo and Ting Fu for guiding before and through the project. Gratitude to Shivansh Saini (my senior) for reviewing the proposal.
