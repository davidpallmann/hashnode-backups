# Hello, Cloud9!

#### This episode: AWS Cloud9 and using a cloud IDE. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Cloud9 and show how to use it for .NET on AWS development. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# AWS Cloud9 : What is it, and why use It?

> "Cloud nine gets all the publicity, but cloud eight actually is cheaper, less crowded, and has a better view." —G. Carlin

> "Every great platform has a great IDE." —Werner Vogels, CTO, Amazon

Your local machine isn't always the best choice for development and testing. For example, if you work on a Windows laptop but plan to deploy your .NET code Linux, you might want to code and test on a Linux machine.

[AWS Cloud9](https://aws.amazon.com/cloud9/) is a cloud-based Integrated Development Environment (IDE). AWS describes it as "A cloud IDE for writing, running, and debugging code". You can use it code your application, run it, and test it, using just a browser.

You might choose Cloud9 for any of these reasons:

*   You want to work on a different operating system than your local machine.
    
*   You need a more powerful class of machine to test your code.
    
*   You need to get at your IDE from anywhere there's a browser.
    
*   You want to collaborate with others in your IDE.
    
*   Local machine development is disallowed at your organization.
    

Cloud9's browser-based IDE is full-featured. It includes a code editor, debugging, and a full terminal. Although you'll need to install .NET on the instance, the code editor has support for C# syntax highlighting. You can customize editor theme and keystroke bindings. There's also a built-in image editor. Cloud9 tracks file revision history, allowing you to reference past code changes. Cloud9 also integrates with AWS CodeStart, which you can leverage for continuous integration.

You can collaborate with others on Cloud9 by sharing the development environment with your team. You'll see each other type in real-time, and you can chat in the IDE. [image source](https://aws.amazon.com/cloud9/)

![real-time-collab.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668984852583/H5R2wGme0.png align="left")

The Cloud9 terminal windows lets you interact with your EC2 instance naturally with sudo privileges. You can run your usual dotnet commands here to create projects from templates, build your code, and run applications. If you're a Linux newbie, this is a great way for .NET developers to start getting familiar with it.

Cloud9 includes a browser tab you can use to test your web applications. There's also a local testing facility for building and debugging serverless applications.

With Cloud9 you pay only for the EC2 instance and EBS volume for your environment; Cloud9 does not bring any additional charges. You are charged the normal rates for AWS resources. See the [AWS Cloud9 Pricing page](https://aws.amazon.com/cloud9/pricing/) for details.

# Our Hello, Cloud9 Project

In this tutorial, you'll create a Cloud9 suitable for modern .NET development on AWS. Then, you'll create a webapi project and test it.

![aws-new-ide-open.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669042333722/go_rL-Bev.png align="left")

[source code](https://github.com/davidpallmann/%5Blink%5D)

## One-time Setup

You will need the following:

1.  An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/).
    

## Step 1: Create Cloud9 Environment

In this step, you'll set up a Cloud9 environment for NET 7 development.

1.  Sign in to the AWS console. At top right, select the region you want to work in. You can check supported regions for Cloud9 on the [AWS Regional Services List page](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/). I'm using **us-west-2 (Oregon)**.
    
2.  Navigate to **Cloud9** and click **Create environment**.
    
    A. Give the environment a name, such as **Test**.
    
    B. On the New EC2 Instance, choose the EC2 instance size to create. For getting familiar, I recommend the default free tier t2.micro instance. If you choose a larger size, be mindful of the rate you will be charged.
    
    ![aws-create-name.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668982779442/GGaD7gzt6.png align="left")
    
    C. Click **Create**.
    
    D. Wait for the environment to be created, which will take several minutes.
    
    ![aws-creating.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668982907528/z3z0Cl23N.png align="left")
    
    Note: If environment creation fails because the t2.micro instance type is not available in the region, repeat the above steps and select a different small instance type, such as t3-small.
    

## Step 2: Get Familiar with the IDE

In this step, you'll get familiar with the IDE.

1.  In the AWS console, on the AWS Cloud9 &gt; Environments page, click the **Open** link next to your environment.
    
    ![aws-click-open.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668985687369/8aLMV0-r7.png align="left")
    
2.  If AWS needs to start up your EC2 instance, you'll see a message like this and have a brief wait.
    
    ![aws-starting-instance.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668985761814/d1lvSYFdf.png align="left")
    
    Soon, you'll be in your Cloud9 environment.
    
    ![aws-new-ide-open.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668985831937/1TFdsbN3T.png align="left")
    
3.  Get familiar with the environment.
    
    Take a look around. Most of the display is a Welcome tab. You'll be able to open up other tabs for editing code and testing your app. On the Welcome tab, you have links to the documentation, and can configure preferences such as theme.
    
    Below the Welcome tab is a terminal window with the Bash shell, and a splitter bar you can use to change its size. Run the command **hostnamectl** to see the Linux distribution and version this instance is running, which for me is Amazon Linux 2, kernel 4.14.296-222.539.amzn2.x86\_64.
    
    ![hostnamectl.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668986544282/OLuEJ2hze.png align="left")
    
    There's a menu at top, and Preview and Run buttons. The Run button lets you run or debug your application. The preview button lets you test your application in a browser tab.
    
    There are icons on the left and right sides of the screen that open different panes. A pane already open on the left side is a file explorer. Later, we'll also open up an AWS Explorer.
    
    ![ide-file-explorer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668987657895/-llNN4imf.png align="left")
    
4.  Configure your AWS account. In the Bash terminal window, run AWS configure and configure it for your AWS account. If you don't already have an access key ID and secret key, you can generate credential keys for your user in the AWS Console, under **IAM &gt; Users**.
    
    ```plaintext
aws configure
```


5. On the left side of the screen, click the AWS logo. AWS explorer will open. Confirm you can view your AWS resources.

    ![aws-explorer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669043084510/1dSf9-9qp.png align="left")

## Step 3. Install the .NET SDK

In this step, you'll run commands in the Bash terminal window to install the .NET SDK and runtimes, and to update environment variables. We'll be following the command documented at [.NET Core sample for AWS Cloud9](https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-dotnetcore.html), Steps 1 and 2.

1. Check whether dotnet is already installed by running **dotnet --version**. If it is and has the version you want to work with, you can skill the rest of this step.

    ```bash
dotnet --version
```

2.  Run the commands below to ensure the latest security updates are applied.
    
    ```plaintext
sudo yum -y update sudo yum -y install libunwind
```

    ![bash-sudo-yum-update.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668988682741/pU_BVsti-.png align="left")

    ![bash-sudo-yum-install.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668988749084/RzIhr-VCk.png align="left")

3. Run the command below to download the .NET Code SDK installer script.

   ```bash
wget https://dot.net/v1/dotnet-install.sh
```

    ![bash-wget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668988802423/3b0FWRzNX.png align="left")

4.  Run the chmod command below to make the installer script executable.
    
    ```plaintext
sudo chmod u=rx dotnet-install.sh
```

5. Run the installer script. If you want the latest .NET (.NET 7 as of this writing), use the command below:

    ```bash
./dotnet-install.sh -c Current
```

 If you prefer to install the latest Long-Term Support (LTS) release (.NET 6), run this command instead:

    ```bash
./dotnet-install.sh -c LTS
```

    ![bash-dotnet-install.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668989020104/dBS1UVMrz.png align="left")

5. Add the .NET SDK to your path. Run the command below to open the file ~/.bashsrc in the vi editor.

   ```bash
vi ~/.bashrc
```

    A. Use the down arrow key to move to the line that starts with **export PATH**.

    B. Press your INSERT key. You'll see **-- INSERT --** displayed at the bottom left.    

    C. Use the right arrow or $ key to move to the end of the line.

    D. Add the following to the end of the line: **:$HOME/.dotnet:$HOME/.dotnet/tools**

    E. On the next line, insert this line: **export DOTNET_ROOT=$HOME/.dotnet**.

    F. Save the file, by pressing ESCAPE and then **:wq** [ENTER].

    ![bash-vi.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668989111601/Y3VB7Lw-5.png align="left")

6.  Run the command to load the .NET SDK by sourcing the .bashsrc file:
    
    ```bash
. ~/.bashrc
```

7. Run the dotnet --info command to confirm dotnet is installed.

    ![bash-dotnet-info.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668989635427/2igLGiGP4.png align="left")

8. Install the .NET deploy tools:

    ```bash
dotnet tool install -g aws.deploy.tools
```

    ![bash-dotnet-tools-install.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669043376959/tPBaSHuQv.png align="left")

That was a bit to do, but .NET is installed and you are now equipped for .NET development on Cloud9.

## Step 4. Create a .NET Web API Project

In this step, you'll create a .NET WebAPI project with the dotnet new commanf.

1.  In the Bash terminal window, run the commands below to create a development folder and cd to it. Notice the dev folder is now visible in the File Explorer at top left.
    
    ```bash
mkdir dev cd dev
```

     ![mkdir.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669046036898/GQRbtYQ8x.png align="left")

2. Run the dotnet new command below to generate a WebAPI project. You see generated project files and folders in the File Explorer.

     ```bash
dotnet new webapi -n HelloCloud9
```

    ![bash-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668989954295/GpgvezA2k.png align="left")

3.  CD to the HelloCloud9 folder.

    ```bash    
cd HelloCloud9
```

4. In the File Explorer, expand the folders and double-click **dev/HelloCloud9/Controllers/WeatherForecast.cs**. Now you are editing the C# code in the Cloud9 IDE.

    ![ide-edit-controller.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668990317576/r-VfJ6fq5.png align="left")

    If you like, make changes to the code or data. This is optional. If you do make changes, click Ctrl+S to save them.

5. In the File Explorer, double-click **appsettings.json**. The file opens in another IDE code editor tab. Add the **Kestrel** section below, and press Ctrl+S to save your changes. We're doing this because Cloud9 can only run applications on HTTP over port 8080, 8081, or 8082, with the IP address of 127.0.0.1, localhost, or 0.0.0.0.

   ```JSON
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:8080"
      }
    }
  }
}
```

    ![ide-edit-settings.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668990758965/73bJLH9Zo.png align="left")

## Step 5: Build, Run, and Test the Web API Project

In this step, you'll build your project, run it, and test that it responds.

1.  In the Bash terminal window, run dotnet build and dotnet run to build and run the application. You see confirmation the service is running on HTTP over port 8080.
    
    ![bash-dotnet-run.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668991049600/gxbeWvLlg.png align="left")
    
2.  At the top of the screen, right click **Run** and select **New run configuration**.
    
    A. In the tab that opens, set Command to **dotnet run**.
    
    B. On the right side of the tab, click **CWD** and select the **HelloCloud9** folder.
    
    C. Click the Run button on the left. The dotnet run command executes, and you see confirmation the service is listening at http://localhost:8080.
    
3.  Let's test the API from another terminal window. To the right of lower set of tabs where the Bash terminal window is, click the Plus sign (+) and select **New Terminal**. A new terminal window tab opens.
    
    ![bash-new-terminal.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668991154155/HLqyjbEsM.png align="left")
    
4.  In the new terminal window, run the curl command below to test accessing the service.

    ```plaintext
curl -v http://localhost:8080/WeatherForecast
```

    ![bash-curl.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668992439412/TMHZ5J3Wc.png align="left")

    You see the output from the API displayed. Congratulations! You've used Cloud9 to develop and run a .NET application.

## Step 6: Deploy to AWS

Now we can deploy our Web API project to AWS.

1. In the Bash terminal window, run the dotnet aws deploy command.

    ```bash
dotnet aws deploy
```

2.  Respond to the prompts. Choose a new cloud application deployment and select a destination service. I selected AWS App Runner.
    
    ![bash-dotnet-aws-deploy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669043713271/bsux2IVcK.png align="left")
    
3.  Complete the wizard, accepting defaults.
    
4.  Wait for deployment to complete, and note the endpoint URL.
    
5.  Refresh the AWS Explorer, and you will see you HelloCloud9 App Runner service listed.
    
6.  In a browser tab, visit the endpoint URL with **/WeatherForecast** at the end of the path.
    

## Step 7: Shut it Down

When you are done with this tutorial, deallocate the AWS resources. You don't want to be charged for something you're not using.

1.  In the AWS console, navigate to **App Runner** and delete the HelloCloud9 service.
    
2.  In the AWS console, navigate to **Cloud9** and delete the Test environment.
    

# Where To Go From Here

AWS Cloud9 is a powerful, easy to use cloud-based IDE. For the .NET developer, it allows you to code and test modern .NET apps on Linux, and get familiar with Linux. In this tutorial, you created a Cloud9 environment, learn a little bit about it, and installed .NET. You generated, edited, built, ran, and tested the app locally. Then, you deployed it to AWS.

To go further, learn more about Cloud9 by reviewing the documentation and other resources linked below.

# Further Reading

AWS Documentation

[AWS Cloud9](https://aws.amazon.com/cloud9/)

[AWS Cloud9 Documentation](https://docs.aws.amazon.com/cloud9/index.html)

[AWS Cloud9 User Guide](https://docs.aws.amazon.com/cloud9/latest/user-guide/welcome.html)

[Setting up AWS Cloud9](https://docs.aws.amazon.com/cloud9/latest/user-guide/setting-up.html)

[Previewing running applications in the AWS Cloud9 Integrated Development Environment (IDE)](https://docs.aws.amazon.com/cloud9/latest/user-guide/app-preview.html)

\[tutorial-link\]

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

\[video-link\]

[Introducing AWS Cloud9 - AWS re:Invent 2017](https://youtu.be/fwFoU_Wb-fU)

Blogs

\[blog-link\]

[Building .NET Core Apps on Cloud9](https://blog.todotnet.com/2017/06/building-net-core-apps-on-cloud9/)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)