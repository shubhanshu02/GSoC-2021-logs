# Daily Logs

This page contains the daily logs for the project work related to the project.

## Community Bonding Period

- **Some days back to 17/05/2021:** Install OpenVINO and configure FFmpeg with OpenVINO backend. (Look a really long time figuring out what's wrong)
- **18/05/2021:** Study OpenVINO and TensorFlow backend in detail for refactoring common code. Revise the GSoC proposal.
- **19/05/2021:** Extract `TaskItem` and `InferenceItem` from OpenVINO backend to `dnn_backend_common` and update the OpenVINO backend accordingly. Tested on De-rain filter.
- **20/05/2021:** Adjust `TaskItem` according to the TensorFlow backend, so that it can be used in both backends.

  1. Convert `char *output_name` to `char **output_names`.
  2. Add `nb_output` to `TaskItem` (required in TensorFlow backend)
  3. Adjust the OpenVINO backend accordingly and test.

- **21/05/2021:** Convert TensorFlow backend to `TaskItem` based inference and create a pull request [#407](https://github.com/intel-media-ci/ffmpeg/pull/407/) for the older changes.

- **22/05/2021:** Add `RequestItem` to the TensorFlow backend.

  1. Create and add `tf_infer_request` to `RequestItem` where `tf_infer_request` contains the execution parameters for the current request.
  2. Refactor the existing code to separate filling of `RequestItem` and its completion callback.
  3. Change the execution mechanism to Request-Task based inference.
  4. Test on superresolution and derain filters. Push to PR branch.

- **23/05/2021:** As from the discussion earlier:

  1. Change the execution to use `InferenceItem` for `RequestItem` execution instead of `TaskItem`.
  2. Study the memory allocation again and look for which instances to free and when.
  3. Correct pointer type in OV backend while freeing.
  4. Test on superresolution and derain filters. Push to PR branch.
  5. After the changes, the execution time for sync mode speed up from 3m14.318s to 2m35.280s (for a video with TensorFlow C API with CPU support only)

- **24/05/2021:** Following things:

  1. Revise threading from articles and book.
  2. Change the `do_ioproc` and `async` flags from `int` to `uint8_t`.
  3. Try extracting common part from `get_output_<backend>`. (After trying, didn't seem useful to extract)
  4. Study Native backend for similar changes in it.

- **25/05/2021:** Start work for asynchronous mode in TensorFlow backend.

  1. Start with adding thread id and attributes to the `RequestItem` (one thread per `RequestItem`). Init with detached attributes.
  2. Add the start routine for the thread. (for now, call the completion callback from this function)
  3. Add async option in `execute_model_tf`. Add function for getting resultant frame in async mode.
  4. Test on superresolution and derain filters. Result: <span style="color:red">Fail! Segmentation Fault.</span>

- **26/05/2021:** Complete work for async mode in TensorFlow backend.

  1. Check the source of segmentation fault. Source: Flush frame function does not exist.
  2. Add flush frame function to TensorFlow backend
  3. Test. Result: <span style="color:green">Working!</span>
  4. Finalize changes, rebase to PR branch and push.
  5. Document functions related to `tf_infer_request`

- **27/05/2021:** Discussions for further refactoring of code with mentor's idea of unifying async and sync mode in the backends.

  1. Corrections from code review of the PR.
  2. Add handling of cases where POSIX threads isn't supported.
  3. Further refactor code for async execution in TensorFlow backend.
  4. Tests? `make fate`? Result: <span style="color:green">Passed!</span>

- **28/05/2021 and 29/05/2021:** Study the DNN module for unification of async and sync modes and discussions over email.

- **30/05/2021 and 31/05/2021:** Two days break from development work. Plan further changes for unification.

- **01/06/2021:**

  1. Unify functions for async and sync inference.
  2. Temporarily disable inference in all three backends.
  3. Add `async` flag to `TFOptions` to be fetched from `backend_configs`.
  4. Unify execution functions in TensorFlow backend and enable its execution.

- **02/06/2021:** Unify functions in OpenVINO backend and enable its execution from the DNN interface.

- **03/06/2021:** Study native backend and plan the changes for unification.

- **04/06/2021:** Add Task-Inference-based mechanism to Native Backend and enable its execution from the DNN interface.

- **04/06/2021:** Add Task-Inference-based mechanism to Native Backend and enable its execution from the DNN interface.

- **06/06/2021:** Divide the large async patch to 2-3 patches of smaller size.

## Week 1

- **08/06/2021:** Cleanup after unification of sync and async mode. Remove async from DNNContext and edit documentation as required.

- **09/06/2021-10/06/2021:** Study native backend for RequestItem based inference to make async inference mechanism.

- **11/06/2021-12/06/2021:** Plan async inference mechanism in Native Backend and discuss with the mentor over email. The two methods for inference mechanisms were as follows:

  #### **Method 1**: On the existing inference mechanism, use the same DnnOperand from `native_model->operands` for every `RequestItem`.

  1. Acquire mutex lock
  2. Fill the input operand with the frame data and run all the layers' execution functions.
  3. Fill the output frame in `TaskItem` with the data from the output operand.
  4. Release the lock

  **Pros**: Low memory usage as compared to method 2.

  **Cons**: Single `RequestItem` is being executed at a time. All other threads are waiting for the lock.

  #### **Method 2**: This method creates new `DnnOperand` items for the execution as follows.

  1. Create a copy of `native_model->operands` and store it in the current `RequestItem`.
  2. Fill its input operand with the frame data and run all the layers' execution functions using the newly created operands.
  3. Fill the output frame in `TaskItem` with the data from the output operand.

  **Pros**: Multiple `RequestItem` instances can execute at the same time, so faster execution.

  **Cons**: More memory and CPU utilization. May lead to less responsiveness in the system (since already many joinable threads are executing in the Convolution layer)

- 5 Patches Merged

  - [lavfi/dnn: Extract TaskItem and InferenceItem from OpenVINO Backend](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/f5ab8905fddee7a772998058e8cf18f93649fc5a)

  - [lavfi/dnn: Convert output_name to char\*\* in TaskItem](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/446b4f77c106add0f6db4c0ffad1642d0920d6aa)

  - [lavfi/dnn: Add nb_output to TaskItem](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/9675ebbb91891c826eeef065fd8a87d732f73ed0)

  - [lavfi/dnn: Use uint8_t for async and do_ioproc in TaskItems](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/6b961f74096aff114d32480670943ce4d6d66826)

  - [lavfi/dnn: Fill Task using Common Function](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/55092358189b98682d133c7b05bfcbb7ab6c750f)

- **13/06/2021:** Refactor `execute_model_native` to two functions - one for filling operand with model input (from the `input_frame`) and one for completion callback to extract the `output_frame` from the DNN data.

## Week 2

- **14/06/2021:** Identified a memory leak in OpenVINO backend and discussions for Native backend new operand method. Add `copy_operands` function to copy Operands in Native Backend for use in `RequestItem` objects.

- **15/06/2021:** Start work for Native backend async.

  - Add `nireq` and `RequestItem` to Native backend. `RequestItem` contains a copy of Operands from the model and POSIX threads elements like id and attributes.
  - Use `request->operands` instead of `native_model->operands`.

- **16/06/2021:** Refactor the execution function to two functions - one for filling operands with frame data and other for completion callback.

- **17/06/2021:** Complete `RequestItem` based execution in `execute_model_native` and create functions for asynchronous execution of `RequestItem` in the Native Backend.

- **18/06/2021:** Continue Async Mode in Native Backend and Bug Fixes.

  1. Write functions and thread routine for async execution in Native Backend.
  2. Review TensorFlow backend asynchronous execution patchset again. Found some memory leaks. <span style="color:green">Fixed!</span>
  3. Identified a memory leak in OpenVINO backend. In case of failures in sync mode, the `RequestItem` should be pushed back to the `ov_model->request_queue` or the memory held should be released.

- **19/06/2021:** Documentation for newly created functions in Native Backend.

- **20/06/2021:** Continue async support in Native Backend.

- **21/06/2021:** Async Support for TensorFlow Backend - Apply suggestions from code review.

- **22/06/2021:** Try async support for TensorFlow Backend using different sessions (`TF_Session`) for each request.

- **23/06/2021:**
  1. Test speed of inference in the work done on previous day as compared to sync mode and current async implementation.
  2. Native backend Convolution layer thread number discussion.
