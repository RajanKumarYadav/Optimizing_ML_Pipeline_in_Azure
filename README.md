# Creating & Optimizing an ML Pipeline in Azure using Azure Auto ML and Azure Hyperdrive

## Overview
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary
The dataset contains data about individuals applying for the bank loans. Main aim of the project is to build Machine Learning Model, based on historical data of customer profiling about each person, And the outcomes of the project will be prediction of the customer who will use or subscribe to a specific bank service.

The best performing model was voting ensemble with 91.8% accuracy having Max iterations 23 and Regularization Strength 0.62.
 

## Scikit-learn Pipeline

**Explain the pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**

The Scikit-learn pipeline pull data from the provided URL. Following data download, a number of data cleaning steps are carried out including:

Handled NULL values from the original dataset.
Applied One-hot encoding on categorical features like contact, education variables.
Applied feature preprocessing on months & weekdays.
Applied Integer Encoding on target variable.

After preprocessing steps we split the data into a training and test set in our case i took test set size of 40% of total data for model training(You can find code in train.py).

As the main goal of the project was classification.So we used classification algorithm logistic regression, which gives pretty good results in binary classification problems solutions.
Logistic Regression is used when the dependent variable(target) is categorical.
For example,
To predict whether an email is spam (1) or (0)
Whether the tumor is malignant (1) or not (0)


Hyperdrive service from Azure ML is used for hyperparameter optimization.Hyperparameter optimization, is the process of finding the configuration of hyperparameters that results in the best performance. The process is typically computationally expensive and manual so we used Hyperdrive.

In order to complete the steps required to tune hyperparameters with the Azure Machine Learning SDK we followed below steps:

Define the parameter search space
Specify a primary metric to optimize
Specify early termination policy for low-performing runs
Allocate resources
Launch an experiment with the defined configuration
Visualize the training runs
Select the best configuration for your model


Parameter sampler

I used RandomParameterSampling as parameter sampler:

ps = RandomParameterSampling(
    {
        "C": uniform(0, 10),
        "max_iter": quniform(20, 180, 1)
    }
)

C: is the inverse of the regularization term (1/lambda). It tells the model how much large parameters are penalized, smaller values result in larger penalization; must be a positive float.Though a higher C will cause the model to misclassify less, but is much more likely to cause overfit.
Common values for C is: [0.001,0.1 …10..100]
max_iter is Maximum number of iterations taken for the solvers to converge in logistic regression.

##Early stopping policy

In this project i used BanditPolicy(evaluation_interval=2, slack_factor=0.1) that defines an early termination policy based on slack criteria, and a frequency and delay interval for evaluation.This is optional and represents the frequency for applying the policy. Each time the training script logs the primary metric counts as one interval.

Benefits of the chosen early stopping policy: Too little training will mean that the model will underfit the train and the test sets. Too much training will mean that the model will overfit the training dataset and have poor performance on the test set.A compromise is to train on the training dataset but to stop training at the point when performance on a validation dataset starts to degrade. This simple, effective, and widely used approach to training machine learning mode is called early stopping.

slack_factor: The amount of slack allowed with respect to the best performing training run. This factor specifies the slack as a ratio.
Any run that doesn't fall within the slack factor or slack amount of the evaluation metric with respect to the best performing run will be terminated. This means that with this policy, the best performing runs will execute until they finish.This allows a relatively intuitive method to screen models, only retaining those with similar or better performance.


## AutoML
**Description of the model and hyperparameters generated by AutoML.**
Automated machine learning (AutoML) is the process of automating the process of applying machine learning to real-world problems. AutoML covers the complete pipeline from the raw dataset to the deployable machine learning model. AutoML was proposed as an artificial intelligence-based solution to the ever-growing challenge of applying machine learning. The high degree of automation in AutoML allows non-experts to make use of machine learning models and techniques without requiring becoming an expert in the field first.
In ML Azure we have to configure the AutoML run so i configured it as below:

automl_config = AutoMLConfig(
    experiment_timeout_minutes=50,
    task="classification",
    primary_metric="accuracy",
    training_data=data,
    label_column_name='y',
    n_cross_validations=5,
    )
    
The algorithm used in AutoML was SparseNormalizer, XGBoostClassifier, MaxAbsScaler, LightGBM and VotingEnsemble.Among all the models VotingEnsemble was performing best with the highest Accuracy of 0.9180.    

## Pipeline comparison
Both the models performed very well , Azure ML hyperdive model achieved 91.4% accuracy and the autoML model achieved 91.8% accuracy.
Here is the best run summary for the model:
Metrics
  Accuracy:0.9140150818466066
  Max iterations:23
  Regularization Strength:0.619579284712275
  
Parameter sampling
Sampling policy name
RANDOM

Parameter space
{"C":["uniform",[0,10]],"max_iter":["quniform",[20,180,1]]}

## Future work
In order to improve the model accuracy we can do more feature engineering to find best optimal features for the prediction model.
Also we can do experimentation with n_cross_validations. As cross-validation is the process of taking many subsets of the full training data and training a model on each subset, the higher the number of cross validations will be, the higher we can achieved the accuracy of model.
Furthur we can also try different classification algorithm, In this case we can apply XGBoost alogoritm, as in some cases it is obseved that XGBoost performed well in comparison to other classification algorithm special when dataset is large.
