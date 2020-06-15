# AutoAIChallenge

## Overview

In this lab, you will see how to create a machine learning model with AutoAI and deploy it as a web service using Watson Machine Learning service. It also covers how to create a machine learning model with jupyter notebooks on cloud and deploy it as a web service using Watson Machine Learning service. 

## Pre-requisites:

1. Create an [IBM Cloud](https://cloud.ibm.com/login) account

## Steps:

1. Go to IBM Cloud, click on catalog.

![](img/1.png)

2. Search for ``Watson Studio`` or find Watson studio under AI services. Click on it.

![](img/2.png)

3. Click on ``create``

![](img/3.png)

4. Click on ``Get Started``

![](img/4.png)

5. Click on ``Create an empty project``

![](img/5.png)

6. Name the project and click on ``Add``

![](img/6.png)

7. It will redirect you to a page that will ask you to instantiate a cloud object storage, click on ``create``

![](img/7.png)

8. Go back to watson studio and click on ``Refresh``

![](img/8.png)

9. Under My Projects, click on the project that you have just created which will look something like this. On the right corner of the screen, click on the icon as shown below.

![](img/9.png)

10. Browse for the dataset on local system which is provided on the git repo.

![](img/10.png)

11. Make sure that the dataset is loaded within the project which will look something as shown below.

![](img/11.png)

12. Click on ``Add to project`` and the click on ``AutoAI experiment`` to add it to your project.

![](img/12.png)

13. Click on ``Associate a Machine Learning service instance`` which will land you in a new tab and then create the service needed. Head back to watson studio and click on ``Reload``

![](img/13.png)

14. When you click on ``Associate a Machine Learning service instance``, it will take you to a page like this.

![](img/14.png)

15. Make sure the machine learning instance is added to the auto ai experiment, it will look something like this.

![](img/15.png)

16. Click on ``Select from project`` and then select the dataset that you have added just now to the project.

![](img/16.png)

17. Click on ``Experiment Settings``

![](img/17.png)

18. Adjust the settings as shown in the image below and click on ``save changes``

![](img/18.png)

19. Click on ``swap view``

![](img/19.png)

20. Once the experiment is run, click on ``save as`` and select ``model``. Head back to project page, you will see a model saved. click on it and click on ``Deployments`` and then click on ``Add Deployment``

![](img/20.png)

21. Check the status of the deployment, wait for it to initialize.

![](img/21.png)

22. Once the initialize stage is finished, click on the deployment and then navigate to ``Implementation``

![](img/22.png)

23. Go to ``Test`` which will have a test deployment, give in the values and click on ``predict`` to see the predictions.

![](img/23.png)


HAPPY CODING!!
