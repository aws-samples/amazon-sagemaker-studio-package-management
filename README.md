# How to manage Python packages in Amazon SageMaker Studio notebooks
This repository presents hands-on samples for the recommended practices on how to manage Python packages and package versions in Amazon SageMaker Studio Notebooks.

You have the following options for installing packages and creating virtual environments in Studio:
1. Use a SageMaker custom app image
2. Use Studio notebook lifecycle configurations
3. Use Studio's EFS to persist Conda environments
4. Use `pip install` 

Studio notebooks [run in a Docker container](https://aws.amazon.com/blogs/machine-learning/dive-deep-into-amazon-sagemaker-studio-notebook-architecture/), while SageMaker Notebook instances are hosted on EC2 instances. Because of this difference, there are some specifics how you create and manage Python virtual environments in Studio notebooks, for example usage of Conda environments or persistence of ML development environments between kernel restarts.

Conda doesn't work well within Docker container, for example see the blog post [Activating a Conda environment in your Dockerfile](https://pythonspeed.com/articles/activate-conda-dockerfile/).

Following sections give run down of each of four recommended package management options.

## How to run the notebooks
- `create-custom-app.ipynb`: run in a [SageMaker Notebook Instance](https://docs.aws.amazon.com/sagemaker/latest/dg/nbi.html), not in Studio
- `persistent-conda-environments-on-efs.ipynb`: run in Studio
- `studio-notebook-lifecycle-configurations.ipynb`: run in Studio

### SageMaker custom app image
A SageMaker image or app image is a Docker container that identifies the kernels, language packages, and other dependencies required to run a Jupyter notebook in Studio. You use these images to create environments that you then run Jupyter notebooks on. Amazon SageMaker provides many [built-in images](https://docs.aws.amazon.com/sagemaker/latest/dg/notebooks-available-images.html) for you to use. 

If you need different functionality and packages, you can bring your own [custom images](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-byoi.html) to Studio (BYOI). You can create app images and image versions, and attach image versions to your domain, using the SageMaker control panel, the [AWS SDK for Python](https://aws.amazon.com/sdk-for-python/) (Boto3), and the [AWS Command Line Interface](https://aws.amazon.com/cli/) (AWS CLI).

The main benefit is that all packages are ready to use immediately since they are already installed in the image. You can implement a CI/CD pipeline to produce custom images and enforce your organization specific guardrails and governance processes. 

The provided [notebook](notebooks/create-custom-app.ipynb) implements an image creation process for Conda-based environments.

Refer to these [sample notebooks](https://github.com/aws-samples/sagemaker-studio-custom-image-samples/) for more details on custom app implementation.

You can [use Studio image build CLI](https://aws.amazon.com/blogs/machine-learning/using-the-amazon-sagemaker-studio-image-build-cli-to-build-container-images-from-your-studio-notebooks/) to automate process of app image creation and deployment.

### Studio notebook lifecycle configurations
Studio [Lifecycle Configurations](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-lcc.html) define a startup script that executed at each restart of the kernel gateway application and can install the required packages.
The main benefit is that a data scientist can choose which script to execute to customize the notebook container with new packages, without rebuilding the container and in most of the cases without requiring a custom image at all as they could customize the [built-in ones](https://docs.aws.amazon.com/sagemaker/latest/dg/notebooks-available-images.html). 
The main limitation is that installing packages at each restart might be slow. It might even timeout and you need to define a process to let a data scientist customize these scripts. You also have an overhead for managing the lifecycle scripts at scale.

Refer to these [lifecycle configuration examples](https://github.com/aws-samples/sagemaker-studio-lifecycle-config-examples) for more details.

### Persist Conda environments to Studio's EFS
SageMaker domain and Studio use an Amazon EFS volume as a persistent storage layer. You can save your Conda environments on the EFS volume. These environments are persistent between kernel, app, or Studio restarts. Studio automatically picks up all environments as KernelGateway kernels. 
This is a straightforward process for a data scientist, but there is a material (about one minute) delay for the environment to appear in the list of selectable kernels. There also might be issues with using environments for kernel gateway apps that have different compute requirements, for example CPU-based environment on a GPU-based app.

Refer to this [example](https://github.com/durgasury/efs_backed_conda) for detailed instructions.

### `pip install`
You can install packages directly into default conda environment or into the default Python environment. Create a `setup.py` or `requirements.txt` file with all required dependencies and run `%pip install . -r requirement.txt`. You have to run this command every time you restart kernel or re-create an app. This approach is recommended for ad hoc experimentation because these environments are not persistent. Some enterprise environments block all egress and ingress internet connections and you cannot use `pip install` to pull Python packages.

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
- [AWS Batch Architecture for RFDesign](https://github.com/aws-samples/aws-batch-architecture-for-rfdesign)

## QR code for this repository
You can use the following QR code to link this [repository](https://github.com/aws-samples/amazon-sagemaker-studio-package-management).

![](img/github-repo-qrcode.png)

Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
SPDX-License-Identifier: MIT-0