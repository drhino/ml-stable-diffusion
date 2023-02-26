## <a name="example-results"></a> Example Results

There are numerous versions of Stable Diffusion available on the [Hugging Face Hub](https://huggingface.co/models?search=stable-diffusion). Here are example results from three of those models:

`--model-version` | [stabilityai/stable-diffusion-2-base](https://huggingface.co/stabilityai/stable-diffusion-2-base) |  [CompVis/stable-diffusion-v1-4](https://huggingface.co/CompVis/stable-diffusion-v1-4) |  [runwayml/stable-diffusion-v1-5](https://huggingface.co/runwayml/stable-diffusion-v1-5) |
:------:|:------:|:------:|:------:
Output | ![](assets/a_high_quality_photo_of_an_astronaut_riding_a_horse_in_space/randomSeed_11_computeUnit_CPU_AND_GPU_modelVersion_stabilityai_stable-diffusion-2-base.png) | ![](assets/a_high_quality_photo_of_an_astronaut_riding_a_horse_in_space/randomSeed_13_computeUnit_CPU_AND_NE_modelVersion_CompVis_stable-diffusion-v1-4.png) | ![](assets/a_high_quality_photo_of_an_astronaut_riding_a_horse_in_space/randomSeed_93_computeUnit_CPU_AND_NE_modelVersion_runwayml_stable-diffusion-v1-5.png)
M1 MacBook Pro 16GB Latency (s) | 24 | 35 | 35 |

## <a name="image-generation-with-python"></a> Image Generation with Python

<details>
  <summary> Click to expand </summary>

Run text-to-image generation using the example Python pipeline based on [diffusers](https://github.com/huggingface/diffusers):

```shell
python -m python_coreml_stable_diffusion.pipeline --prompt "a photo of an astronaut riding a horse on mars" -i <output-mlpackages-directory> -o </path/to/output/image> --compute-unit ALL --seed 93
```
Please refer to the help menu for all available arguments: `python -m python_coreml_stable_diffusion.pipeline -h`. Some notable arguments:

- `-i`: Should point to the `-o` directory from Step 4 of [Converting Models to Core ML](#converting-models-to-coreml) section from above.
- `--model-version`: If you overrode the default model version while converting models to Core ML, you will need to specify the same model version here.
- `--compute-unit`: Note that the most performant compute unit for this particular implementation may differ across different hardware. `CPU_AND_GPU` or `CPU_AND_NE` may be faster than `ALL`. Please refer to the [Performance Benchmark](#performance-benchmark) section for further guidance.
- `--scheduler`: If you would like to experiment with different schedulers, you may specify it here. For available options, please see the help menu. You may also specify a custom number of inference steps by `--num-inference-steps` which defaults to 50.

</details>

## <a name="image-gen-swift"></a> Image Generation with Swift

<details>
  <summary> Click to expand </summary>

### Example CLI Usage
```shell
swift run StableDiffusionSample "a photo of an astronaut riding a horse on mars" --resource-path <output-mlpackages-directory>/Resources/ --seed 93 --output-path </path/to/output/image>
```
The output will be named based on the prompt and random seed:
e.g. `</path/to/output/image>/a_photo_of_an_astronaut_riding_a_horse_on_mars.93.final.png`

Please use the `--help` flag to learn about batched generation and more.

### Example Library Usage

```swift
import StableDiffusion
...
let pipeline = try StableDiffusionPipeline(resourcesAt: resourceURL)
pipeline.loadResources()
let image = try pipeline.generateImages(prompt: prompt, seed: seed).first
```
On iOS, the `reduceMemory` option should be set to `true` when constructing `StableDiffusionPipeline`

### Swift Package Details

This Swift package contains two products:

- `StableDiffusion` library
- `StableDiffusionSample` command-line tool

Both of these products require the Core ML models and tokenization resources to be supplied. When specifying resources via a directory path that directory must contain the following:

- `TextEncoder.mlmodelc` (text embedding model)
- `Unet.mlmodelc` or `UnetChunk1.mlmodelc` & `UnetChunk2.mlmodelc` (denoising autoencoder model)
- `VAEDecoder.mlmodelc` (image decoder model)
- `vocab.json` (tokenizer vocabulary file)
- `merges.text` (merges for byte pair encoding file)

Optionally, for image2image, in-painting, or similar:

- `VAEEncoder.mlmodelc` (image encoder model) 

Optionally, it may also include the safety checker model that some versions of Stable Diffusion include:

- `SafetyChecker.mlmodelc`

Note that the chunked version of Unet is checked for first. Only if it is not present will the full `Unet.mlmodelc` be loaded. Chunking is required for iOS and iPadOS and not necessary for macOS.

</details>

## <a name="swift-app"></a> Example Swift App

<details>
  <summary> Click to expand </summary>

🤗 Hugging Face created an [open-source demo app](https://github.com/huggingface/swift-coreml-diffusers) on top of this library. It's written in native Swift and Swift UI, and runs on macOS, iOS and iPadOS. You can use the code as a starting point for your app, or to see how to integrate this library in your own projects.

Hugging Face has made the app [available in the Mac App Store](https://apps.apple.com/app/diffusers/id1666309574?mt=12).

</details>

##  <a name="faq"></a> FAQ
  
<summary> <b> Q7: </b> How do I generate images with different resolutions using the same Core ML models? </summary>

<b> A7: </b> The current version of `python_coreml_stable_diffusion` does not support single-model multi-resolution out of the box. However, developers may fork this project and leverage the [flexible shapes](https://coremltools.readme.io/docs/flexible-inputs) support from coremltools to extend the `torch2coreml` script by using `coremltools.EnumeratedShapes`. Note that, while the `text_encoder` is agnostic to the image resolution, the inputs and outputs of `vae_decoder` and `unet` models are dependent on the desired image resolution.
</details>
