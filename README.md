# 🔴 CircleCi Deploying documentation <sup>[_Udacity Project 3_]</sup>


◾ ◾ ◼️ ⬛ ``` (First Part: How to config CircleCi .circleci/config.yml) ``` ⬛ ◼️ ◾ ◾
---

    version: 2.1
    orbs:
      # orgs contain basc recipes and reproducible actions (install node, aws, etc.)
      node: circleci/node@5.0.2
      eb: circleci/aws-elastic-beanstalk@2.0.1
      aws-cli: circleci/aws-cli@3.1.1
      # different jobs are calles later in the workflows sections
    jobs:

``version:``
**It is the the version of circleci config file, it warns you about deprecations or breaking changes [_Orbs needs Version 2.1_]**

``orbs:``
**It is the main tools you need to deploy your App, you can find more about available orbs _[here](https://circleci.com/developer/orbs)_**

``jobs:``
**Here we're going to define our main jobs that we're going to go through for deploying**

## 📗 `` Jobs: ``

Under the jobs section we can find many actions like **Build** , **Test** etc...

each of this actions needs tasks to be done so it can run successfully, for that we're going now to define the basics of each step
### 🔲 `` Build: ``
It is the first stage we need to go with, to install the orbs and the main packages then building the App.

    build:
      docker:
        - image: "cimg/node:14.15"
      steps:
        - node/install:
            node-version: '14.15'         
        - checkout
        - when:
          condition:
            equal: [main, << pipeline.git.branch >>]
              - run:
                name: Install Front-End Dependencies
                command: |
                  npm run frontend:install

#### `` Docker: ``
Is the used executer to run the needed action with orbs so we use it to run the job steps.

There are many other executers than docker like _machine_ , _macos_ , _windows_  , _shell_ , *resource_class* and others , you can read more about executers _[Here](https://circleci.com/docs/using-docker)_

in our job we're using docker image for node:14.15 so we can install and build our node app

#### `` Steps: ``
Is the required steps to be done to setup/install our apps

- ```- node/install:```  is the first step to install node and choosing it's version
- ```- checkout``` checking that node has been installed successfuly 
- ```- when``` using condetions to check something before running steps
  - ```condetion:``` delcaring the condetions we nned to check before running steps,  i'm using __equal:__ to check the right branch before running.
- ```- run``` running the needed steps to install our dependencies 
	- ```name:``` giving a name for this stage
	- ```command: |``` the commands we're going to use from our root package.json file like _npm run frontend:install_

you will use the `- run` step in the same way for each action like ( __api:install__ , __frontend:build__ and __api:build__)

### 🔲 `` Test: ``
It's the second stage we need to run after building our application to test it.

    test:
      docker:
        - image: "cimg/base:current"
      steps:
        - run:
            name: Test FrontEnd
            command: |
              echo "# TODO: Testing FrontEnd"
              npm run frontend:test

- We need first to choose an executer ( _docker_ ) we can use the current image using `image: cimg@base:current`
- We need to define what scripts to run for the **test** stage into the _steps_, the same as we done before at the **build** stage

### 🔲 `` Deploy: ``
It is the third stage we need to go with, to deploy our code using the CI/CD

    deploy:
      docker:
        - image: "cimg/base:stable"
      steps:
        - node/install:
            node-version: '14.15' 
        - eb/setup
        - aws-cli/setup
        - checkout
        - run:
            name: Deploy App
            # TODO: Install, build, deploy in both apps
            command: |
              npm run deploy

- We need to choose an executre it will be also ( _docker_ ) with stable image ``image: cimg@base:stable``
- We've to define the needed steps agian to install _node_, _elasticbean cli_, _aws-cli_ then the actions we need to run the same as we did at the **build** stage

at this point we can say we made our config file ready, the next step is how we're going to call stages and how it can run

## 🔲 `` Workflows: ``
Here we can now run our predefined actions as stages, in any order we need it to run at.

    workflows:
      udagram:
        jobs:
          - build
          - test
            requires:
              - build
          - hold:
              filters:
                branches:
                  only:
                    - main
              type: approval
              requires:
                - test

- ``udagram`` Is the name of our workflow, so we're going to start our workflow by defining it's name
- ``jobs`` Here we're going to define the order of our predifened stages
  - Under jobs we're going to call our predefined parameters like _build_, _test_ and _deploy_
  - We can run the actions automatically
  - Also we can define some requirements for each case, like the `` - test `` parameter, as you can see it cannot run until the `` - build `` parameter finishs and success,
  - Also in other cases we can use `` - hold `` parameter to stop the process depending on some more actions, like `` filters `` to work only on ``main branch`` of the repo the same as what we use in our example. [ _You can use regEx with filters_ ] e.g ( __branch: /branch-.*/__ )

**You can read more about workflows _[Here](https://circleci.com/docs/workflows#scheduling-a-workflow)_ as there is much more to use with your workflow**


---

◾ ◾ ◼️ ⬛ ``` (Second Part: How to Deploy my App using CircleCi/GitHub ) ```  ⬛ ◼️ ◾ ◾
---

**After configuring your CircleCi, you're now ready to deploy your app, just some more few steps to reach your goal**

1) Firstly we're going to add our App to a GitHub Repo
  - You need to go to your project main folder then open the CLI and use the fellow commands:
  
        git init
        git add .
        git commit -m 'init'
        git remote add origin <GitHub-Repo-URL.git>
        git push -u origin master

  - After adding the Repo to your Github account, wait til the push process finishs.
2) Now we're going to start our CircleCi account by visiting this [URL](https://circleci.com)
3) Click on the upper right button ( Go to Application )
  ![Go to Application](https://user-images.githubusercontent.com/76433966/183743076-1a1c615a-d628-4263-b71d-8fd5db7d7241.png)
4) Connect using Your Github account, and choose repos you need to add to your CircleCi Projects
5) Go to Projects from the left-side panel, and start your setup journey
  ![Setup Porject](https://user-images.githubusercontent.com/76433966/183744280-5ddd2a78-9040-4202-9eb8-b3005fac2e67.png)
6) Now select your ``config.yml`` file if you already uploaded at your project main or create a template using the CircleCi config.yml editor
  ![Select config.yml](https://user-images.githubusercontent.com/76433966/183745088-67c8f4dd-2618-40a6-8c7c-a1b5b3e7618a.png)
7) Now We've added our App to CircleCi, it might start deploying if you already added your ```config.yml```, and you'll be redirect to this screen
  ![Process Added](https://user-images.githubusercontent.com/76433966/183746718-4a926c09-5f1a-49f2-bb9d-8a11d1710fa1.png)
8) We've to add the environment variables now to our project so it can be passed later into the deploying process
  - First click on the Project settings 
  - ![Settings](https://user-images.githubusercontent.com/76433966/183747289-14ff604e-9976-4f17-87a6-5d0f1e947c26.png)
  - Then choose the Environment Variables
  - ![Env](https://user-images.githubusercontent.com/76433966/183747682-f6187d14-d421-4032-aab2-dd2c7e77cd6e.png)
  - Then click on ```Add environment variable``` And put all your environment variables in it, you can also import them from another Project
  - ![ADD Env](https://user-images.githubusercontent.com/76433966/183748363-0e02af2b-aabc-4571-bb07-d4b2fb319552.png)

almost DONE, now you have passed almost everything, it should now work and success if you have done everything correctly.
otherwise you can click on the failure and check the reason to fix it, and after fix you can re-run the same process from where it failed or from scratch

---
Finally, i wish if this fast walk through can help you to find your answers, and if you need further help just send me a direct message through _[WhatsApp](https://wa.me/201102423786?msg=About+CircleCi)_ or _[Messenger](https://m.me/bori0o)_
