# Sharp binary installation on AWS lambda

## npm install binary
Sharp binaries are installed as per the OS, so doing a npm install on your machine say window/mac will get the window/mac binary for sharp and that wont work on a lambda run on linux
We also assume that you choose x64 and not arm as a architecture for running your lamba
if you choose arm look at installing arch arm binaries for sharp using the *npm install --arch=arm command* 

IMPORTANT - Before you start sharp installation 
- Make sure to sure the node version compatible with AWS Lambda, check the Lambda config settings to find the supported node version*
- Use nvm to manage node versions, much easier and nvm install+use the node version tahts used by the lambda or there will be compatibility issues with the layer 
- To change Lambda Runtime from AWS Lambda console - AWS Console -> Lambda -> function name -> Code Tab -> Scroll down to Runtime settings

`
npm install --arch=x64 --platform=linux --prefix nodejs sharp
`

## Without using Layers
Create a zip file with the downloaded sharp node_modules with any name you like, I named mine as sharpbinnodenn.zip
*nn in the zip file stands for the node version just to identify the sharp binary later with multiple node versions*
I tried not using layers and use uplaod the zip file in the lambda function itself, but then I could not edit my index.js file in the lambda console.
I dont use any cli frameworks for lambda, so i need the aws console to write and test my lambda code, so moving onto adding the sharp binary as a lambda layer

## Adding the sharp binary as a layer to Lambda
Before creating a layer ensure your zip file has the following structure 
- nodejs[folder]
-- package-lock.json
-- node_modules[FOLDER]
Then create a zip file with this structure with any name you like, I just named mine as sharpnode18x.zip
*18x in the zip file stands for the node version just to identify the sharp binary later with multiple node versions, replace 18x with your node version used in the nvm use command*

AWS Console -> lambda -> layers ->create a new custom lambda layer
Give it a name eg 'sharpnode18x', choose the proper runtime eg Node 18.x
Upload the zip file created earlier 'sharpnode18x.zip'
You should now see a layer in the lambda layers called sharpnode18x

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
- I got an index.mjs file in lambda, thats a ES6 file, as I used node 18x as runtime, a require will work in an index.js file 
`
const sharp = require('/opt/nodejs/node_modules/sharp/lib/index.js');
`

## Test
Creating a test event should give success if there are no errors finding the sharp module in the imported path, if there are errors look at the path in the imports
