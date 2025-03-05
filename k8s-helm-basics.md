# Helm Basics

![image](https://github.com/user-attachments/assets/b855d451-89f8-4adf-84ba-6f714243c532)

We need to apply kubectl apply on every YAML file. We can just write all object declarations ina single YAML file and be done with it.

![image](https://github.com/user-attachments/assets/00c65686-89ad-47f8-b711-63ec21e153f8)

## Helm Components

![image](https://github.com/user-attachments/assets/2f4c2f93-14ce-4b5a-8067-fbeb1aa134e1)

The releases are installed, the charts used, revision states and so on, Helm will need a place to savve this data. This data is known as metadata. If another person would need to work with our releases thorugh Helm, they would need a copy of this data. So instead, Helm does the smart thing and save this metadata directly in our k8s cluster as k8s secret. This way, the data survives and everyone from our team can access it, so they can do Helm upgrades or whatever it is that they want to do. So Helm will always know about everything it did.

## Helm Charts

![image](https://github.com/user-attachments/assets/d25176ee-32da-4692-ab6c-7e85bba695be)

![image](https://github.com/user-attachments/assets/87f6f50e-fd6d-4f94-af69-c6ff5f47f9c6)

As far as the human operators are concerned, charts are just a bunch of text files. Each specific file named in a sepcific way has a well-defined purpose. 

![image](https://github.com/user-attachments/assets/94da35f5-0533-4cab-a571-fb45f8d715df)

Example chart.yaml

![image](https://github.com/user-attachments/assets/56c537a7-7c70-4424-866d-9638b58b4d6d)

![image](https://github.com/user-attachments/assets/7b9614b2-7db9-435d-b349-d3421dcb0b06)

The app version is a version of the application that is inside of this chart. There are 2 types: application (defalut type), library.

![image](https://github.com/user-attachments/assets/5f9922b5-95ae-4f6c-8e42-0824904eac0e)

![image](https://github.com/user-attachments/assets/bbb64671-8fe3-4d1e-9dd9-65c53017796f)

### Helm Chart Structure
It has a templates directory that has the template files that we just talk about. 

![image](https://github.com/user-attachments/assets/f2f5c094-5930-410d-8521-07f3d7bf2cb3)

## Helm Releases

![image](https://github.com/user-attachments/assets/02fc60cd-b170-44aa-85c7-911e4b821249)

## Helm Repositories

![image](https://github.com/user-attachments/assets/7df3f480-f306-4127-81d5-70ac78ff436b)

![image](https://github.com/user-attachments/assets/75294413-ab99-466a-be2a-384eb3a0378b)


# Kustomize Basics

![image](https://github.com/user-attachments/assets/2dc8dcff-4061-404d-a9d9-464b244539be)

Let's say that we want to customize our deployments so that it behaves a little bit differently in each one of our environments.

And specifically in this case, let's say we wanna modify the replicas on a per environment basis so that in our development environment we're developing on our local machine, which is not very powerful we just wanna have one replica. In our staging we might have two to three and then in our production which is gonna be handling a lot of traffic maybe we want 5 to 10, so how do we actually go about modifying the different configurations on a per environment basis 'cause right now we just have one nginx-deployment.yml file. And so it's going to deploy one replica across all of our environments.

![image](https://github.com/user-attachments/assets/34905580-dda4-4485-81a3-9e902fa09483)

One of the simplest solutions to this problem is to create three separate directories, one for each environment, and what we want to do is we want to have all of the Kubernetes configs for each environment within their respective folder. So we're essentially duplicating the configs across all of the three different environments and then just modifying the attributes on a per environment basis so that we can have different replicas for each environment.

![image](https://github.com/user-attachments/assets/631dd749-9154-4a24-bd6e-fac9668c9d4e)

So you could see it's the exact same config that we're essentially copying in the three different folders, we're just updating the specific parameters that we want updated. And in this case it's just the replicas.

![image](https://github.com/user-attachments/assets/9337552e-269c-4f59-b475-a92795155924)

![image](https://github.com/user-attachments/assets/beca0454-64c5-4054-941e-42171a4874ea)

However, I will say that it is not the most optimal solution and it's certainly not the most scalable solution. And the reason I say this is because imagine if we start to expand our Kubernetes environment so we start creating more resources.

![image](https://github.com/user-attachments/assets/7660259b-dead-41cd-b997-240079566ec7)

And so let's say we wanna create a new service. We created a service.yml file. Well now when we create a service.yml file, we have to remember to copy it to all three directories because we don't want it to be missing from one of our environments.

And really, it's going to be only a matter of time before either you or one of your teammates forgets to copy or change something in all of the directories. And then you're gonna have a mismatch in configs across your different environments. And so that's why I would ultimately not recommend this solution. It may work for very small deployments, but as your, the number of resources that you have continues to grow, it's going to be harder and harder to maintain this. And this is one of the main reasons why Kustomize was created.

We need a better solution to addressing this issue.

We wanna treat this just like regular application code where we're not repeating everything more than once. And I do wanna really reiterate what is the underlying problem. And that is that we want a way to reuse our Kubernetes configs and only modify what needs to be changed on a per environment basis.

![image](https://github.com/user-attachments/assets/dc0bb3d0-7ec3-473d-8806-df59b9bd6223)

When you're using Kustomize, your folder structure is going to follow a very similar pattern. You're going to have a base directory. The base directory, once again, is going to contain all of your base configurations. So all of your Kubernetes resources, all of your configs that are going to be pretty similar across all of your deployments, you're gonna wanna put them there. And then you're gonna have a separate folder for overlays. And within the overlays folder, you're gonna have a different folder for each one of your environments. And each one of these environments are going to have the values that you want to overwriten and change from the base config as well as any new resources that should only be added exclusively for that specific environment.

![image](https://github.com/user-attachments/assets/764258d0-b141-4302-8ac1-41702bb71ab6)

And so the workflow is going to look like this.

![image](https://github.com/user-attachments/assets/18d6ae6d-9e61-47d9-8e23-6d1f45be611b)

You're gonna have your base configs, you're gonna have your overlays. Kustomize is going to take both of them, and it's going to create the final Kubernetes manifests that we can then apply to our Kubernetes cluster. The great part about Kustomize is it actually comes built in with kubectl, so you don't have to install any other packages.

![image](https://github.com/user-attachments/assets/28045c75-f27a-4d29-8c24-b80345cd5f7f)

## Kustomize vs helm

![image](https://github.com/user-attachments/assets/06526d05-6f3f-42d4-a94f-dca3764c0cd7)

You can see we have the replica's property and the variable name is called replicaCount. So to provide a value for the replicaCount variable, we would create a values.yaml file that's going to contain all of the values for the variables.

![image](https://github.com/user-attachments/assets/58d7bfb1-1c77-48e7-b226-67df97190165)

Under the templates directory, this is going to contain all of our Kubernetes manifests where we've inserted all of those variables using that Go templating syntax.

![image](https://github.com/user-attachments/assets/b81fc430-2211-401e-99f4-9ab03edde940)

![image](https://github.com/user-attachments/assets/be6ee44c-f7ea-4620-9aac-dc31cf418efa)

## Installation/Setup Kustomize

![image](https://github.com/user-attachments/assets/ef887300-2180-4041-abca-d110f6673a6a)

## kustomization.yaml file

![image](https://github.com/user-attachments/assets/7fee0453-e9bd-4922-8d4c-a0e8a86473e6)

- The first is it's going to contain a list of all of the Kubernetes resources that should be managed by Kustomize. And so this is going to be just a list of all of the YAML files that you want Kustomize to manage. And so in this case, I have my nginx deployment YAML file, and I have my nginx service YAML file. And those are both listed under the resources section of my kustomization.yaml file.

- The second thing is going to be all of the customizations that we want apply to or transformation. So what are the things that we need to change? So in this example, I have a very simple one where we're going to add a common label to all of the resources that we create using Kustomize. And it's going to add a label that has a key of company and a value of KodeKloud.

![image](https://github.com/user-attachments/assets/ceb2b6d0-acfe-4b22-a460-4aeb400484e2)

And most importantly, you can see the transformations, which is we applied an extra label called company: KodeKloud, and it's gonna get applied to both the service and the nginx. And that's what the common label transformation does. It applies a specific label to all of your Kubernetes resources.

**Conclude**

![image](https://github.com/user-attachments/assets/fde60384-22a7-4204-9835-97ad22693447)

## Kustomize Output

![image](https://github.com/user-attachments/assets/734871b6-535b-4617-af40-d407ba2ac834)

![image](https://github.com/user-attachments/assets/91182334-f969-4083-8652-2b4b8dd075b4)

## Kustomize ApiVersion & Kind

```Bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
```

![image](https://github.com/user-attachments/assets/2233c0c2-f1ec-45c4-bb42-1a107c20fad1)

## Managing Directories

![image](https://github.com/user-attachments/assets/cd161e35-f6b1-4fae-9795-abfa4003cde5)

However, as we grow out the number of directories or sub directories that we have, it's gonna start to become a little bit of a pain having to go into each subdirectory and doing a kubectl apply because then every time we make changes, every time we want to, apply our configs or delete our configs, we'd have to go inside each one of our sub directories and make sure we run that command inside each subdirectory. And we'd also have to configure our CICD pipeline to go inside each one of them and do the same exact thing. And things start to get messy at that point. This is where customized comes into play. So we can create a kustomization.yaml file in the root of our k8s directory.

![image](https://github.com/user-attachments/assets/3483854a-6c53-44b6-b643-f65dc5a85e61)

So we can create a kustomization.yaml file in the root of our k8s directory. And inside this YAML file or the kustomization.yaml file, we're going to list out all of the resources we want it to import. So we're gonna provide the relative path from the kustomization.yaml file, to all of the specific deployment and service yaml files within the API and database directory. So now customize is made aware of all of the different Kubernetes configs that we wanted to import.

![image](https://github.com/user-attachments/assets/c13330ee-a25d-442b-b67b-e8b12d1ca019)

We just run this within the root of the k8s directory. It's going to pull that kustomization.yaml file and the Kustomization,yaml file has the path to all of the individual YAML files we wanna deploy. And if you wanna do this natively with kubectl, you can always do a kubectl apply -k k8s. So customize has helped us address the issue of splitting all of our configs into separate directories.

![image](https://github.com/user-attachments/assets/b08c012a-188c-455e-8cfb-e7302236f93f)

I think we have a better way of handling this using customize. So what we can actually do is we can add in a separate kustomization.yaml file within each one of the sub directories. And then in the root Kustomization.yaml file, all we'll do is we'll provide a path to all of the different directories that we want included. So when we do that, when we specify either the API directory or the database directory inside that root kustomization.yaml file, it's going to go into that subdirectory and it's gonna look for a kustomization.yaml file.

![image](https://github.com/user-attachments/assets/9277100e-82df-4354-a1f4-be914c888de1)

![image](https://github.com/user-attachments/assets/a349a140-fdf3-4d5f-ab64-21a59198e37a)

## Common Transfomers

![image](https://github.com/user-attachments/assets/d83e99fc-bf30-45e6-9d46-c3e0b9fc8231)

Well, let's say that we want to specifically add a label in this case, so something like org KodeKloud, or maybe we wanna go into all of our Kubernetes objects and add a specific prefix or suffix to the name. So maybe we wanna append the word **-dev** to the end of our name, and we'll do the same thing across all of our Kubernetes configs.

![image](https://github.com/user-attachments/assets/7d6f803e-23b2-4cc7-91f7-a1cd34b9408d)

Now, we can go into each one of our yaml files and add these in. But keep in mind, in a production environment, you're going to have significantly more than just two yaml files. And so doing this by hand is not a scalable solution, and it's gonna be very time-consuming and it's going to lead to a lot of errors. And so this is why we have these common transformations in Kustomize. What they do is they allow us to make these common configuration changes across all of our Kubernetes resources. So let's go over some of the common transformations that we can apply.

![image](https://github.com/user-attachments/assets/ad6643fe-4057-4b65-b5f9-3d65434c23ae)

Well, we would go into our Kustomization.yaml file, and just add the commonLabel property and then specify the labels that you wanna add, and that's gonna go ahead and add this to all of your Kubernetes resources. Or more specifically, it's going to add it to all of the resources that are being imported by this specific Kustomization.yaml file.

![image](https://github.com/user-attachments/assets/891693d8-25ea-46de-a85c-4b3578986a3f)

So the Namespace Transformer is just going to put all of your Kubernetes resources under a specific namespace.

![image](https://github.com/user-attachments/assets/c701802f-7e8b-4e21-8978-c0ea9b3ee4cc)

For the Prefix/Suffic Transformer

![image](https://github.com/user-attachments/assets/f2d4a576-c725-45e7-b5f0-284302d7e7a2)

And for the last common transformation, we have the annotation transformation.

![image](https://github.com/user-attachments/assets/7c42a629-c276-4731-80a9-abb266751e51)

## Image Transformers

![image](https://github.com/user-attachments/assets/43324be7-8d07-4de2-918e-c372d3ab257c)

So now we know how to change the image using customize. 

![image](https://github.com/user-attachments/assets/2005e0ae-a603-4635-8c15-df2c898d28d3)

Now if we want to, we can actually combine the new image and new tag properties together so that we can modify not only the image, but also the tag.

![image](https://github.com/user-attachments/assets/c77b590f-9b5b-4cfa-9f8d-15c46f300b68)

## Patches Intro

![image](https://github.com/user-attachments/assets/957652e9-b837-4773-857c-c2e89bbb89cd)

So basically, if you want to apply a configuration across the board, like adding a label or assigning a namespace to all of your Kubernetes objects, you're gonna use a common transformer. However, if you wanna apply or change something on one specific object or just a couple of objects, you would use a kustomize patch. So things like updating the number of replicas in a deployment, you would use a kustomize patch to match a specific object and then change that value.

 ![image](https://github.com/user-attachments/assets/bc6dada9-f976-4922-b4db-21e79a548459)

In this case, I'm matching on a kind of deployment. So it's going to look through all of my Kubernetes configs and it's going to match on only things that are of type deployment. And then I'm going to match on a deployment with a specific name of api-deployment. And that's going to match this exact Kubernetes object. Then we have to specify patch, and then we do this | and then a -. So you have to add that, and this is for what is referred to as an inline patch.

![image](https://github.com/user-attachments/assets/6ae9a71d-f717-498d-8e0c-fcf30cebf65d)

Now look at one more example

![image](https://github.com/user-attachments/assets/c92ac052-d10d-4e43-9879-f412160bfaf0)

In kustomize, there's actually two different ways to define a patch. So till now, we've only been working with what's referred to as a JSON 6902 patch. And so a JSON 6902 patch has two things that you have to provide, the Target, which is going to tell kustomize what Kubernetes object do we want to patch, and then we have to provide the patch details.

But I wanted to go over the second way to define a patch, and that's called a **strategic merge patch**.

I basically just copied the original API deployment file and I just pasted it in there, and then I deleted all the stuff I don't wanna change. And so this is one of the perks of using a strategic merge patches that it's just using regular Kubernetes configs. And then what it's going to do is it's going to take this new config and it's going to merge it with the old config, and it's going to figure out exactly what's changed and it's going to update those properties.

![image](https://github.com/user-attachments/assets/71d07407-b0bc-429a-8eaa-86ae6b371bdb)

## Different Types of Patches
There are 2 different ways you can define a patch
- Json 6902 patch Inline vs Separate File

![image](https://github.com/user-attachments/assets/f8b38611-35a9-4911-b707-db096ec3b664)

- Strategic merge patch Inline vs Separate File

![image](https://github.com/user-attachments/assets/8ccd4917-4448-42a5-801c-2ab3d904a0f8)

## Patches Dictionary

![image](https://github.com/user-attachments/assets/8e882608-32f0-4254-b464-13e5ab9b24d5)

Replace Dictionary Strategic Merge Patch

![image](https://github.com/user-attachments/assets/bc41f741-39ad-4f38-8679-ff528144bb5a)

So we saw how we can replace or modify a key and a dictionary using a JSON 6902 patch. Add Dictionary

![image](https://github.com/user-attachments/assets/7ce90654-a771-45d2-ba7b-5b650f629291)

The same as using Strategic Merge Patch

![image](https://github.com/user-attachments/assets/a08babbc-4982-41d8-9d40-29f608d82d6b)

The same with remove. Using Json6902

![image](https://github.com/user-attachments/assets/31d1f259-2af2-4a1b-bd44-0617d771859e)

Using Strategic Merge Patch

![image](https://github.com/user-attachments/assets/e34014eb-b866-45f9-bacd-c6fe34945940)

## Patches list
Let's now take a look at how we can perform those operations on a list.

We have a zero. So what does this zero mean? 

Well, the number at the end of the path represents the index of the container you wanna change. So remember, this is a list, so you can have more than one container, and the index is going to represent which item in that list you wanna update.

![image](https://github.com/user-attachments/assets/bc261fd9-4d8a-4aa9-a924-ebc8a6935619)

![image](https://github.com/user-attachments/assets/eaed48ae-8ce3-416f-8d31-6f50da30d2c1)

**ADD List**

![image](https://github.com/user-attachments/assets/93c4e380-6310-4aa8-9e76-cda7ffc8c592)

![image](https://github.com/user-attachments/assets/df99d46c-3e60-43df-8fe3-591bd64351df)

**DELETE List**

![image](https://github.com/user-attachments/assets/a2e98c63-1a8c-4d66-9a4e-ad253e58606b)

![image](https://github.com/user-attachments/assets/45d92371-3eba-42cd-a078-6a0821ded32b)

## Overlays

![image](https://github.com/user-attachments/assets/9ce2a4c0-0718-40e5-b6cb-e38e30f3c0be)

So this is going to be where all of your shared configuration, so all of your Kubernetes configs that are gonna be shared across all of your environments or all of your default configs are going to reside. So your base config is going to have all of these shared and default configurations. And then what we're gonna do is on a per-environment basis, we're gonna define a overlay folder for each environment. And here we get to specify all of the environment specific configs so that we can take the base configs and then modify it to our liking for that specific environment. And so that's why we have a folder for dev, and a folder for staging, and a folder for production.

- That's it, so your dev folder is going to have a certain number of patches that will change things to match what the dev should have.
- Staging folder is gonna do the same thing. They're gonna have a bunch of patches for the staging environment, and the same thing goes for production.

![image](https://github.com/user-attachments/assets/4eb7a0b6-1890-4edf-a04e-05fc568b3dbe)

if you see two dots and a slash, that means go up a directory. So we see **../**, so that means we go from the dev folder to the overlays folder, and then we have another **../**, so that means we go from the overlays folder to the k8s folder, and then we go into the base directory. Once that's defined, Kustomize will look for the kustomization.yaml file in the base directory to know all the resources it should be importing.

But in one of these environment folders, in your overlay folders, you can have new configs that weren't defined in your base folder.

![image](https://github.com/user-attachments/assets/0b2228b2-7462-4a7a-a0a3-207bba6468d8)

So in this example, I have a grafana-depl.yaml, and you'll notice that doesn't exist in the base folder. So this means that in my production environment, I'm going to add a Grafana deployment that isn't available in any of my other environments. And so I just wanted to highlight the fact that in these environment-specific folders, you can add as many brand new resources as you want. You don't have to just modify preexisting ones, you can use new ones. And so the way that this would look like in your kustomization.yaml file is, under the resources section, we just import it like a regular file. So we have our bases, we have our patches, and then we import our resources that are in our current folder.

![image](https://github.com/user-attachments/assets/bb2a3c37-a0f8-45f7-9efa-8a72f4af1071)

In fact, kustomize gives you a lot of flexibility in how you wanna structure your Kubernetes configs. So you can break it out into subfolders. So your base folder can be broken out into, you know, extra subdirectories based off of features like we've done before, and you could break it out however you want to. Don't think that you just have to jam everything under the base directory.

If you wanna break it out into features as well, you can do that as well. And remember, these subdirectories don't have to match up with what the base directory is. They can be completely separate. They can follow their own patterns. It's completely up to you, just make sure that you have the resources being imported properly into the proper kustomization.yaml files.

## Components

![image](https://github.com/user-attachments/assets/5e6e8bc4-369d-4fc0-83ac-da7362a3d0bb)

We usually try to avoid copying and pasting things like that. 'Cause a lot of times it leads to some config drift. If you make changes in one, you forget to make changes in the other. So components are just reusable blocks of Kubernetes configs, that's all it is. Don't complicate it, it's just all of the resources for a specific feature, all of the patches, all of the config maps, secrets, and any other Kubernetes related configs associated with the feature.

![image](https://github.com/user-attachments/assets/b873b87e-4e3a-443c-a2ee-21ca88dd301c)

So we're writing out all of our Kubernetes configs for our caching functionality, and we're trying to figure out where exactly do we put it in our folder structure. The first thing you're thinking is, we put it in our base configuration directory. And the problem with doing that is, all three overlays will get the configuration, but remember, only self-hosted and premium should get the caching configuration, we don't want development to get it. So putting it in the base folder is not an option.

So what else can we do? Well, we can copy the caching configuration in both of the premium and self-hosted overlays, and that would work just fine. However, if you make changes in one of the folders, you'd have to remember to make changes in the other one. And let's say we over time decide to add a few more variations to how we can deploy our application. We would have to remember to copy it into all of those. So like most other things like developing any applications, you wanna avoid copying and pasting whenever you can.

So we've got the exact same folder structure with our three overlays and our base configuration, but now we're gonna create an extra folder and we're gonna store all of our components here.

![image](https://github.com/user-attachments/assets/8cc2d719-1bc2-42b1-8582-35d22dcc7178)

They were kind of created so that we could have a reusable block of code that we can apply in several different overlays. So we could take our caching configuration, all the Kubernetes config for that, and create a component, a customized component, and then we can just import that component into all of the different overlays that should be using it.

So inside the caching folder, we're gonna have all of the Kubernetes configs that are associated with the caching feature. So like I said, that's going to be probably like a Redis instance with all of the configurations and secrets for the authentication side of our Redis database, as well as anything else that we need for our Redis database. As well as, we're gonna have a folder for our external database feature. And that's going to contain all of the configs that we need to implement that specific feature. So they're all isolated into their own folder. Then what we can do is, all of the overlays that need the caching functionality, they can just import that component. So they just have one line in their configs that say, "Hey, I want to import caching." And then for all of the overlays that need the database, they can just import the database component.

And all of the logic for the external database functionality sits in the database folder inside the components directory. And that way we get to write it only once and then we could import it into as many overlays as we want.

I'm just creating all of the resources I need to get the external database functionality to work.

![image](https://github.com/user-attachments/assets/49ff223b-3950-48eb-9ede-af0224dfb6a5)

Now, if we take a look at the kustomization.yaml file, the first thing that you'll notice is the apiVersion and the kind are different. Because this is a component, we have to change the version to the one listed there, and the kind to be component, just something to keep in mind.

![image](https://github.com/user-attachments/assets/cc9aa1b1-4786-4055-90e4-74280589d2cd)

![image](https://github.com/user-attachments/assets/231fe54a-81e7-4b75-9940-6611277b0fb9)

I also figured that since we have a database, we probably need to store a secret for the database

So if we take a look at the deployment-patch.yaml file, you can see this is a strategic merge patch. And we're going to update the API-deployment in our base configuration, and we're gonna add a brand new environment variable, and that's going to be the database password.

![image](https://github.com/user-attachments/assets/4f516c80-cd8b-418b-8cba-2b333b285d44)

So we have to import the components that we created.

![image](https://github.com/user-attachments/assets/4968649d-0698-4ae5-acd2-48e1448af520)








