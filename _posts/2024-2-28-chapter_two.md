I got [my bridge classifier](https://huggingface.co/spaces/ianco15/bridges) into production. It can classify images of the Brooklyn, Golden Gate, and Tower bridges.

When I was building the classifier, I noticed some of the code in the fast.ai notebooks doesn't work. It took some digging in the fast.ai forums and stack overflow to get everything working. One example is with the nbdev library. The example code to export your notebook doesn't work but I found that the following code works:
`from nbdev.export import nb_export`
`nb_export('title-of-notebook', './')`
`print('Export Successful')`

Since the provided notebooks can't be run as is, it takes some effort to get everything running but it is extra satisfying when you do. 

I repurchased my membership to paperspace gradient.

Here is a recap of the steps I took to build my classifier.

Using a [GPU-powered notebook](https://console.paperspace.com/t9132vjg9g/notebook/rjc1ez2o31ax9y4?file=%2F02_production.ipynb) on paperspace gradient:
1. Use duckduckgo search to find images of the three bridges, save each each image in a folder for that specific bridge
2. Augment and crop the images to provide extra training examples without needing additional images
3. Train a model to recognize each bridge using the images
4. Create a confusion matrix to see how often our model made incorrect predictions and check the images with the highest loss to see they should be reclassified or removed (there was a javascript error that made reclassifying the images difficult)
5. Retrain the model
6. Export the trained model

Then [on my CPU](https://huggingface.co/spaces/ianco15/bridges/tree/main), I opened an instance of jupyter notebook and:
1. Imported the trained model
2. Created a function that can be used to make predictions on new images
3. Created a Gradio object that referenced the predictions function
4. Exported the code as a python file and pushed it to Gradio

That's all for now. Onto chapter 3!
