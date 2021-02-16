# sagemaker-custom-model
This repository is a collection of commonly used containers that can be deployed easily on sagemaker and addresses common issues like installing specific versions of python/R which become problematic while creating custom containers on docker

## Structure of Repository
The following structure will be followed for subsequent modelling containers in the repositories

    .
    ├── model-name                                                # catboost/lightgbm/xgboost etc
        ├── Readme.md                                             # Readme file on how to use test model on local Docker and then train on sagemaker
        ├── Dockerfile                                            # Dockerfile to create container
        ├── build_and_push.sh                                     # Build and push container
        ├── local_test                                            # Material to test local deployment of code
        |    ├── train_local.sh                                   # Train using container locally
        |    ├── serve_local.sh                                   # Serve model locally
        |    ├── predict.sh                                       # Predict model on local serving using payload
        |    ├── payload.csv                                      # payload for local testing
        |    └── test_dir                                           
        |            ├── input
        |            |   ├── config 
        |            |   |   └── hyperparameters.json             #Hyperparameters for local testing
        |            |   └── data
        |            |       └── training
        |            |           └──  train.csv                   #Training data (1000 rows) for local testing
        |            └── model
        |                └── model object                         #Once trained locally model object will be saved here
        └── model_container
              ├── nginx.conf                                      #Setup server
              ├── predictor.py                                    #Prediction function
              ├── serve                                           #Implements the scoring service shell
              ├── train                                           #Training Code 
              └── wsgi.py                                         #Wrapper for gunicorn

