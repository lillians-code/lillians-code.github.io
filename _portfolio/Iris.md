---
title: "Iris prediction using a Deep Neural Network"
excerpt: "A story of persistence and deployment frustrations"
date: 2024-10-30T20:36:52+00:00
header:
  image: /assets/images/iris/iris.jpg
  teaser: assets/images/iris/iris.png
---
[Launch Iris predictor](https://iris.lillian-zhang.com/){: .btn .btn--success} [See source code](https://github.com/lillians-code/iris-dnn){: .btn .btn--info}

The goal of this project was to have some fun building a deep neural network and to build and deploy a Flask app frontend for the model prediction inputs. 

### Goals
1. Build a deep neural network using tensorflow
2. Build a half decent Flask app
3. Host app

### Learnings
1. Web dev is really hard and something I could (and should) improve on.
2. Persistence is not optional. Try, try, try again.
3. PythonAnywhere has really outdated libraries out of the box - and a really limited capacity on the free tier, making it very difficult to install more recent versions of these libraries. 

### Build a deep neural network using tensorflow
For this project I decided to use the classic Iris dataset as I had never attempted a project with this data before. No data cleaning to be done with this dataset, but I did plot it in a 3 dimensional space, reducing the 4 features to 3 first using principle component analysis (see below). 

This neural network has a **train accuracy of 99% and a test accuracy of 98%**.

<html>
<head><meta charset="utf-8" /></head>
<body>
    <div>                        <script type="text/javascript">window.PlotlyConfig = {MathJaxConfig: 'local'};</script>
        <script type="text/javascript">/**
* plotly.js v2.32.0
* Copyright 2012-2024, Plotly, Inc.
* All rights reserved.
* Licensed under the MIT license
*/
/*! For license information please see plotly.min.js.LICENSE.txt */
</body>
</html>

### Build a half decent Flask app
This was quite fun, although my app looked terrible to start with as it had just the 4 input form fields and a submit button, with the picture of the species of iris at the top. I enlisted the help of ChatGPT to prettify the page and generate CSS for me to make the page responsive and create form validations etc.

### Host app
This was harder than expected to say the least. I had initially planned to use PythonAnywhere to host, however the free tier on PythonAnywhere only comes with tensorflow 2.9.0 pre-installed which doesn't support pickling and unpickling model files. Setting up a venv and installing a newer version of tensorflow also did not work, as the free tier doesn't come with enough storage to install tensorflow 2.18.0 (which is a whopping 600mb+).

I decided to change my approach at this point because I'm not prepared to spend real dollars to make this happen. After a day of fiddling around with nginx and gunicorn config, I managed to deploy my app on an AWS EC2 instance [following this article], and set it up to link to my custom subdomain using [this other article], with some debugging (read: a lot of debugging) here and there where it all fell apart and the issues weren't covered by these articles or Stackoverflow.

[Launch Iris predictor](https://iris.lillian-zhang.com/){: .btn .btn--success} [See source code](https://github.com/lillians-code/iris-dnn){: .btn .btn--info}


[following this article]: https://plainenglish.io/community/deploying-a-flask-application-on-ec2-dab5d3

[this other article]: https://medium.com/@kawsarlog/from-flask-to-live-deploying-your-app-with-nginx-gunicorn-ssl-and-custom-domain-1e8b57709fc0