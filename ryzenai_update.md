# RyzenAi-SW 1.1

All of the instructions are to be run from a powershell terminal in the riallto_venv Python virtual environment.

## Uninstall previous ONNX Runtime installation

To make sure that the previous installation doesn't interfere with the new packages use the Riallto utility script `uninstall_onnx.ps1` to purge the RyzenAI-SW 1.0 files from your Python environment. **Make sure you sourced the riallto venv first!**

```
activate-venv.ps1
uninstall_onnx.ps1
```

## Download RyzenAI-SW-1.1

To upgrade to the latest RyzenAI-SW provided execution provider for ONNX Runtime you will have to download the official [RyzenAI-SW-1.1 package](https://account.amd.com/en/forms/downloads/ryzen-ai-software-platform-xef.html?filename=ryzen-ai-sw-1.1.zip). Once downloaded, extract and navigate to the `ryzen-ai-sw-1.1` directory.

```
Expand-Archive .\ryzen-ai-sw-1.1.zip
cd .\ryzen-ai-sw-1.1\ryzen-ai-sw-1.1\
```

## Install Python packages

We install the additions to onnxruntime, the voe (VitisAI onnxruntime execution provider) and quantizer packages.

```
py -m pip install .\voe-4.0-win_amd64\onnxruntime_vitisai-1.15.1-cp39-cp39-win_amd64.whl
py -m pip install .\voe-4.0-win_amd64\voe-0.1.0-cp39-cp39-win_amd64.whl
py -m pip install .\vai_q_onnx-1.16.0+69bc4f2-py2.py3-none-any.whl

```

The runtime execution provider relies on C APIs that bind to dlls delivered by the AMD NPU driver, these need to be copied to the onnxruntime site-packages directory.

```
$site_dir = (py -m pip show pip | Select-String "Location:").Line.Split(" ")[1]
Copy-Item C:\Windows\System32\AMD\xrt_core.dll $site_dir\onnxruntime\capi\
Copy-Item C:\Windows\System32\AMD\xrt_coreutil.dll $site_dir\onnxruntime\capi\
Copy-Item C:\Windows\System32\AMD\amd_xrt_core.dll $site_dir\onnxruntime\capi\
Copy-Item C:\Windows\System32\AMD\xdp_ml_timeline_plugin.dll $site_dir\onnxruntime\capi\
Copy-Item C:\Windows\System32\AMD\xdp_core.dll $site_dir\onnxruntime\capi\
```

## Verify installation

There's a quicktest script you can run in ryzen-ai-sw-1.1 doing the following:
```
cd quicktest
py quicktest.py
```

Some of the text might look garbled in powershell, however you should be able to spot the lines:

```
[Vitis AI EP] No. of Operators :   CPU     2    IPU   398  99.50%
...
Test Passed
```

This will quickly confirm that you are able to interact with the NPU using onnxruntime with the VOE EP.

## Riallto notebooks

The previous 5_1 and 5_2 notebooks should work as before, make sure to delete `modelcachekey` and `resnet.qdq.U8S8.onnx` before running them since these are compilation outputs cached from the previous revision of the tools.
