## Introduction to Layers in AWS Lambda Functions


> 
This article has been originally published on  [CodingSight](https://codingsight.com/introduction-to-layers-in-aws-lambda-functions/).

AWS Lambda is an event-driven, serverless cloud computing service provided by Amazon, a part of the Amazon Web Services. It responds to multiple events like uploading files to S3, clicks on websites, etc. Then, it takes action based on the trigger.

AWS Lambda supports multiple programming languages, such as Node, Python, C\# (_using dot net core_), etc. For each of them, multiple runtime versions are supported. For example, it is possible to develop Python applications that target multiple versions of the runtime (3.6, 3.7, and 3.8).

Lambda functions are also known as serverless, as you do not need provisioning any infrastructure to run your applications. You can just write the code, deploy it to Lambda, and get it executed.

This article will discuss the packaging and deployment of custom libraries. Also, it will highlight the efficient usage of Layers in Lambda to reuse those libraries. We'll focus on concepts explained by Python, but the approaches apply to other programming languages supported by Lambda as well.

## Packaging Applications

When you write code for cloud-based applications, you might want to include several libraries to refer to them within the projects. They can be standard python libraries like Pandas, NumPy, etc., or custom-tailored libraries required for your project. AWS Lambda does not provide these libraries out of the box. Therefore, programmers need to manage their own dependencies while deploying codes to Lambda Functions.

AWS Lambda supports two ways to achieve this:

1. Manage dependent libraries within the directory of each project.
2. Use Layers to create a reusable library that can be referred to while deploying projects.

This article is going to explore the second way, i.e., deploying code by using Layers in AWS Lambda.

## What are Layers in AWS Lambda

So far, we have talked about packaging and deploying applications. Now, let us learn what Layers exactly are and what purpose they serve in AWS lambda.

Suppose you are developing an application in Python that uses the [Pandas](https://pandas.pydata.org/) library. While you are writing your code on your development machine, you might use some virtual environment or global libraries present on that machine. The code works fine on your machine because you can refer to the libraries. It is possible to execute the application.

Problems arise when you deploy your code chunk to AWS Lambda. By default, Lambda does not provide any environment to host the Pandas library which your code refers to. As a result, when you execute the code, you'll get an error saying that the Pandas module is nowhere to be found.

![Sample code to use the Pandas library in python](http://codingsight.com/wp-content/uploads/2021/05/image-76.png)
_Figure 1 -- Sample code to use the Pandas library in python_

When you execute the above, there is the following error:

![Execution Results - Pandas not found an error](http://codingsight.com/wp-content/uploads/2021/05/image-77.png)
_Figure 2 -- Execution Results -- Pandas not found an error_

As you can see, the Pandas module is not found in the runtime. Hence, an exception takes place.

## How to create a Layer in AWS Lambda

To overcome this issue, we need to provide the runtime with a valid distribution of the library along with its dependencies. This way, our code can refer to it during execution.

For running Pandas, we need to use the following libraries while creating the Layer in Lambda:

1. **Pandas** -- the official Pandas library from PyPi.
2. **NumPy** -- is required to support Pandas.
3. **PyTz** -- the library required by Pandas to handle the time zone-related data.

You can download all libraries from the official PyPi repository. Notice that since Amazon runs the Lambda containers on Amazon Linux distributions, we must download Linux versions of the libraries.

The necessary files are as follows.

1. [pandas-1.2.4-cp38-cp38-manylinux1\_i686.whl ](https://files.pythonhosted.org/packages/13/2c/5a8afc50cd2508905c23322c1d80dceb640717ce93f47f52aa4a4742553c/pandas-1.2.4-cp38-cp38-manylinux1_i686.whl)(9.4 MB)
2. [numpy-1.20.2-cp38-cp38-manylinux1\_x86\_64.whl ](https://files.pythonhosted.org/packages/fd/67/ea80d7f693a027854e34a44a4c3e91e0fcdfaa5e8283e7b9c9ee0056c09f/numpy-1.20.2-cp38-cp38-manylinux1_x86_64.whl)(13.7 MB)
3. [pytz-2021.1-py2.py3-none-any.whl ](https://files.pythonhosted.org/packages/70/94/784178ca5dd892a98f113cdd923372024dc04b8d40abe77ca76b5fb90ca6/pytz-2021.1-py2.py3-none-any.whl)(510.8 kB)

Once you download these files, rename the extension to ZIP and extract them:

![ Renaming the WHL files and extracting the contents to folders](http://codingsight.com/wp-content/uploads/2021/05/image-78.png)
_Figure 3 -- Renaming the WHL files and extracting the contents to folders_

After extracting the libraries to their folders, we must create a new folder named **_python_**. Then we add the library contents to the **_python_** folder.

Add the contents of all three libraries to the **_python_** directory:

![Moving the contents of each library to the python folder](http://codingsight.com/wp-content/uploads/2021/05/image-79.png)
_Figure 4 -- Moving the contents of each library to the _**python **_folder_

As you can see, all libraries are present in the same **_python_** folder. The next step is to compress the **_python_** folder to **_python.zip_**. At this point, the compressed _zip_ folder contains all the necessary files required by the Pandas library to run on AWS Lambda.

### Creating the Layer

When we have our dependencies compressed to an archive file, we can create a layer in AWS Lambda and upload the compressed folder. Then Lambda functions can use this Layer as a reference and execute the code upon it.

In the AWS console, open Lambda and click **_Layers_** on the left. Then click **_Create Layer_** and provide the following details:

* **Name** is the name of your Lambda layer. Use a short descriptive name.
* **The description** is an optional field, but you can provide a long text description to specify which details the layer can handle.
* **Upload Files** -- you can do it in two ways, to upload a file to the layer directly, or to upload files to the S3 bucket and then provide the file URI. For simplicity, we will proceed with the first option.
* **Compatible Runtimes** is also an optional field. However, you can specify the runtimes that are compatible with the libraries in the layer.
* **License** is optional. If applied, it specifies the license required to access the packages in the layer.

![Creating a Layer in AWS Lambda](http://codingsight.com/wp-content/uploads/2021/05/image-80.png)
_Figure 5 -- Creating a Layer in AWS Lambda_

Once you have provided all the details, click **_Create_** -- the layer will be created and ready to use. The process might take some time depending on your compressed file size. When the Layer is ready, you will see the confirmation as below:

![ Layer created in AWS Lambda](http://codingsight.com/wp-content/uploads/2021/05/image-81.png)
_Figure 6 -- Layer created in AWS Lambda_

## Using Layers in Lambda

So, we have our Layer created and ready to use. Let us start using it.

Navigate to the Lambda function you have created earlier and click **_Layers_**. There is an option to add layers to the existing Lambda function. Click **_Add a Layer_**, and you will see the page to add a new layer created earlier.

![Choosing the layer in AWS Lambda](http://codingsight.com/wp-content/uploads/2021/05/image-82.png)
_Figure 7 -- Choosing the layer in AWS Lambda_

We have selected **_Custom Layers_** since it was not provided by AWS. Then we selected the Layer name and version.

Click on **_Add_**, and the layer will be included in the Lambda Function. After that, it will execute without any errors.

![Lambda executed without error](http://codingsight.com/wp-content/uploads/2021/05/image-83.png)
_Figure 8 -- Lambda executed without error_

## Conclusion

We have understood where and how we can use the AWS Lambda Function. Also, we have explored deploying the Lambda code using a Layer.

For those Lambda functions that use compressed deployment packages, you can use layers to manage the dependencies. The code size gets smaller, as the dependencies will be handled by the layer.

The Lambda Layer can be associated with custom libraries, multiple runtime versions, and other dependencies.

To learn more about working with AWS Lambda Layers, you can use the [official reference](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html).