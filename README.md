# tensorflow-serving-openfaas

Example of packaging TensorFlow Serving with OpenFaaS to be deployed and managed through OpenFaaS with auto-scaling, scale-from-zero and a sane configuration for Kubernetes.

This example was adapted from: https://www.tensorflow.org/serving

## Pre-reqs

* [OpenFaaS](https://docs.openfaas.com/)
* OpenFaaS CLI
* Docker

## Instructions

* Clone the repo

```sh
$ mkdir -p ~/dev/
$ cd ~/dev/
$ git clone https://github.com/alexellis/tensorflow-serving-openfaas
```

* Clone the sample model and copy it to the function's build context

```sh
$ cd ~/dev/tensorflow-serving-openfaas

$ git clone https://github.com/tensorflow/serving

$ cp -r serving/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_cpu ./ts-serve/saved_model_half_plus_two_cpu
```

* Edit the Docker Hub username

You need to edit the stack.yml file and replace `alexellis2` with your Docker Hub account.

* Build the function image

```sh
$  faas-cli build
```

You should now have a Docker image in your local library which you can deploy to a cluster with `faas-cli up`

* Test the function locally

All OpenFaaS images can be run stand-alone without OpenFaaS installed, let's do a quick test, but replace `alexellis2` with your own name.

```sh
$ docker run -p 8081:8080 -ti alexellis2/ts-serve:latest
```

Now in another terminal:

```sh
$ curl -d '{"instances": [1.0, 2.0, 5.0]}' \
   -X POST http://127.0.0.1:8081/v1/models/half_plus_two:predict

{
    "predictions": [2.5, 3.0, 4.5
    ]
}
```

From here you can run `faas-cli up` and then invoke your function from the OpenFaaS UI, CLI or REST API.

```sh
$ export OPENFAAS_URL=http://127.0.0.1:8080

$ curl -d '{"instances": [1.0, 2.0, 5.0]}' $OPENFAAS_URL/function/ts-serve/v1/models/half_plus_two:predict

{
    "predictions": [2.5, 3.0, 4.5
    ]
}
```

<!-- You can even run the inference asynchronously with a callback-URL or separate function set-up to receive the result. -->

Find out more information about your function's endpoints:

```sh
$ faas-cli describe ts-serve
Name:                ts-serve
Status:              Ready
Replicas:            1
Available replicas:  1
Invocations:         5
Image:               alexellis2/ts-serve:latest
Function process:    
URL:                 http://127.0.0.1:8080/function/ts-serve
Async URL:           http://127.0.0.1:8080/async-function/ts-serve
Labels:              com.openfaas.function : ts-serve
                     function : true
```

<!-- 
Create a request bin: https://requestbin.fullcontact.com/ i.e. `http://requestbin.fullcontact.com/tgjgrrtg` then run:

```
$ export OPENFAAS_URL=http://127.0.0.1:8080
$ export CALLBACK_URL=http://requestbin.fullcontact.com/tgjgrrtg

$ curl -H "X-Callback-Url: $CALLBACK_URL" -d '{"instances": [1.0, 2.0, 5.0]}' $OPENFAAS_URL/async-function/ts-serve/v1/models/half_plus_two:predict
``` -->

* Try your own model

Now you can try your own model by editing ts-serve/Dockerfile.

Let me know what you think via [Twitter](https://twitter.com/alexellisuk)


