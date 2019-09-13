# Jenkins

## Credentials

We must store three credentials:

1. AWS credentials to access the private registry.
2. A GitHub token to register the hooks in the repository.
3. An account for kubernetes.

## General configuration

We have to configure a Git server to register the repository hooks. We will use the token that we have configured in credentials.

We have configured environment variables to make it more transversal to make configuration changes, they are the following:

| Variable | Valor | Descripción |
| -------- | ----- | ----------- |
| AMAZON_ECS_REGISTRY | 303679435142.dkr.ecr.eu-west-1.amazonaws.com | Registry Docker privado |
| DOCKER_IMAGE_NAME_E2E_TEST | springboot-e2e | Nombre de la imagen Docker para pruebas e2e |
| DOCKER_IMAGE_NAME_SPRING_BOOT_EXAMPLE | springboot-example | Nombre de la imagen Docker para la aplicación |

## Jobs

We have created Pipelines to solve the tasks of the continuous integration cycle.

But first we describe the method of work, being Kubernetes what we create is a pod with them containing the containers that we will use during the pipeline. We also define the environment variables and the volumes we map within the containers.

Likewise, being Kubernetes, we do not have workers in the traditional style, so we generate one in each execution of the pipeline and name it with a random name to avoid collisions.

* Big_pipeline_test

The definition of the pipeline is found in `jenkinsfile_pre.groovy`

In this pipeline we make continuous integration. We will describe each stage.

The steps to follow:

1. Clone the repo.
2. Build the package.
3. Run unit tests.
4. Build the development Docker image that will be tagged with the commit id of the last push, the qa tag and the latest tag.
5. We clean the images of Docker type _dangling_ that are images without label and that end up taking up a lot of disk space.
6. The three Docker images are uploaded to the private registry.
7. The Kubernetes manifesto is updated with the development docker image we have created.
8. The dev application is deployed using the kubernetes manifest.
9. We hope the application is up and running.
10. We launched the e2e test.
11. If everything is fine so far, we remove the application that we have deployed in development.
12. We update the Kubernetes manifesto with the QA tag.
13. We deploy the QA environment. This environment will be deployed in the infrastructure.

* Big_pipeline_release

The definition of the pipeline is found in `jenkinsfile_release.groovy`

The release is done by hand by setting the value of the release tag and knowing the commit id tag that is going to be promoted to release.

On this occasion the steps are:

1. Clone the repo.
2. Swallow the dev image as release.
3. Clean the docker dangling images.
4. Push the image to the repository.
5. Update the Kubernetes manifesto.
6. Deploy the application.

* Build_e2e_container

The definition of the pipeline is found in `jenkinsfile_build_e2e_docker_image.groovy`

This job builds a Docker image to test our application. This job is executed manually, so the Docker image tag is needed as an input parameter.

The image contains, among others, a browser, Maven and Java

The steps in this job are simple,

1. Clone the repo.
2. Build the image.
3. Clean the docker dangling images.
4. Push to repository.

* change_release

The definition of the pipeline is found in `jenkinsfile_set_release.groovy`

This job serves in the case of finding a problem with the release that is deployed and we want to change the version. Basically it is an excerpt from the release and test jobs in which we label the manifest with the version we want to deploy at the moment.

So the steps are:

1. Clone the repo.
2. Update the manifest tag.
3. Update the deployment.

* Helm_Pre

We have written a chart to work with Helm. The Chart depends on MySQL, so the template for the database is included in the charts directory.

In the pipeline we have based on the previous one of _pre_ and we have replaced the stages where we use Kubernetes with those we use Helm. We have also used a new container that includes both _kubectl_ and _helm_.

In the Chart we have parameterized the tag of the docker image that we are going to use, the persistence (not in dev and qa, if in prod) and the Service Type, we leave it as an IP cluster for dev and in LoadBalancer for qa and prod.

The deployment with Helm has the peculiarity that it is done differently if there is already a previous deployment so we have written the casuistry and check if it exists or not to act accordingly.

* Helm_Release

If we want to release with Helm we have also based on the pipeline with Kubernetes.

We have added the container with _kubectl_ and _helm_ to be able to operate against the Kubernetes cluster.

The image we want to promote to production is also tagged and uploaded to development.

** Note **: For it to work, there must be a prior deployment of the application.
