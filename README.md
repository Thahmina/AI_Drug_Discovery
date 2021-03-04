# Accelerating small molecule (drug) discovery and synthesis

Table of Contents
-----------------
- [Objective](#objective)
- [Motivation](#motivation)
- [Data](#data)
- [Approach](#approach)
  1. [Preprocess Data](#preprocessdata) 
  2. [Training](#training) 
  3. [Inspirational Adverserial Image Generation](#inspirationalgeneration) 
- [Potential Results](#results)


## <a name="objective"></a> Objective-
Facilitate the development of a useful application for the automation of small molecule (drug) discovery and synthesis by using an AI model Generative adversarial nets (GAN) that will be trained to generate molecular structures.

## <a name="motivation"></a> Motivation-

The field of synthetic chemistry has greatly enhanced human health generating thousands of new medicines. Yet the laboratory methods used to synthesize medicinal compounds are still highly manual, frustratingly slow, and difficult to reproduce and scale. 

There is a continued need for development of new drugs,  thus using state-of-the-art tools such as Artificial Intelligence (AI) will accelerate the potential to make any molecule at will, inexpensively and on a meaningful timescale. This will be especially critical for the future in discovering the possibilities of synthesizing  the next generation of life-enhancing  molecules.

## <a name="data"></a> Data

#### SMILES (Simplified Molecular Input Line Entry System) 
---
Chemical notation that allows a user to represent a chemical structure in a way that can be used by the computer.

**Example:**

The molecular formula for Ethene (CH2CH2)  contains a double bond which will be denoted in SMILES with = symbol, notated as C=C.


#### PubChem:  
---

Download: 

Unfiltered SMILES of every chemical compund (CID) from PubChem can be retrieved: ftp://ftp.ncbi.nlm.nih.gov/pubchem/Compound/Extras/CID-SMILES.gz (uncompressed data is ~7GB)
 
PubChemPy:
A simple Python wrapper around the PubChem PUG REST API: https://pypi.org/project/PubChemPy/

## <a name="approach"></a> Approach-

Progressive GAN’s (PGAN) pytorch implementation by Facebook research (pytorch_GAN_zoo) will be used to train the model:

#### Preprocess Data:

1) Obtain chemical compound data in SMILES notation (PubChem database).
 
2) Classify the SMILES notation data into functional groups (python library rdkit).

3) Convert the classified data into molecular structure images of size 128x128 (python library rdkit).

#### Training:


1) Obtain the pytorch_GAN_zoo toolkit:

```

git clone https://github.com/facebookresearch/pytorch_GAN_zoo.git
```

2) Resize the training data (smiles_structure) into low resolution images with the following command:

```

python datasets.py smiles_structure $PATH_TO_IMAGES -o $OUTPUT_DIR -f
```

`datasets.py` will resize images in steps of (64, 128, 512, 1024) until the image size of the data. Scale is a concept that is unique to PGAN and every scale is associated with the number of layers in the model, iterations and image resolution. The max_scale is calculated using following formula, constant 2 is added to max_scale because the training layers start from resolution of 4x4:

image_size = 2**(2+max_scale)

A config file gets generated will contain the path to the resized images and the original images and number of iterations per scale. 

`$PATH_TO_IMAGES` is the location of the training images.
 
`$OUTPUT_DIR`  this is the location where resized images will be saved. 

`-f`  generate resized images before training starts. 

3) Train the model with the following command:

```

python train.py PGAN -c $CONFIG_FILE -n $DATASET_NAME -d $WEIGHTS_DIR
```

`PGAN` refers to Progressive GAN 

`$CONFIG_FILE`  path to config file generated by datasets.py. 

`$DATASET_NAME`   name of custom dataset

`$WEIGHTS_DIR`  location where weights will be stored. 

Note: Google provides $300 dollars of credit and 12 months as a free tier user of their cloud platform. Cloud computing instances such as the Nvidia Tesla K80 GPU can be set up.

4) Evaluate the model:

There are several methods pytorch_GAN_zoo provides tools to evaluate the performance of the model:

i) Inception score: 

```

python eval.py inception -c $CONFIGURATION_FILE -n $modelName -m $modelType -d $WEIGHTS_DIR
```

ii) Sliced Wasserstein distance (SWD):

```

python eval.py laplacian_SWD -c $CONFIGURATION_FILE -n $modelName -m $modelType -d $WEIGHTS_DIR
```

#### Inspirational Adverserial Image generation:  Image generation "inspired" from a reference image using an already trained GAN

This tool uses gradient descent and extracts input vectors on the input image which will be used to generate new images that share characteristics of the input image. 

1) Build a feature extractor using the vgg16/vgg19 model to be used for input vector generation:

```

python save_feature_extractor.py {vgg16, vgg19}\   $PATH_TO_THE_OUTPUT_FEATURE_EXTRACTOR --layers 3 4 5
```

2) Run the model which will generate molecule structure:

```

python eval.py inspirational_generation -n $modelName -m $modelType\ --inputImage $pathTotheInputImage -f \ $PATH_TO_THE_OUTPUT_FEATURE_EXTRACTOR -d $WEIGHTS_DIR
```

## <a name="results"></a> Potential Results-

The expected outcomes include images of predicted chemical structure of compounds. Below are examples.

<img src="https://github.com/Thahmina/AI_drug_discovery/images/chemstruc_examples.png"  />


The overall goal would be to deploy the model as an application via Heroku that can be offered as a tool. The tool is designed to predict structures of chemical compunds tailored for medicinal or drug activity that will allow the ability to efficiently allocate the necessary resources and time required in the discovery and syntheis process. 

