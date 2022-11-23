# Sharp binary installation on AWS lambda

## npm install binary
Sharp binaries are installed as per the OS, so doing a npm install on your machine say window/mac will get the window/mac binary for sharp and that wont work on a lambda run on linux
We also assume that you choose x64 and not arm as a architecture for running your lamba
if you choose arm look at arch arm binaries for sharp 

`
npm install --arch=x64 --platform=linux --prefix nodejs sharp
`
Notes 
- Make sure to sure the node version compatible with AWS Lambda, check the Lambda config settings to find the supported node version*
- My sharp isntallation asked for a 14.x node, so i used nvm to install a 14.x version of node and changed my lambda runtime to 14.x
- Use nvm to manage node versions, much easier 

## Create a zip file
Create a zip file with the downloaded sharp node_modules say sharpbin.zip

## Without using Layers
I tried not using layers and use uplaod the zip file in the lambda function itself, but then I could not edit my index.mjs file in the lambda console.
I dont use any cli frameworks for lambda, so i need the aws console to write and test my lambda code.

## Adding the sharp binary as a layer to Lambda
Before creating a layer ensure your zip file has the following structure 
nodejs[folder]
- package-lock.json
- node_modules[FOLDER]
Then create a zip file with this structure eg sharpnode14x.zip

AWS Console, lambda, layers, create a new custom lambda layer
Give it a name eg 'sharpnode14x', choose the proper runtime eg Node 14.x
Upload the zip file created earlier 'sharpnode14x.zip'
You should now see a layer in the lambda layers

Layers help reuse the binaries/functions across lambda functions so this should prove helpful if we want to use sharp in any otehr lambda functions.

## Using the created Layer in our lambda function
On the lambda function page, you should see a layers icon along with API-Gateway and destination, click on the layers icon.
Add a layer, Custom layers, select the layer we just added 'sharpnode14x', select a version eg 1 and we should see 'Layers 1' in the lambda function

## Importing sharp in our lambda function
In our lambda index.mjs file 
`
import sharp from '/opt/nodejs/node_modules/sharp/lib/index.js';
`
Notes -
- Figure out the absolute import path by looking into the package folder in node_modules 
- I got an index.mjs file in lambda, thats a ES6 file, a require may work in ES index.js 

## Test
Creating a test event should give success if there are no errors finding the sharp module in the imported path, if there are errors look at the path in the imports
