# Deskripsi Masalah

Kami memiliki aplikasi SpringBoot + MySQL. Kami ingin memiliki aplikasi dalam produksi dan yang lain di QA.

## Requirement

1. Unit testing.
2. Tes end to end.
3. env QA.
4. env produksi.
5. delivery to production dengan cepat.
6. Mampu mengubah ke versi apa pun dengan cepat.

## Struktur Repositori

- docker: dockerfile
- helm: Template untuk manajer paket Helm Kubernetes.
- k8s: Manifes untuk Kubernetes.
- src: repository.
- tools: Skrip yang berguna untuk proses pengembangan.

## Docker

We build two Docker images. One with the application itself and one with the tools to run the tests end to end (e2e).

We have a private Docker Registry on AWS to upload Docker images.

## Helm

In Helm we have created the Charts to deploy the application. The template is parameterized so that we can choose which version of the application to deploy.

## K8s

We have written two manifests for the test / qa environment and another for production. The difference is that the production environment configures persistence for the database.

## src

As we say this directory contains the source code of the Maven project.

## Tools

Useful scripts, for the moment one that loops until the application is ready.
