Meteor on Container Engine

This is a fork of "an initial attempt at a deploy script for running [Meteor](http://meteor.com) on [Google Container Engine](https://cloud.google.com/container-engine/), itself running [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)."

My version allows you to run any meteor application in the cluster.

Assuming you have your google cloud account and tools ready you only have to do two things:

Build your Meteor application.

1) meteor build --architecture=os.linux.x86_64 ./

This will create a .tar.gz with a bundle, that you have to copy to a visible URL. You can use dropbox. 

2) Change the BUNDLE_URL in meteor-controller.json value to this URL.

3) Run the ./setup.sh 

Go grab a coffee.. When you are back - congratulations! you have a running Meteor + MongoDB cluster at the google cloud. 

If you don't have your google cloud ready, you have to:

1) create an account at [google cloud platform](https://cloud.google.com/)
2) Go to "my console". 
3) Add a project.
4) Click on a project name, go to APIs, under the "Google Cloud APIs" click on Compute Engine API, enable it, go back, click "more" and do the same for Container Engine API.

Now go to the console, install the gcloud tool
6) $ curl https://sdk.cloud.google.com | bash

After that you might wanna restart your shell. 

Login to the platform. (Make sure you use the same gmail ccount that you used to create the account at the cloud. If you see weird authentication errors later on in the console, this might be the reason)
7) $ gcloud auth login

Set the project. This is the Project ID that you can find at the top level of your google developer console. 

8)  gcloud config set project PROJECTID

And now you are ready to do the first part of this tutorial. 
