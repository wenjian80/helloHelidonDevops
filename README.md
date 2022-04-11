

# Disclaimer
**Disclaimer: The information presented in this article is for educational/general information purposes only. Any scripts or examples are presented where-is, as-is, and are unsupported by Oracle Support and Development. Always refer to the official oracle documentation and other necessary documentation for offical steps.**

1. Please refer to offical documentation for proper steps.
2. The scripts provided are just a reference/educational.

# Purpose
1. A simple demo on how oci devops work
2. Some steps are not explain in detail.
3. It is a shorten version from our oracle live labs as it involve quite a number of steps . For full explanation please refer to (https://oracle.github.io/cloudtestdrive/AppDev/cloud-native/livelabs/standalone/devops-into-oke/index.html?lab=devops-into-oke)
4. These instructions allow you to clone this repo to your own git follow the general instructions below and you are good to go.

# Prereq
1. We are making use of oke, container registry, container artifacts, vaults, oci devops service
2. OKE is used to deploy our sample app.
3. Application is build devops services and the image is push to registry.
4.  Yaml files are stored in container artifacts.
5. Vaults are use to store some secret info.
6. Policy rules need to be created. Refer to the doc link. Below is some screen shots of example policy. https://docs.oracle.com/en-us/iaas/Content/devops/using/devops_iampolicies.htm
![General Policy](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/polciy1.JPG)
![Dyanmic Group Policy](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/polciy2.JPG)

8. In this demo we are mirror from the git. Refer to documentation on how it is configured. (https://docs.oracle.com/en-us/iaas/Content/devops/using/create_connection.htm)
9. We are also storing some secret variables such as the url and path of the registry in the vault so that the build yaml can pull the values.

# Step 1 (OKE)
1. Create an OKE cluster.
2. We are using public endpoints here. OIC devops support private endpoints as well.
3. Necessary policy rules need to be create.

# Step 2 (Registry)
1. Create a registry below in your compartment.
2. The inital bj is going to pass as an parameter in devops service later.

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/registry.JPG)

# Step 3 (Valut)
1. We using vault to store the git access token as well the registry url and name. Refer to https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token to create an access token to store in vault.
2. In the below screen you will create 3 secret. Access token, url of registry repo and registry namespace of repo

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/scret.JPG)

# Step 4 (Create Artifcat registry)
1. Under Developer Services->Artifcat Registery. Go create one under your compartment

# Step 5 (Create Devops Project)
1. Under Developer Services->Devops ->Project. Go create one under your compartment

# Step 6 (Create Environment)
1. Create an environment to your oke.

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/environment.JPG)

# Step 7 (Create code repo)
1. Mirror your repo. The name will helloHelidonDevops

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/mirror.JPG)

# Step 8(Create build pipeline)
1. Create a build pipeline with the below stages.
2. Basically it have 2 main stage. One is based on build_spec yaml to compile and build the image. The other steps is to deliver the artifacts.
3. The heart of the CI here is actually the build specs.
4. Refer to https://docs.oracle.com/en-us/iaas/Content/devops/using/build_specs.htm to get familiar with the syntax 
5. Create a build stage. It will make use of yaml/build spec to compile and build the image

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/stage1.JPG)
6. Create a deliver image stage. It will push the image to the registry as well as the yaml artifacts

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/stage2.JPG)
7. Next create a parameter.

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/stage4parameters.JPG)
8. The end state will look like this

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/satge3.JPG)

10. Let examine the build spec and explain what is going on.
11. In the below we are getting the values from vault secrets using the ocid. We assign those values to "exported variables". These export variables can be use in later deployment pipeline as well as replacing the yaml variables. YOUR_INITIALS is also a parameter that is pass in from the UI or cli as well.

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/variables1.JPG)
12. If you look at the [app.yaml](https://github.com/wenjian80/helloHelidonDevops/blob/main/yaml/app.yaml)  it is replcaing the values in the yaml

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/app_yaml.JPG)

14. Click on "start manual run" and see the CI tools take place. You can also look and dwonload the logs from the left side of the screen

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/run2.JPG)

16. After the build is finished, you will see the image being push to the registry as well as the artifcats

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/output1.JPG)
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/output2.JPG)

# Step 9(Create deploy pipeline and link it up)
1. Click on Deployment pipelines and create a new pipeline
2. Create a stage by choosing ""Apply manifest to your kubernetes cluster" This below stage will deploy the app deployment

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/deploy1.JPG)

4. Create another stage. This below stage will deploy the ingress controller
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/deploy2.JPG)

5. Create another stage. This below stage will deploy the ingress configuratuion to point to the app
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/deploy3.JPG)

6. The end state will look like this
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/deploy4.JPG)

7. Next let link up the CI with the CD. Click on the previous build pipeline and add a stage to trigger this deployment after the build is finish.
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/deployment1.JPG)

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/deployment2.JPG)

7. The end state will look like the below
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/deployment3.JPG)

8. Click on manual run again. This time it will compile and deploy to your k8 cluster
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/run1.JPG)

9. You will see all the resources getting deploy into your k8
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/run2.JPG)

10. Trying htting the app and you will see that it is version 0.1
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/run4.JPG)

# Step 9(Trigger a build when code are commited)
1. Click on trigger to create trigger
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/trigger1.JPG)

![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/trigger2.JPG)
  
2. Now let edit some code and commit the changes. We are going to change /helloHelidonDevops/blob/main/src/main/resources/application.yaml.
3. We going to change version: "0.1" to version: "0.2" and commit.
4. Since our mirror is 2 min let's wait and you see see the trigger is triggered off.
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/triggerbuild.JPG)

5. After the process is done, try hitting the app again and you will see now is version 0.2
![enter image description here](https://github.com/wenjian80/helloHelidonDevopsScreenShots/blob/main/triggerbuild2.JPG)