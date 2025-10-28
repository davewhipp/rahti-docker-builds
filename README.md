# Building Docker images on Rahti
Date: 24.02.2025

## General

The steps below present an overview of how to build a Docker image on the CSC's Rahti system for use in Noppe. Although it is possible to build Docker images locally and push them to Rahti, this is not that easy for users with Apple Silicon machines. The instructions are written for users who already are familiar with the CSC environment and have experience with their systems. However, you can likely follow them if you read some of the documentation linked below as well.

## Useful links

- https://docs.csc.fi/cloud/rahti/images/creating/#using-a-local-folder-for-building
- https://docs.csc.fi/cloud/rahti/usage/cli/#how-to-login-in-the-registry
- https://docs.csc.fi/cloud/rahti/usage/cli/#how-to-install-the-oc-tool
- https://docs.csc.fi/cloud/rahti/images/keeping_docker_images_small/
- https://docs.csc.fi/cloud/rahti/usage/projects_and_quota/#creating-a-project

## Steps

1. Make sure you have a CSC project that is able to be used with Rahti (e.g., geo-python; check at https://my.csc.fi/projects/)
2. Make sure you have a project in Rahti that refers to the CSC project number (see https://docs.csc.fi/cloud/rahti/usage/projects_and_quota/#creating-a-project)
3. Log into Rahti at https://rahti.csc.fi
4. Install the openshift client, if needed (I did so using homebrew as `brew install openshift-cli`)
5. Log in to Rahti using open shift (e.g., `oc login --token=... --server=https://api.2.rahti.csc.fi:6443`). You can get the command by clicking on your name on the Rahti website and then "Copy login command"
    - Note that if you have more than one project, you may need to change the active project using the `oc project <project name>` command. For example, `oc project geo-python`.
6. Create the space for building: `oc new-build --to=my-hello-image:devel --name=my-hello --binary` where `my-hello-image` should be the name of the Docker image to create, `devel` is the tag, and `my-hello` is the name used when building (I think). Rename as appropriate. For example, `oc new-build --to=geo-python:2025.0 --name=geo-python --binary`

    - Thus, when updating an image with a new tag, both the tag and build name should be updated. For example, `oc new-build --to=geo-python:2025.1 --name=geo-python1 --binary`

8. Copy the dockerfile to a directory (with all other needed files and nothing else) as `Dockerfile`
9. Build the image by changing into that directory and running `oc start-build my-hello --from-dir=./ -F`. For example, `oc start-build geo-python --from-dir=./ -F`
    - If the build fails, it may be necessary to bump up the resources by editing the yaml BuildConfig file as shown below in the section "Build resources". See https://docs.csc.fi/cloud/rahti/images/creating/#troubleshooting. I got things to work by just copying and pasting the example resource info below (delete the curly braces that are there by default in the build resource yaml file).
10. The new image will be available as `image-registry.apps.2.rahti.csc.fi/<project-name>/my-hello-image:devel`. For example, `image-registry.apps.2.rahti.csc.fi/geo-python/geo-python:2025.0`.
11. Allow anonymous image access: `oc policy add-role-to-user registry-viewer system:anonymous -n <project>`. For example, `oc policy add-role-to-user registry-viewer system:anonymous -n geo-python`
12. Create/update Noppe workspace to use the new image at location from point 9 above.
13. Test on Noppe!

### Build resources

```yaml
resources:
  limits:
    cpu: '1'
    memory: 8Gi
  requests:
    cpu: 200m
    memory: 1600Mi
```
