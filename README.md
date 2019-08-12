
# Convert and Compare Torch, ONNX, and CoreML Resnet50  
The notebooks in this folder are code for, and notes about, converting a "standard" **PyTorch** **Resnet50** model to **CoreML**.  Because there is no direct conversion from Pytorch to CoreML,  ONNX is used as an intermediate format.  The conversion flow is:

    Torch->ONNX,  ONNX->CoreML,  then CoreML->iOS using XCode

This was primarily a learning exercise, and secondarily an experiment to investigate if an off-the-shelf  *Torch Resnet50* - e.g. from the Python *torchvision*  distribution - would convert successfully (... The answer is: no, it does not - but for now I'm assuming that the root cause is me, and that I need to learn more.). 

In addition to demonstrating the conversion process, and CoreML model repair where necessary, these notebooks show and compare predictions generated by the original models and their converted descendants.

### Why?

I hear you ask ...

I tried to port my own ResNet50 model from Torch(Fastai) to CoreML. Unsuccessfully.

As part of investigating how and why that conversion went astray, I thought it might be enlightening to convert a "standard" Pytorch Resnet50 model to CoreML, and see what happened.   I also included a "standard" ONNX Resnet50 model, because as long as I was doing this, why not? ... I wanted to see how its predictions would compare to the others.



### Overview
Here is the general approach:
- Obtain the intial,off-the-shelf model (By downloading from web or loading from a package).
- Sanity-check that I've set up the model correctly by generating a few predictions and confirming that they are correct.
- Convert,repair when necessary, and verify results until we generate a working CoreML model.
- Compare the prediction results of the original model(s) to those of the converted descendants.

Test images were selected randomily from a collection of dogs, cats, plants and food images that reside on my local machine.

### Summary of Conversion Results

*Torch->Onnx->CoreML*

Conversion of the Torch Resnet50 model `torchvision.models.resnet50(pretrained=True)` to  ONNX was successful. The conversion of that ONNX model to CoreML failed. The *ONNX->CoreML* step created a CoreML model that raised shape-mismatch errors when run. See [20-convert-torch-to-coreml][20-convert].

To get the faulty CoreML model to work, I edited the problematic layers (using `coremltools` and `coreml_help`) and re-verified in Python.

*Onnx->CoreML*

As part of investigating the error above, I attempted to convert an off-the-shelf ONNX Resnet50 to CoreML (see [30-convert-onnx-to-coreml][30-convert] ).  That conversion was successful.  One interpretation of such a result is that  the *Torch -> ONNX* conversion generates an ONNX model that is either not entirely correct, or not handled by the *ONNX->CoreML* conversion tools, or both.  However,  *more likely*  is that I introduced the error thru misunderstanding some part of the conversion procedure.  I'm still looking into this.

[20-convert]:20-convert-torch-to-coreml.ipynb
[30-convert]:30-convert-onnx-to-coreml.ipynb

### Summary of Comparision Results
After conversion and repair, I compared predictions from the CoreML-converted-from-Torch Resnet50 model to predictions generated by the originals, the intermediate models, and to predictions from "off-the-shelf" ONNX and CoreML Resnet50 models. See [40-Compare-torch-onnx-coreml](40-Compare-torch-onnx-coreml.ipynb).

**95% Agreement - the "Torch Family"** - The predictions generated by the original Torch model, the intermediate ONNX model dervied from it, and the final (after repair) CoreML model - call these three together the *Torch Family*  - were very consistent. About 95% or more of the predictions agreed.

*60% Agreement - the "ONNX Family"* - The "Model Zoo" ONNX model and its CoreML descendant - call these two the *"ONNX Family"* - generated modest agreement. A bit surprising that it was not higher.

*53% Agreement* - between *Torch* and *ONNX* and *CoreML*


#### Conclusion
  * Any conversion from Torch to CoreML will probably require some amount of CoreML "model surgery" followed by comprehensive testing.
  * The converted model should behave very much like the Torch model, which is good news.


### Notebooks
**[10-Intro-resnet50-convert-compare](10-Intro-resnet50-convert-compare.ipynb)**

This notebook.

**[20-convert-torch-to-coreml](20-convert-torch-to-coreml.ipynb)**

Load the packaged Torch Resnet50 (`torchvision.models.resnet50(pretrained=True)`), convert to ONNX, then convert to CoreML, checking at each step that the models work.  The first conversion,*Torch->ONNX*, succeeds, but the second one, *ONNX->CoreML*, fails. Surgery is required to fix the CoreML model. Surgery is attempted, patient survives, and the CoreML model successfully validated.  A brief analysis of agreement between the models is followed by a display of images and the resulting predictions

**[30-convert-onnx-to-coreml](30-convert-onnx-to-coreml.ipynb)**

The notebook converts an off-the-shelf ONNX Resnet50 to CoreML.  Successfully. 

The initial attempt at converting from ONNX->CoreML failed. Is this because of a fault in the ONNX->CoreML conversion tools, or is it  because the original model started life as a Torch model? This notebook is an experiment to investigate.

This conversion was successful. Which could mean something about the two-step (Torch->ONNX, ONNX->CoreML) conversion process is breaking. Or that I don't know enough yet.

**[40-compare-torch-onnx-coreml](40-compare-torch-onnx-coreml.ipynb)**

This notebook compares the predictions of six Resnet50 Models. 
It loads the original, intermediate and final models created by the conversions described previously. 

The models are named:   
```
 torchm Torch model from the torchvision distribution.  
 t2o    An ONNX Model. The result of converting "torchm" to ONNX.  
 t2c    A CoreML Model. The result of converting the intermediate model "t2o" to CoreML.  
 onnxm  The ONNX Resnet50 model from the ONNX Model Zoo on Github.  
 o2c    A CoreML Model. The result of converting "onnxm" to CoreML.  
 cml    A CoreML Model. For comparison, I added the "native" CoreML Resnet50 model.  
```
Agreements between the predictions of all models are tabulated and presented in an "Agreement Matrix". 
A selection of images and the predictions made for them by each model is shown.


### References
#### Tools and Packages

- **[Netron](https://github.com/lutzroeder/netron)** for examining models
- [torch, torchvision](https://onnx.ai)
- [onnx](https://onnx.ai) for conversion
- [onnx_coreml](https://github.com/onnx/onnx-coreml) to convert to CoreML
- [onnxruntime](https://microsoft.github.io/onnxruntime/) for verification and debugging.
- [coremltools](https://github.com/apple/coremltools) for verification and debugging and model editing
- [coreml_help](https://github.com/mcsieber/coreml_help) (minor) Python helper tools that I wrote. A convenience. Not required.
- [pred_help](https://github.com/mcsieber/coreml_help) (minor) More python helper tools. Not required.
- [XCode 10](https://developer.apple.com/xcode/) to include the model in an iOS app and load the app onto my iPhone

#### Helpful Books, Articles, Tutorials etc ...

- [**Core ML Survival Guide**](https://leanpub.com/coreml-survival-guide) by [*Matthijs Hollemans*](https://github.com/hollance)  for real help with CoreML
- [How I Shipped a Neural Network on iOS with CoreML, PyTorch, and React Native](https://attardi.org/pytorch-and-coreml)
- [raywenderlich.com](https://www.raywenderlich.com) for iOS sample apps and tutorials


```python

```
