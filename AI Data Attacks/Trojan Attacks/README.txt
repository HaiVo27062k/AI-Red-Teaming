Technical Implementation: MNIST Backdoor (Trojan) Attack
This document outlines the technical logic and architecture for implementing a targeted trojan backdoor within a Convolutional Neural Network (CNN) using the MNIST digit classification dataset.

Logic and Methodology
1. Trigger Injection Logic
The core of the attack relies on a spatial trigger function that modifies image tensors. The function identifies a specific pixel coordinate range defined by a starting position and a fixed square size. For MNIST grayscale images, the logic targets the single color channel and sets the pixel values within this $3 \times 3$ grid to a maximum intensity value. This modification is performed on the raw tensor range before any standard deviation or mean-based normalization is applied, ensuring the trigger represents a high-contrast feature.

2. Dataset Poisoning and Label Manipulation
The poisoning process is handled by a custom dataset class that wraps the standard training data. The logic follows a three-step sequence:
+ Filtering: The dataset identifies all indices corresponding to the source class (digit 7).
+ Stochastic Selection: A precise percentage (10%) of these identified indices is randomly sampled for corruption.
+ Relabeling: For the sampled indices, the trigger is injected into the image, and the ground-truth label is programmatically overwritten with the target class (digit 1). All other images, including the remaining 90% of the source class, retain their original labels and pixel data.

3. Evaluation Metrics
The implementation uses two distinct metrics to verify the efficacy of the attack:
+ Clean Accuracy (CA): Measured by evaluating the model on a standard, unmodified test set to ensure the backdoor does not degrade the model's primary utility.
+ Attack Success Rate (ASR): Measured by taking all instances of the source class from the test set, applying the trigger to every one of them, and calculating the percentage that the model incorrectly predicts as the target class.

Libraries and Frameworks
PyTorch (torch)
torch.nn: Used to construct the CNN architecture, specifically implementing two-dimensional convolutional layers, max-pooling for spatial downsampling, and linear layers for the final classification head.
torch.optim: Utilized for the Adam optimization algorithm, managing weight updates and weight decay (L2 regularization) during the training phase.
torch.utils.data: Employs the Dataset and DataLoader classes to manage the ingestion of poisoned tensors, batching, and shuffling for the training loop.
Torchvision
torchvision.datasets: Used for the automated retrieval and loading of the MNIST training and testing partitions.
torchvision.transforms: Defines the preprocessing pipeline, converting PIL images to tensors and applying normalization constants (Mean: 0.1307, Std: 0.3081) required for the model to converge effectively.
NumPy and Random
Used for deterministic seeding to ensure reproducibility of the poisoning selection and for the calculation of the sampling indices.
TQDM
Integrated into the dataset initialization and training loops to provide telemetry on the progress of the data poisoning and epoch-by-epoch loss reduction.
