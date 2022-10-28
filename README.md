# How to manage Python packages in SageMaker Studio notebooks
This document present options and recommended practices how to mange Python packages and versions in Amazon SageMaker Studio Notebooks.

You have the following options for installing packages and creating virtual environments in Studio:
1. Use a custom app image
2. Use Studio notebook lifecycle configuration
3. Use Studio's EFS to persist Conda environments
4. Use `pip install` 

Studio notebooks run in a Docker container rather than direct on a EC2 instance, which is the case with SageMaker Notebooks. Because of this difference, there are some specifics how you create and manage virtual environments in Studio notebooks. Refer to the blog post [Activating a Conda environment in your Dockerfile](https://pythonspeed.com/articles/activate-conda-dockerfile/).

Following sections explain each of four options in details.

### Use a custom app image
A SageMaker image or app image is a Docker container that identifies the kernels, language packages, and other dependencies required to run a Jupyter notebook in Studio. You use these images to create an environments that you then run Jupyter notebooks on. Amazon SageMaker provides many [built-in images]((https://docs.aws.amazon.com/sagemaker/latest/dg/notebooks-available-images.html)) for you to use. 

If you need different functionality and packages, you can bring your own custom images to Studio (BYOI). You can create app images and image versions, and attach image versions to your domain, using the SageMaker control panel, the AWS SDK for Python (Boto3), and the AWS Command Line Interface (AWS CLI).

The main benefit is that all packages are ready to use immediately since they are already installed in the image. You can implement a CI/CD pipeline to produce custom images and enforce your organization specific guardrails and governance processes. 

The provided [notebook](notebooks/create-custom-app.ipynb) implements an image creation process for Conda-based environments.

Refer to these [sample notebooks](https://github.com/aws-samples/sagemaker-studio-custom-image-samples/) for more details on custom app implementation.

You can [use Studio image build CLI](https://aws.amazon.com/blogs/machine-learning/using-the-amazon-sagemaker-studio-image-build-cli-to-build-container-images-from-your-studio-notebooks/) to automate process of app image creation and deployment.

### Use Studio notebook lifecycle configuration
Studio [Lifecycle Configurations](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-lcc.html) define a startup script that can install the required packages at each restart of the kernel gateway application.
The main benefit is that a data scientist can choose which script to execute to customize the container with new packages, without rebuilding the container and in most of the cases without requiring a custom image at all as they could customize the [built-in ones](https://docs.aws.amazon.com/sagemaker/latest/dg/notebooks-available-images.html). 
The main limitation is that installing packages at each restart might be slow. It might even timeout and you might need to define a process to let a data scientist customize these scripts. You also have an overhead for managing the lifecycle scripts.

Refer to these [lifecycle configuration examples](https://github.com/aws-samples/sagemaker-studio-lifecycle-config-examples) for more details.

### Use Studio's EFS to persist Conda environments
SageMaker domain and Studio mounts an Amazon EFS volume as a persistent storage layer. You can create Conda environments on this EFS volume. These environments are persistent and Studio automatically picks all environments as Jupyter kernels. 
This is a straightforward process for a data scientist, but there is an about one minute delay for the environment to appear in the list of selectable kernels. There also might be issues with using environments for kernel gateway apps that have different compute requirements, for example CPU-based environment on a GPU-based app.

Refer to this [example](https://github.com/durgasury/efs_backed_conda) for detailed instructions.

### Use `pip install`
You can install packages directly into default conda environment or into the default Python environment. Create a `setup.py` or `requirements.txt` file with all required dependencies and run `%pip install . -r requirement.txt`. You have to run this command every time you restart kernel or re-create an app. This approach is recommended for ad hoc experimentation because these environment are not persistent. Some enterprise environments blocks all egress and ingress internet connections and you cannot use `pip install` to pull Python packages.

## Resources
- [Available Amazon SageMaker Images](https://docs.aws.amazon.com/sagemaker/latest/dg/notebooks-available-images.html)
- [Developer guide for BYOI](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-byoi.html)
- [Custom SageMaker image specifications](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-byoi-specs.html)
- [Conda environments as kernels](https://github.com/aws-samples/sagemaker-studio-custom-image-samples/tree/main/examples/conda-env-kernel-image)
- [Custom app image samples GitHub](https://github.com/aws-samples/sagemaker-studio-custom-image-samples/)
- [How to use Studio Container Build CLI](https://github.com/aws/amazon-sagemaker-examples/tree/main/aws_sagemaker_studio/sagemaker_studio_image_build)
- [Using the Amazon SageMaker Studio Image Build CLI to build container images from your Studio notebooks](https://aws.amazon.com/blogs/machine-learning/using-the-amazon-sagemaker-studio-image-build-cli-to-build-container-images-from-your-studio-notebooks/)
- [Create a new Conda environment in Sagemaker home directory (Amazon EFS)](https://github.com/durgasury/efs_backed_conda)
- [Automating the Setup of SageMaker Studio Custom Images](https://towardsdatascience.com/automating-the-setup-of-sagemaker-studio-custom-images-4a3433fd7148)
- [Activating a Conda environment in your Dockerfile](https://pythonspeed.com/articles/activate-conda-dockerfile/)
- [Customize Amazon SageMaker Studio using Lifecycle Configurations](https://aws.amazon.com/blogs/machine-learning/customize-amazon-sagemaker-studio-using-lifecycle-configurations/)

## QR code for this repository
You can use the following QR code to link this repository.

![](img/github-repo-qrcode.png)

Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
SPDX-License-Identifier: MIT-0