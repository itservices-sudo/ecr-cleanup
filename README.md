# ecr-cleanup

ECR Cleanup to clean old unused images from the repository


To cleanup old images from ECR we can run script. But it can affect our production server. If we don't know what images are in production server.

So to we can give tags to the Docker images. So that we can identify which image is running on production server.

We can run script to remove untagged images.

#!/bin/bash

# Script to clean images with no tag from ECR repos

# Usage:
# ./clean-untagged-ecr-images.sh <IMAGE-REPO-NAME>

# Dependencies
# Requires jq and aws command line
# aws credentials are also needed to access the repo. These can be set up using the "aws configure" command.

# Check if jq is available
type jq >/dev/null 2>&1 || { echo >&2 "The jq utility is required for this scipt to run."; exit 3; }

# Check if aws cli is available
type aws >/dev/null 2>&1 || { echo >&2 "The aws cli is required for this script to run."; exit 3; }

# Check number of arguments parsed
if [ $# -ne 1 ]; then 
	echo "Useage ./clean-untagged-ecr-images.sh <IMAGE-REPO-NAME>"
	exit 1
fi
REPO=$1

IMAGES=$(aws ecr list-images --repository-name $REPO --query 'imageIds[?type(imageTag)!=`string`].[imageDigest]' --output text)

read -p "Delete all images with no tag from $REPO (y/n)? " CHOICE
case "$CHOICE" in 
  y|Y ) 
    for DIGEST in ${IMAGES[*]}; do
      aws ecr batch-delete-image --repository-name $REPO --image-ids imageDigest=$DIGEST
    done
  ;;
  
  
 or
 
  
  * ) exit 0 ;;
esac



We can setup lifecycle in AWS 
 we can make a lambda function to clean the repository time to time 
(1) The build system builds and pushes the container image. The build system also tags the image being pushed, either with the source control commit SHA hash value or the incremental package version. This gives full control over running a versioned container image in the task definition. The versioned images (images tagged in this way version the container images) are further used to run the tasks, using your own scheduler or by using the ECS scheduling capabilities.
(2) The Lambda function gets invoked by CloudWatch Events at the specified time and queries all available clusters.
(3) Based on the coded Python logic, the container image tags being used by running tasks are discovered.
(4) The Lambda function also queries all images available in ECR and, based on the coded logic, decides on the image tags to be cleaned.
(5) The Lambda function deletes the images as discovered by the coded logic.


