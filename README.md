# GSoC'21 Timeline

This repository serves as a log for the weekly progress of my GSoC'21 project. The discussions with the mentors are not logged in this log.

Project: [Async Support for TensorFlow Backend in FFmpeg](https://summerofcode.withgoogle.com/projects/#5224576843251712)

## Community Bonding Period

- **Some days back to 17/05/2021:** Install OpenVino and configure FFmpeg with OpenVino backend. (Look a really long time figuring out what's wrong)
- **18/05/2021:** Study OpenVino and TensorFlow backend in detail for refactoring common code. Revise the GSoC proposal.
- **19/05/2021:** Extract `TaskItem` and `InferenceItem` from OpenVino backend to `dnn_backend_common` and update the OpenVino backend accordingly. Tested on De-rain filter.
- **20/05/2021:** Adjust `TaskItem` according to the TensorFlow backend, so that it can be used in both backends.

  1. Convert `char *output_name` to `char **output_names`.
  2. Add `nb_output` to `TaskItem` (required in TensorFlow backend)
  3. Adjust the OpenVino backend accordingly and test.

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
