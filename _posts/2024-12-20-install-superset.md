---
title: "Install Apache Superset"
date: 2024-12-20
categories: [Apache Superset]
tags: [Analytics, Visualization, Software]
---

# Apache Superset as Python Virtual Environment on Windows

Software installation can occasionally require a little more effort than what the documentation suggests, even when it is accessible. Installing software does not follow a precise, consecutive process. In this instance, I discovered that installing SuperSet was a little more involved, particularly when using Windows as a Python virtual environment.

Apache SuperSet is an open source tool for data analytics and visualization. With this software, anyone can show and analyze their own data in an elegant way. In this post, I will attempt to provide a slightly more detailed explanation of how to install Superset as a Python environment on a Windows machine. It should be easy for anyone to follow and comprehend from beginning to end.

Apache Superset's own website has [instructions for installing](https://superset.apache.org/docs/installation/pypi#python-virtual-environment) it as a Python virtual machine, but let's add more information from beginning to end.


## Install Python Software

For this, let's install Python 3.11 on a Windows machine. I like to install applications in Windows as chocolatey packages because it makes managing all of my software much simpler. You can omit this step if you already have Python 3.6 or above. 

Lets install chocolatey from this [website](https://chocolatey.org/install). Just copy the CLI command and run it in your powershell.

For example.
```batch
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Now, install python 3.11 with this command.
```batch
choco install -y python311
```
Your python should now be up and running.

## Create a Python Virtual Environment

First, install the virtualenv package.
```python
pip install virtualenv
```

Then, `cd` to a folder you want to work on and create the virtual environment. I will call mine `superset`.
```python
python -m venv superset
```

To activate the python virtual environment on Windows, run the command.
```batch
.\superset\Scripts\activate
```

You should see the venv name prefix in your CLI like below.
```
(superset) PS C:\Development\Superset>
```
![CLI View](/assets/img/posts/2024-12-20-install-superset/superset-activate-cli.png){: width="600"}

> **_Remember_**: You are inside the venv, if you see your venv name prefix in your CLI.
{: .prompt-tip }

> **_Note_**: You can deactivate the virtual environment by simply running `deactivate` inside the venv.
{: .prompt-tip }


## Install Apache SuperSet

Inside your venv, install the superset package. You have venv activated if you see the venv name prefix in your CLI, if not see previous step.
```
pip install apache-superset
```

After installation, you must define a few required configurations. As mentioned in the introduction, I assume you are using a Windows machine; as a result, we will need to manually change a few files to add the necessary configurations.

If your venv is also called `superset`, you will notice several `activate` named files with different file types within the `\superset\Scripts` folder. After the final keyword command *(usually toward the bottom of the page)*, add the following lines to each one after opening it in a text editor. You can use any string as the secret key as long as it remains constant.

For `activate` bash file *(this is mostly for Linux but lets update it anyway)*:
```
export SUPERSET_SECRET_KEY=YOUR-SECRET-KEY
export FLASK_APP=superset
```

For `activate.bat` batch file:
```
set SUPERSET_SECRET_KEY=YOUR-SECRET-KEY
set FLASK_APP=superset
```

For `Activate.ps1` powershell file:
```
$Env:SUPERSET_SECRET_KEY = "YOUR-SECRET-KEY"
$Env:FLASK_APP = "superset"
```

Reactivate you venv to make sure the following environment variables are set. Run `deactivate` if inside venv, then `.\superset\Scripts\activate` to activate again.


## Run SuperSet with Examples

Inside your venv with superset installed, initialize a sample sqlite database with command below.
```python
superset db upgrade
```

Setup database as admin.
```python
# Create an admin user in your metadata database 
# (use `admin` as username to be able to load the examples)
superset fab create-admin

# Load some data to play with
superset load_examples

# Create default roles and permissions
superset init

# To start a development web server on port 8088, use -p to bind to another port
superset run -p 8088 --with-threads --reload --debugger
```
> **_Note_**: You can remove `--debugger` to save some processing time

If everything worked, you should now have Superset running with sample data. Navigate to **_hostname:port_** in your local browser *(e.g. locally by default at localhost:8088)* and login using the username and password you created.

![Login page](/assets/img/posts/2024-12-20-install-superset/superset-login-page.png){: width="600"}

![With examples](/assets/img/posts/2024-12-20-install-superset/superset-with-examples.png){: width="600"}

You should be able to display an example dashboard using sample data like above. Happy visualizing!