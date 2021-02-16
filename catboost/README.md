# How to deploy & use the container

Providing detailed instructions for deploying and using the container for prediction and serving jobs

## Container Deployment

If you are working on sagemaker notebooks it can be very easily deployed to ECR using the following shell script

```
$ ./build_and_push.sh <your-container-name>
```

## Local Testing

### Training

If you have successfully deployed the container to ECR then you can locally test the coontainer by simply cd ing local test directory and then running a training job which will create a model which can be served

```
$ cd local_test/
$ ./train_local.sh <your-container-name>

rm: remove write-protected regular file ‘test_dir/model/catboost-model’? 
rm: cannot remove ‘test_dir/output/*’: No such file or directory

Starting the training.
['/opt/ml/input/data/training/train.csv']
0:      learn: 0.6836703        test: 0.6846169 best: 0.6846169 (0)     total: 62.7ms   remaining: 564ms
1:      learn: 0.6767255        test: 0.6782776 best: 0.6782776 (1)     total: 64.9ms   remaining: 260ms
2:      learn: 0.6709505        test: 0.6730388 best: 0.6730388 (2)     total: 71.8ms   remaining: 167ms
3:      learn: 0.6663161        test: 0.6689348 best: 0.6689348 (3)     total: 77.2ms   remaining: 116ms
4:      learn: 0.6625784        test: 0.6656455 best: 0.6656455 (4)     total: 79.4ms   remaining: 79.4ms
5:      learn: 0.6595348        test: 0.6630076 best: 0.6630076 (5)     total: 81.5ms   remaining: 54.3ms
6:      learn: 0.6570557        test: 0.6608955 best: 0.6608955 (6)     total: 83.6ms   remaining: 35.8ms
7:      learn: 0.6550280        test: 0.6592036 best: 0.6592036 (7)     total: 87.4ms   remaining: 21.8ms
8:      learn: 0.6533832        test: 0.6578596 best: 0.6578596 (8)     total: 89.3ms   remaining: 9.93ms
9:      learn: 0.6520434        test: 0.6567920 best: 0.6567920 (9)     total: 91.4ms   remaining: 0us

bestTest = 0.6567919769
bestIteration = 9

Training complete.
```

### Serving

Once the training is complete you can serve the model using the following steps

```
sh-4.2$ ./serve_local.sh <your-container-name>
Starting the inference server with 2 workers.
[2021-02-16 20:30:13 +0000] [9] [INFO] Starting gunicorn 19.10.0
[2021-02-16 20:30:13 +0000] [9] [INFO] Listening at: unix:/tmp/gunicorn.sock (9)
[2021-02-16 20:30:13 +0000] [9] [INFO] Using worker: gevent
[2021-02-16 20:30:13 +0000] [14] [INFO] Booting worker with pid: 14
[2021-02-16 20:30:13 +0000] [15] [INFO] Booting worker with pid: 15


```
Keep this shell running

### Predictions

In a seperate shell cd to the local_test directory & use the payload to generate prediction

```
$ cd local_test/
$ ./predict.sh payload.csv
*   Trying 127.0.0.1:8080...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /invocations HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.68.0
> Accept: */*
> Content-Type: text/csv
> Content-Length: 2111
> Expect: 100-continue
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 100 Continue
* We are completely uploaded and fine
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.10.3 (Ubuntu)
< Date: Tue, 16 Feb 2021 20:32:34 GMT
< Content-Type: text/csv; charset=utf-8
< Content-Length: 191
< Connection: keep-alive
< 
0.403228810117072
0.403228810117072
0.403228810117072
0.4189208464511033
0.4189208464511033
0.4189208464511033
0.40430497095967266
0.40430497095967266
0.40430497095967266
0.40131509112970815
* Connection #0 to host localhost left intact

```
You will simultaneously get response on the serve shell as well which will tell you if payload is being ingested correctly

## Traing estimator object and Create Endpoint

Once the container has been deployed and local testing has been succesful then you can use the container to train models on larger data sets on sagemaker

```py

model = sage.estimator.Estimator(  container_image_uri,
                                   role, 1, 
                                   'ml.c5.4xlarge', 
                                   output_path= s3_model_output,
                                   sagemaker_session=sess,
                                   hyperparameters={                                      
                                       "learning_rate": 0.01, 
                                       "loss_function": "Logloss", 
                                       "iterations": 10000, 
                                       "depth": 8,  
                                       "random_seed": 9, 
                                       "verbose": "true", 
                                       "validation": "true", 
                                       "cat_indices": 12, # Till which column there is categorical index in your data
                                       "od_type": "Iter", 
                                       "od_wait": 300
                                   }
                                )
model.fit(s3_input_train)

# Create Endpoint
from sagemaker.predictor import CSVSerializer
predictor = model.deploy(1, 'ml.m4.xlarge', 
                         serializer=CSVSerializer(),
                         endpoint_name= "<your-endpoint-name">
                        )
                        
# Use endpoint
import boto3
runtime= boto3.client('runtime.sagemaker')                       
win_response = runtime.invoke_endpoint(EndpointName="<your-endpoint-name">,
                                              ContentType='text/csv',
                                              Body=your_payload_csv)
```





