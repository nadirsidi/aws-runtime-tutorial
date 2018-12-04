# README

## AWS Custom Runtime Tutorial

This is a repo containing the code to run the first part of the AWS tutorial for custom lambda runtimes, which is found [here](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-walkthrough.html) in the AWS Lambda documentation.

The tutorial and related documentation on custom runtimes and the Lambda Runtime API isn't entirely clear to me. I am trying to better understand the tutorial through this repo in an effort to build an `R` runtime layer.

## Run Locally with Docker

I was able to get this example to run by copying the `bootstrap` and `function.sh` scripts, making them both executable with `chmod +x` and then running them with the excellent Lambda Docker image maintained by [Lambci](https://hub.docker.com/r/lambci/lambda/). They have an image with the tag `provided` which is specifically for runtimes provided with a `bootstrap` file.

To run this example from the locally pulled repo (assuming you're set-up with Docker):
```
docker pull lambci/lambda:provided

docker run --rm -v `PWD`:/var/task lambci/lambda:provided function.handler '{"some": "request"}'
```

## Explanation / Notes

Here I am making some notes on what the `bootstrap` and `function.sh` files are doing. I am not a bash scripting power user, so I'll have to look-up the commands as I go along.

### The `bootstrap` script

* The `set` command improves bash script error handling. Option `-e` will cause the script to exit on error rather than continue, `-o pipefail` will print the exit status of the right command in a failed pipe scenario, and `-u` will through an error for unset variables.

* Sourcing the handler function
  + The location of the lambda code which is at the value of the env variable `LAMBDA_TASK_ROOT`. For this image, it appears that's the `/var/task` directory, which is why we bind mount the Lambda function code there.
  + The location of the lambda handler function is at the value `_HANDLER` which seems to be the second argument for the docker image, `function.handler` in this case.
  + The `cut -d. -f1` pulls out the name of the shell script with the function. In the example, `function.handler` is the name of the handler function, so the `cut -d. -f1` will return `function` so we can source the `function.sh` script.

* In the `while` loop:
  + Make a unique temporary file
  + Use curl to silently `-s` get the request `-X` from the runtime API endpoint and dump the header `-D` to the temp file. Show any errors `-S`.
  + Use some grep magic to pull the `Lambda-Runtime-Aws-Request-Id` from the header. This makes up part of the endpoint that the script needs to POST the function result back to.
  +

## Next Steps for Building Custom Runtime

It seems only lines 16 & 17 are where the function gets executed. If the binary for the runtime is accessible, this could be a call to a function as long as it understands the syntax of the request payload.

Use the existing bash curl statements to interact with the runtime api, there's no need to implement that in each language.
