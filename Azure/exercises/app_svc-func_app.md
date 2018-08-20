# Azure App Services and Function Apps

## Exercise 1

1. Create a new **App Service plan** with the Linux operating system. Use "B1 Basic" pricing tier for your service plan.
2. Then scale up your Service plan to "S1". Also, scale out your service plan to 2 instances.
3. Evaluate the autoscaling options in your service plan and compare it with VM scale set.
4. Create a Web App in **App Services** with Linux OS. Choose PHP 7.2 as the runtime stack and the service plan created earlier.
5. Set the deployment credentials on your web app, connect to FTP session of your application and browse the files available.
6. Enable local git repository in deployment options, git clone the app in your local machine and add a hello world PHP app with name "index.php" in the git repository.
7. Commit and push your changes. Finally, navigate the browser to the app and check if it displays the page as expected.
8. Navigate and browse the repository folder in your app using any FTP client.
9. Connect to app instance using SSH in Azure portal, locate your hello world index.php file and modify its content.
10. Check the changes by reloading the app url in your browser.

### Results

- Step 3

???
![](/Azure/images/app-ex1-autoscale.png)

- Step 4

The Web App Service can be in a different resource group than the App Service Plan.

- Step 5

![](/Azure/images/app-ex1-depcred.png)

- Step 6 and 7

![](/Azure/images/app-ex1-git.png)

```bash
$ git clone https://hp@ns-webapp1.scm.azurewebsites.net:443/ns-webapp1.git
Cloning into 'ns-webapp1'...
Password for 'https://hp@ns-webapp1.scm.azurewebsites.net:443':
warning: You appear to have cloned an empty repository.
$ # create a file and commit
$ git push
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 258 bytes | 258.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '05e7f324eb'.
remote: Generating deployment script.
remote: Generating deployment script for Web Site
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling Basic Web Site deployment.
remote: Kudu sync from: '/home/site/repository' to: '/home/site/wwwroot'
remote: Copying file: 'index.html'
remote: Deleting file: 'hostingstart.html'
remote: Ignoring: .git
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Deployment successful.
remote: App container will begin restart within 10 seconds.
To https://ns-webapp1.scm.azurewebsites.net:443/ns-webapp1.git
 * [new branch]      master -> master
```

- Step 9 and 10
  - File is found at `/home/site/wwwroot`. Changes in the file contents reflects in the browser.

![](/Azure/images/app-ex1-ssh.png)

## Exercise 2

1. Create a new App service plan with the Windows operating system. Use "S1" pricing tier for your app service plan.
2. Create a HTML5 Empty web app in App services. Choose the app service plan created earlier.
3. Create a new repository in GitHub / BitBucket and a Hello World "index.html" file to the repository.
4. In Deployment options, disconnect any existing source and add your new repository as the new source.
5. Disable FTP access to your application.
6. Run a performance test in your application with 25 concurrent users for a period of 1 minute.
7. Find the list of running process and their memory usage. Enable application logging on the filesystem.
8. Create a new deployment slot named "dev" and use the existing app for cloning.
9. Make a change to your html file and publish it on the new slot.
10. Then swap your slot with production and check if new changes are visible on the applications url.

### Results

- Step 5

![](/Azure/images/app-ex2-disable.png)

- Step 7
  - Go to _Process explorer_ within the App Service.

**List of running processes**
![](/Azure/images/app-ex2-proc.png)

**Enable Application logging**
![](/Azure/images/app-ex2-applog.png)

- Step 8 and 9

???
  - If the html file is changed in the same branch on the repo, the production slot also gets updated with the new file automatically. To simulate an actual scenario, use follow the below steps.
    - Go to _Deployment slots_ within the App Service.
    - Choose `Don't clone configuration from an existing slot`.
    - Click on the newly created slot.
    - Configure _Deployment options_ for the same Github repo but a different branch like a dev branch for instance on which a different version of the html file is present.

## Exercise 3

1. Create a new storage account and create two blob containers in the storage account.
2. Create a new function app with windows OS and consumption hosting plan.
3. Create a new function that uses changes in azure blobs as a trigger. Use the first container of the storage account as the trigger to this function.
4. Whenever a file is added to the first container, this function should get invoked and the function should copy the blob file to the second container.
5. Create another function that runs on a periodic interval of 15 minutes. When invoked, this function should delete all the files in both the containers.
6. Write a third function that has HTTP as the trigger. When the endpoint is called, it should return a list of files in both the containers.
7. Create a proxy for the root path of the application that invokes and proxies the HTTP triggered function.
8. View the invocations and other real time metrics in application gateway.

### Results

- Step 2
  - Go to App Services > Web > Function App

- Step 3 and 4

  - Go to Function Apps > [Your function app] and click on the '+' to create a new Function.

  ...
