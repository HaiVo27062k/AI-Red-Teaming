Clean Label Attack Project

Purpose
This project is a learning exercise on clean-label data poisoning attacks. The notebook demonstrates how an attacker can modify selected training samples without changing their labels, retrain a classifier, and cause one specific target sample to be misclassified.

The current implementation attacks a 3-class logistic regression model trained with a one-vs-rest strategy. It is not directly attacking an LLM. However, the same security concept is relevant to larger AI systems, including LLM-based pipelines, because poisoning correctly labeled but strategically crafted training data can influence downstream model behavior.

Definition of a Clean-Label Attack
A clean-label attack is a type of data poisoning attack in which the attacker injects or modifies training examples while keeping the labels unchanged and seemingly correct. The poisoned examples look legitimate to a human reviewer, but they are placed in feature space so that training on them nudges the model's decision boundary in a way that benefits the attacker.

In this notebook, the goal is targeted misclassification:
- keep the poisoned samples labeled as their original class
- move those samples in feature space toward a chosen boundary
- retrain the model
- cause one specific target point to be predicted as the attacker's chosen class

Project Logic
The notebook follows a clear pipeline from data loading to attack evaluation and submission.

1. Setup and hyperparameters
The first cell imports the required libraries, sets a fixed random seed, defines plotting colors, and sets two main attack hyperparameters:
- N_NEIGHBORS: how many training points from the perturbing class will be modified
- EPSILON_CROSS: how far each selected point is moved in feature space

2. Load the dataset and identify the attack scenario
The second cell loads the training and test sets from the NPZ file and reads the target index. It then derives:
- TARGET_CLASS: the true label of the target sample
- PERTURBING_CLASS: the class whose samples will be moved
- MISCLASSIFY_AS_CLASS: the class the target should be predicted as after poisoning

This cell also extracts the target feature vector for later evaluation.

3. Visualization helpers
The plotting helper functions are defined next. These functions are used to:
- plot the 2D dataset by class
- highlight the target point and poisoned neighbors
- draw decision regions for the trained classifier

These plots are important because the attack is geometric. The notebook is showing how the poisoned points shift relative to the model's boundaries.

4. Visualize the clean dataset
Before attacking anything, the notebook plots the original training data and highlights the target point. This gives a baseline picture of where the target sits relative to the other classes.

5. Train the baseline model
The notebook trains a one-vs-rest logistic regression classifier on the clean training data. It then:
- evaluates baseline accuracy on the clean test set
- builds a mesh grid for plotting decision regions
- predicts the baseline decision regions across the grid
- extracts the learned weight vectors and intercepts from each binary classifier

Those extracted parameters are important because they are later used to estimate the direction in which poisoned points should be pushed.

6. Plot the baseline decision boundary
The notebook visualizes the baseline model's decision regions. This helps verify where the target lies before poisoning and how the class regions are arranged in feature space.

7. Perform the clean-label attack
The main attack function does the following:
- selects all training samples belonging to the perturbing class
- finds the nearest neighbors from that class to the target point
- computes a boundary vector using the baseline one-vs-rest classifiers for the perturbing class and the target class
- converts that boundary vector into a normalized push direction
- multiplies the direction by EPSILON_CROSS to create a perturbation vector
- adds the perturbation vector to the selected neighbors' features
- keeps the original labels unchanged

This is the core clean-label idea. The labels stay clean, but the model sees feature vectors that have been repositioned to influence the learned boundary.

8. Train a new model on poisoned data
After generating the poisoned training set, the notebook retrains the same one-vs-rest logistic regression model using the modified features and unchanged labels.

9. Evaluate whether the attack succeeded
The notebook then checks:
- the prediction for the target point under the poisoned model
- whether the prediction matches the required misclassification class
- the overall accuracy of the poisoned model on the clean test set

This matters because a strong targeted poisoning attack should flip the target while preserving acceptable overall model performance.

10. Visualize poisoned data and poisoned decision regions
The notebook plots:
- the poisoned training data with highlighted perturbed points
- the poisoned model's decision regions

These plots make the attack interpretable by showing how a relatively small set of moved points can alter the local geometry around the target.

11. Extract parameters and submit to the evaluator
If the attack succeeds, the notebook extracts the weight vectors and intercepts from the poisoned one-vs-rest model and submits them to the remote evaluator service. The evaluator performs a health check and then scores whether the submitted model meets the lab's success criteria.

Why this matters for LLM security
This notebook is built around a classical classifier, but the lesson generalizes:
- poisoned training examples do not need visibly incorrect labels to be dangerous
- targeted behavior can be induced through data manipulation rather than test-time prompt manipulation
- systems that rely on embeddings, retrieval, fine-tuning, or downstream classifiers can inherit poisoning risk from their training data

In other words, clean-label attacks are part of the broader problem of training-data integrity in machine learning and AI systems, including LLM-adjacent systems.

Summary
This project shows a targeted clean-label poisoning workflow end to end:
- train a baseline classifier
- identify a target example
- select nearby samples from a perturbing class
- move them across a useful direction in feature space without changing their labels
- retrain the model
- verify whether the target is now misclassified

The main educational value of the notebook is that it makes the poisoning process visible, geometric, and measurable.
